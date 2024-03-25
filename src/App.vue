<template>
  <div class="app">
    <div class="input-container">
      <div contenteditable="true" v-html="input" class="text-box" ref="myTextBox" @input="updateText" @click="removePlaceholder" @keydown="updateText" @keyup="updateText" @mouseup="updateText"></div>
      <div class="button-container">
        <button :class="{ 'highlighted': activeButton === 'normal_button' }" @click="surroundTextWithTag('normal_button')">X</button>
        <button :class="{ 'highlighted': activeButton === 'sup_button' }" @click="surroundTextWithTag('sup_button')">X<sup>y</sup></button>
        <button :class="{ 'highlighted': activeButton === 'sub_button' }" @click="surroundTextWithTag('sub_button')">X<sub>y</sub></button>
        <button :class="{ 'highlighted': activeButton === 'stack_button' }" @click="surroundTextWithboth()">X<span class="buttonstacked"><sup>z</sup><sub>y</sub></span></button>
        <button :class="{ 'highlighted': activeButton === 'arrow_button' }" @click="insertOperator('⇒')">⇒</button>
        <button :class="{ 'highlighted': activeButton === 'arrow_button' }" @click="insertOperator('⇄')">⇄</button>
        <button :class="{ 'highlighted': activeButton === 'compute_button' }" @click="this.computedText =compute(this.displayText)">Compute</button>
        <button :class="{ 'highlighted': activeButton === 'compute_button' }" @click="computeTestCases()">Test</button>
      </div>
    </div>
    <div class="text-box readonly-text-box" v-html="computedText"></div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';

class NotImplementedError extends Error {
    constructor(message = 'This feature is not implemented yet.') {
        super(message);
        this.name = 'NotImplementedError';
    }
}

