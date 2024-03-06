<template>
  <div class="app">
    <div contenteditable="true" v-html="inputText" class="text-box" ref="myTextBox" @input="updateText" @click="removePlaceholder" @keydown="updateText" @keyup="updateText" @mouseup="updateText"></div>
    <div class="button-container">
      <button :class="{ 'highlighted': activeButton === 'normal_button' }" @click="surroundTextWithTag('normal_button')">X</button>
      <button :class="{ 'highlighted': activeButton === 'sup_button' }" @click="surroundTextWithTag('sup_button')">X<sup>y</sup></button>
      <button :class="{ 'highlighted': activeButton === 'sub_button' }" @click="surroundTextWithTag('sub_button')">X<sub>y</sub></button>
      <button :class="{ 'highlighted': activeButton === 'stack_button' }" @click="surroundTextWithboth()">X<span class="buttonstacked"><sup>z</sup><sub>y</sub></span></button>
      <button :class="{ 'highlighted': activeButton === 'arrow_button' }" @click="insertOperator('⇒')">⇒</button>
      <button :class="{ 'highlighted': activeButton === 'arrow_button' }" @click="insertOperator('⇄')">⇄</button>

      <button :class="{ 'highlighted': activeButton === 'compute_button' }" @click="compute()">Compute</button>
    </div>
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
      inputText: 'Start typing here...',
      activeButton: 'normal_button',
      displayText: '',
      operators: new Set(['⇒', '⇄']),
      caretPosition: 0,
      tags: new Map([['sup_button', ['<sup>', '</sup>']],
                     ['sub_button', ['<sub>', '</sub>']],
                     ['normal_button', ['', '']]
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
      var selectedText = this.removeTemplateLiterals(this.getSelectedText());
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

    compute(){
      throw new NotImplementedError();
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
        console.log("test")
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
  align-items: center;
}

.text-box {
  border: 1px solid #ccc;
  padding: 10px;
  width: 500px;
  min-height: 50px;
  overflow-y: auto;
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
