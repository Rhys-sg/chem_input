<template>
    <div class="input-container">
        <div contenteditable="true" v-html="input" class="text-box" ref="myTextBox" @input="updateText" @click="removePlaceholder" @keydown="updateText" @keyup="updateText" @mouseup="updateText"></div>
        <div class="button-container">
        <button :class="{ 'highlighted': activeButton === 'normal_button' }" @click="surroundTextWithTag('normal_button')">X</button>
        <button :class="{ 'highlighted': activeButton === 'sup_button' }" @click="surroundTextWithTag('sup_button')">X<sup>y</sup></button>
        <button :class="{ 'highlighted': activeButton === 'sub_button' }" @click="surroundTextWithTag('sub_button')">X<sub>y</sub></button>
        <button :class="{ 'highlighted': activeButton === 'stack_button' }" @click="surroundTextWithboth()">X<span class="buttonstacked"><sup>z</sup><sub>y</sub></span></button>
        <button :class="{ 'highlighted': activeButton === 'compute_button' }" @click="this.computedText =compute(this.displayText)">Compute</button>
        </div>
    </div>
    <div class="text-box readonly-text-box" v-html="computedText"></div>
</template>
<script>
import { defineComponent } from 'vue';

import {
  surroundTextWithTag,
  surroundTextWithboth,
  insertOperator,
  compute,
  computeTestCases,
  getSelectedText,
  updateCaretPosition,
  removeAdjacent,
  normalizeOperators,
  removeTemplateLiterals,
  updateText,
  removePlaceholder
} from './methods.js';

export default defineComponent({
  name: "ChemInput",
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
    surroundTextWithTag,
    surroundTextWithboth,
    insertOperator,
    compute,
    computeTestCases,
    getSelectedText,
    updateCaretPosition,
    removeAdjacent,
    normalizeOperators,
    removeTemplateLiterals,
    updateText,
    removePlaceholder
  }
});
</script>