export default defineComponent({
  data() {
    return {
      input: 'Start typing here...',
      activeButton: 'normal_button',
      displayText: '',
      computedText: '',
      operators: new Set(['⇒', '⇄']),
      caretPosition: 0,
      tags: new Map([
        ['sup_button', ['<sup>', '</sup>']],
        ['sub_button', ['<sub>', '</sub>']],
        ['normal_button', ['', '']]
      ]),
      elements: new Set([
        "H", "He", "Li", "Be", "B", "C", "N", "O", "F", "Ne", "Na", "Mg", "Al", "Si", "P", "S", "Cl", "Ar", "K",
        "Ca", "Sc", "Ti", "V", "Cr", "Mn", "Fe", "Co", "Ni", "Cu", "Zn", "Ga", "Ge", "As", "Se", "Br", "Kr", "Rb",
        "Sr", "Y", "Zr", "Nb", "Mo", "Tc", "Ru", "Rh", "Pd", "Ag", "Cd", "In", "Sn", "Sb", "Te", "I", "Xe", "Cs",
        "Ba", "La", "Ce", "Pr", "Nd", "Pm", "Sm", "Eu", "Gd", "Tb", "Dy", "Ho", "Er", "Tm", "Yb", "Lu", "Hf", "Ta",
        "W", "Re", "Os", "Ir", "Pt", "Au", "Hg", "Tl", "Pb", "Bi", "Po", "At", "Rn", "Fr", "Ra", "Ac", "Th", "Pa",
        "U", "Np", "Pu", "Am", "Cm", "Bk", "Cf", "Es", "Fm", "Md", "No", "Lr", "Rf", "Db", "Sg", "Bh", "Hs", "Mt"
      ])  
    };
  },
  methods: {

    surroundTextWithTag(button) {
      const selectedText = this.getSelectedText();
      let newText = '';
      if (selectedText.length === 0) {
        newText += this.tags.get(button)[0] + '&shy;' + this.tags.get(button)[1]; // WARNING: this uses an invisible character
      } else {
        newText = this.tags.get(button)[0];
        for (let i = 0; i < selectedText.length; i++) {
          if (this.operators.has(selectedText[i])){
            newText += this.tags.get(button)[1] + selectedText[i] + this.tags.get(button)[0];
          } else {
            newText += selectedText[i];
          }
        }
        newText += this.tags.get(button)[1];
      }

      document.execCommand('insertHTML', false, newText);
      this.activeButton = button;
    },

    surroundTextWithboth() {
      let selectedText = this.removeTemplateLiterals(this.getSelectedText());
      if (selectedText.length === 0) {
        selectedText = '▯ ▯';
      } else if (!selectedText.includes(' ')) {
        selectedText += ' ' + '▯';
      }
      let newText = '<span class="stacked"><sup>';
      for (let i = 0; i < selectedText.length; i++) {
        if (selectedText[i] === ' ') {
          newText += '</sup><sub>';
        } else if (!this.operators.has(selectedText[i])){
          newText += selectedText[i];
        }
      }
      newText += '</sub></span> ' + '&shy;'; // WARNING: this uses an invisible character 
      document.execCommand('insertHTML', false, newText);
      this.activeButton = 'normal_button';
    },

    insertOperator(char_entity) {
      const selectedText = this.getSelectedText();
      let newText = '';
      newText += this.removeTemplateLiterals(char_entity);
      document.execCommand('insertHTML', false, newText);
      this.activeButton = 'normal_button';
    },

    // TODO: not sure if it handles inv char
    // TODO: handle arrows
    // TODO: handle + operators
    // CHECK NOTEBOOK TOO
    compute(_input) {

        // Make copy instead of reference
        let input = _input; 

        input = this.unstackText(input);

        // Concatenate adjacent <sub> and <sup> strings
        input = input.replace(/<\/sub><sub>/g, '');
        input = input.replace(/<\/sup><sup>/g, '');

        // Remove ' ' and &shy; inside <sub></sub>
        input = input.replace(/­| /g, '');
        input = input.replace(/<sub>(.*?)&shy;(.*?)<\/sub>/g, '<sub>$1$2</sub>');

        if (!input.trim()) {
            this.computedText = "Error: input text is empty";
            console.log(this.computedText);
            return;
        }

        if (input.slice(0, 5) === '<sub>') {
            this.computedText = "Error: subscript without normal text";
            console.log(this.computedText);
            return;
        }

        let superscriptText = 0;
        let currentCharge = '';
        for (let i = 0; i < input.length; i++) {
          if (input.slice(i, i + 5) === '<sup>') {
            i += 5;          
            while (input.slice(i, i + 6) !== '</sup>') {
              console.log(input[i]);
              currentCharge += input[i];
              i++;
            }
            if (currentCharge === '-') {
              superscriptText -= 1;
            } else if (currentCharge === '+') {
              superscriptText += 1;
            } else {
              superscriptText += parseInt(currentCharge);
            }
            currentCharge = '';
          }
        }
        
        input = input.replace(/<sup>.*?<\/sup>/g, '');
        let pairs = [];
        let subscriptText = '';
        let currentElement = '';
        let insideSub = false;

        for (let i = 0; i < input.length; i++) {
            // If there is a subscript, we need to use it in the current pair
            if (input.slice(i, i+5) === '<sub>') {
                i += 5;
                while (input.slice(i, i+6) !== '</sub>') {
                    subscriptText += input[i];
                    i++;
                }
                insideSub = true;
                continue;
            }
            // If there is an ending subscript tag, reset insideSub and continue
            if (input.slice(i, i+6) === '</sub>') {
                insideSub = false;
                continue;
            }
            
            // Identify currentElement
            let endIndex = i;
            while (endIndex + 1 < input.length && input[endIndex + 1] === input[endIndex + 1].toLowerCase()) {
                endIndex++;
            }
            currentElement = input.slice(i, endIndex + 1);
            i = endIndex;
            
            // If there is a subscript associated with the current element, append it
            if (insideSub) {
                currentElement += '<sub>' + subscriptText + '</sub>';
                subscriptText = ''; 
                insideSub = false;
            }

            // Split currentElement to separate the element name and subscript
            let elementName = currentElement.replace(/<sub>.*<\/sub>/, '');
            if (!this.elements.has(elementName)) {
                this.computedText = `Error: "${elementName}" not a valid element`;
                console.log(this.computedText);
                return;
            }
            
            // If we reach the end of the input or if the next character is not a subscript, add pair [elementName, subscript] to pairs array
            if (i + 1 === input.length || input.slice(i, i+5) !== '<sub>') {
                let subscript = currentElement.match(/<sub>(.*?)<\/sub>/);
                pairs.push([elementName, subscript ? subscript[1] : 1]);
            }       
        }
        console.log(this.computedText);
        return [pairs, superscriptText];
    },
    
    // Split stacked text into subscript and superscript.
    //    Input: <span class="stacked"><sup>Y</sup><sub>Z</sub></span>
    //    Output: <sup>Y</sup><sub>Z</sub> (could also be <sub>Z</sub><sup>Y</sup>)
    unstackText(input) {
      console.log("input:", input);
      let output = '';
      let j = 0;
      for (let i = 0; i < input.length; i++) {
        if (input.substring(i, i+22) === '<span class="stacked">') {
            i += 22;
            j = i;
            while (input.substring(j, j+7) !== '</span>') {
                j++;
            }
            output += input.substring(i, j);
            i = j+6;
          } else {
            output += input[i];
        }
      }
      console.log("output:", output);
      return output;
    },
    
    // TODO: make tests actually work
    computeTestCases() {
      let tests = {
        // Expected inputs
        "HO": [[["H", "1"], ["O", "1"]], 0],
        "H<sub>2</sub>": [[["H", "2"]], 0],
        "H<sub>2</sub>O": [[["H", "2"], ["O", "1"]], 0],

        // Edgecases: Errors
        "": "Error: input text is empty",
        "<sub></sub>": "Error: input text is empty",
        "<sub>1</sub>": "Error: unclosed subscript tag",
        "H<sub>2</sub>O<sub>": "Error: unclosed subscript tag",
        "<sub>2</sub>": "Error: subscript without normal text",

        // Edgecases: adjacent subscripts
        "H<sub>2</sub><sub>2</sub>": [[["H", "22"]], 0],
        "H<sub>2</sub><sub></sub>": [[["H", "2"]], 0],

        // Edgecases: invisible characters: '&shy;'
        "H<sub>&shy;</sub>": [[["H", "1"]], 0],
        "H<sub>&shy;2</sub>": [[["H", "2"]], 0],
        "H<sub>2&shy;</sub>": [[["H", "2"]], 0],
        "H<sub>2&shy;3</sub>": [[["H", "23"]], 0],
        "H<sub>&shy;</sub>C<sub>&shy;</sub": [[["H", "1"], ["C", "1"]], 0],
        "H<sub>&shy;</sub>C<sub>2</sub": [[["H", "1"], ["C", "2"]], 0],

        // Edgecases: working with elements
        "H" : [[["H", "1"]], 0],
        "HCr" : [[["H", "1"], ["Cr", "1"]], 0],
        "H<sub>2</sub>" : [[["H", "2"]], 0],
        "HCr<sub>2</sub>" : [[["H", "1"], ["Cr", "2"]], 0],
        "BeB" : [[["Be", "1"], ["B", "1"]], 0],
        "Be<sub>2</sub>Be" : [[["Be", "2"], ["Be", "1"]], 0],
        "be" : "Error: \"be\" not a valid element",

        // superscript (base)
        "Be<sup>0</sup>": [[["Be", "1"]], 0],
        "Be<sup>1</sup>": [[["Be", "1"]], 1],
        "Be<sup>2</sup>": [[["Be", "1"]], 2],

        // superscript (calculating)
        "Be<sup>1</sup>B<sup>2</sup>": [[["Be", "1"], ["B", "2"]], 3],
        "Be<sup>1</sup>B<sup>-1</sup>": [[["Be", "1"], ["B", "2"]], 0],
        "Be<sup>1</sup>B<sup>-2</sup>": [[["Be", "1"], ["B", "2"]], -1],

        // superscript (shorthand)
        "Be<sup>+</sup>": [[["Be", "1"]], 1],
        "Be<sup>-</sup>": [[["Be", "1"]], -1],
        "Be<sup>-</sup>B<sup>+</sup>": [[["Be", "1"], ["B", "1"]], 0],
      };
      
      let passed = true;
      let testKeys = Object.keys(tests);
      let testValues = Object.values(tests);
      for (let i = 0; i < testKeys.length; i++) {
        passed = this.compute(testKeys[i]) == testValues[i];
        console.log(`Test: ${i+1} -- ${passed ? 'Passed' : 'Failed'}`);
        console.log(`Input: ${testKeys[i]}`);
        console.log(`Expected: ${testValues[i]}`);
        if (!passed) {
          console.log(`Received: ${this.compute(testKeys[i])}`);
        }
      }
    },

    getSelectedText() {
      const selection = window.getSelection();
      if (selection.rangeCount > 0) {
        const range = selection.getRangeAt(0);
        return range.toString();
      }
      return null;
    },
    
    updateCaretPosition() {
      const selection = window.getSelection();
      if (selection.rangeCount > 0) {
        const range = selection.getRangeAt(0);
        const preCaretRange = range.cloneRange();
        preCaretRange.selectNodeContents(this.$refs.myTextBox);
        preCaretRange.setEnd(range.endContainer, range.endOffset);
        this.caretPosition = preCaretRange.toString().length;
      }
    },

    removeAdjacent(char){
      throw new NotImplementedError();
    },

    normalizeOperators(){
      for (let i; i < this.displayText.length; i++){
        if (this.operators.has(this.displayText[i])){
          this.removeTemplateLiterals(this.displayText[i]);
        }
      }
    },

    removeTemplateLiterals(str) {
      const regex = /\${(.*?)}/g;
      return str.replace(regex, '');
    },

    updateText(event) {
      this.displayText = event.target.innerHTML;
      this.normalizeOperators();
      // this.updateCaretPosition();
      // this.removeAdjacent('▯') 

      console.log(`${this.displayText}`); // logs with with HTML template literals
    },

    removePlaceholder(event) {
      if (event.target.innerText === 'Start typing here...') {
        event.target.innerText = '';
      }
    },
  }
});
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.input-container {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
}

.text-box {
  border: 1px solid #ccc;
  padding: 10px;
  width: 500px;
  min-height: 50px;
  overflow-y: auto;
  display: block;
  margin-top: 10px;
}

.button-container {
  margin-top: 10px;
  white-space: nowrap;
}

.button-container button {
  margin-left: 5px;
  display: inline-block;
}

.placeholder {
  color: #bbbbbb;
}

.highlighted {
  background-color: #9e9e9e;
}

.stacked,
.buttonstacked {
  position: relative;
  display: inline-block;
}

.stacked sup,
.stacked sub,
.buttonstacked sup,
.buttonstacked sub {
  position: absolute;
  left: 0;
  display: block;
  text-align: center;
}

/* two different stacked ratios because the button looks weird if it's accurate */
.buttonstacked sup {
  top: -1.8em;
}

.buttonstacked sub {
  bottom: -0.7em;
}

.stacked sup {
  top: -2em;
}

.stacked sub {
  bottom: -1.25em;
}
</style>
