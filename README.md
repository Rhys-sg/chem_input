# Chemical Equation Input

Notes:
* Options for script levels:
  * normal
  * superscript
  * subscript
  * both superscript and subscript
* Highlight a section and clicking superscript or subscript button will change highlighted text to that level.
* Without highlighting, click superscript or subscript button to move caret to that script level.
* Without highlighting, clicking both superscript and subscript button will put caret at the normal level. It will create two boxes at each level.
  * The boxes would ideally be temporary, but I could not figure out a good way of removing them.

Design choices:
* Highlighting section and clicking button for both superscript and subscript will move all highlighted text to superscript.
* Operators will always be on the normal level, even if you highlight them and click to change level. However, all other highlighted text will change.
* Highlighting section with an operator and clicking button for both superscript and subscript will delete the operator.

# Getting Started 

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
