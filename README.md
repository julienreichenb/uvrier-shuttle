# uvrier-shuttle

## Installation
### Prerequisites
This `README.md` assumes a functional development environment:
* [Node.js](https://nodejs.org)
* [Visual Studio Code](https://code.visualstudio.com/download)
    + [Vue/CLI](https://code.visualstudio.com/docs/nodejs/vuejs-tutorial#_welcome-to-vue)
    + [Vetur extension](https://code.visualstudio.com/docs/nodejs/vuejs-tutorial#_vetur-extension)
* [Yarn package manager](https://yarnpkg.com/)

Hint: test that the following commands run correctly:
* `node --version`
* `npm --version`
* `code --version`
* `yarn --version`

## Project setup
### Setup configuration files
For dev machines:
- duplicate [.env.development.example](.env.development.example)
- rename the copy to [.env.development](.env.development)
- edit the "TODO" in [.env.development](.env.development), which involves setting the Bestmile API key.

### Install all package required by project
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn serve
```

### Compiles and minifies for production
```
yarn build
```

### Lints and fixes files
```
yarn lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
