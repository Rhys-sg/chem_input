<template>
  <div class="app">
    <div contenteditable="true" v-html="inputText" class="text-box" @input="updateText" @click="removePlaceholder"></div>
    <div class="button-container">
      <button :class="{ 'highlighted': activeButton === 'normal_button' }" @click="surroundTextWithTag('normal_button')">X</button>
      <button :class="{ 'highlighted': activeButton === 'sup_button' }" @click="surroundTextWithTag('sup_button')">X<sup>y</sup></button>
      <button :class="{ 'highlighted': activeButton === 'sub_button' }" @click="surroundTextWithTag('sub_button')">X<sub>y</sub></button>
      <button :class="{ 'highlighted': activeButton === 'stack_button' }" @click="surroundTextWithboth('stack_button')">X<span class="stacked"><sup>z</sup><sub>y</sub></span></button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputText: '<p class="placeholder">Start typing here...</p>',
      activeButton: 'normal_button',
      displayText: '',
      tags: new Map([
        ['sup_button', ['<sup>', '</sup>']],
        ['sub_button', ['<sub>', '</sub>']],
        ['normal_button', ['', '']]
      ])
    };
  },
  methods: {

    surroundTextWithTag(button) {
      const selectedText = this.getSelectedText();
      let newText = '';
      if (selectedText.length === 0){
        newText += this.tags.get(button)[0] + '&shy;' + this.tags.get(button)[1]; // WARNING: this uses an invisible character
      } else {
        for (let i = 0; i < selectedText.length; i++) {
          newText += this.tags.get(button)[0] + selectedText[i] + this.tags.get(button)[1];
        }
      }
      
      document.execCommand('insertHTML', false, newText);
      this.activeButton = button;
    },

    surroundTextWithboth(button) {
      var selectedText = this.getSelectedText();
      if (selectedText.length === 0){
        // selectedText = '▯ ' + '&shy;';
        selectedText = '▯ ▯';
      } else if (!selectedText.includes(' ')) {
        selectedText += ' ' + '▯';
      }
      let newText = '<span class="stacked"><sub>';
      for (let i = 0; i < selectedText.length; i++) {
        if (selectedText[i] === ' ') {
          newText += '</sub><sup>';
        } else{
          newText += selectedText[i];
        }
      }
      // newText += '</sup></span>';
      newText += '</sup></span> ' + '&shy;'; // WARNING: this uses an invisible character 
      document.execCommand('insertHTML', false, newText);
      this.activeButton = 'normal_button';
    },


    getSelectedText() {
      const selection = window.getSelection();
      if (selection.rangeCount > 0) {
        const range = selection.getRangeAt(0);
        return range.toString();
      }
      return null;
    },

    updateText(event) {
      this.displayText = event.target.innerHTML;
      console.log(`${this.displayText}`); // logs with with HTML template literals (<sup>, </sup>, <sup>, and </sup>)
    },

    removePlaceholder(event) {
      if (event.target.innerText === 'Start typing here...') {
        event.target.innerText = '';
      }
    },
  }
};
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
}

.button-container button {
  margin-left: 5px;
}

.placeholder {
  color: #bbbbbb;
}

.highlighted {
  background-color: #9e9e9e;
}

.stacked {
  position: relative;
  display: inline-block;
}

.stacked sup,
.stacked sub {
  position: absolute;
  left: 0;
  display: block;
  text-align: center;
}

.stacked sup {
  top: -0.3em;
}

.stacked sub {
  bottom: 0.7em;
}

</style>
