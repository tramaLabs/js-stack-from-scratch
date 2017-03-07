# 04 - Webpack, React, e Hot Module Replacement

O código para esse capítulo está disponivel [aqui](https://github.com/verekia/js-stack-walkthrough/tree/master/04-webpack-react-hmr).

## Webpack

> 💡 **[Webpack](https://webpack.js.org/)** é um **empacotador de módulos**. Ele pega vários arquivos fonte, processa-os, e então os junta é um (geralmente) arquivo JavaScript chamado pacote, cujo é o único arquivo que o seu cliente irá executar.

Vamos criar um *olá mundo* básico e empacota-lo com webpack.

- Em `src/shared/config.js`, adcione as seguintes contantes:

```js
export const WDS_PORT = 7000

export const APP_CONTAINER_CLASS = 'js-app'
export const APP_CONTAINER_SELECTOR = `.${APP_CONTAINER_CLASS}`
```

- Crie o arquivo `src/client/index.js` contendo:

```js
import 'babel-polyfill'

import { APP_CONTAINER_SELECTOR } from '../shared/config'

document.querySelector(APP_CONTAINER_SELECTOR).innerHTML = '<h1>Hello Webpack!</h1>'
```

If you want to use some of the most recent ES features in your client code, like `Promise`s, you need to include the [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/) before anything else in in your bundle.

Se você quiser as maiores novidades do ES no seu cliente, como `Promise`, você vai precisar incluir o [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/) antes de qualquer coisa do seu pacote.

- Execute `yarn add babel-polyfill`

Se você executar ESLint nesse arquivo, ele irá reclamar de que `document` não estar definido.

- Adcione o seguinte ao `env` no seu `.eslintrc.json` para habilitar a utilização do `window` e `document`:

```json
"env": {
  "browser": true,
  "jest": true
}
```

Agora, nós precisamos empacotar essa aplicação ES6 em um pacote ES5.

- Crie o arquivo `webpack.config.babel.js` contendo:

```js
// @flow

import path from 'path'

import { WDS_PORT } from './src/shared/config'
import { isProd } from './src/shared/util'

export default {
  entry: [
    './src/client',
  ],
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist/js'),
    publicPath: `http://localhost:${WDS_PORT}/dist/js/`,
  },
  module: {
    rules: [
      { test: /\.(js|jsx)$/, use: 'babel-loader', exclude: /node_modules/ },
    ],
  },
  devtool: isProd ? false : 'source-map',
  resolve: {
    extensions: ['.js', '.jsx'],
  },
  devServer: {
    port: WDS_PORT,
  },
}
```

O arquivo é usado para descrever como nosso pacote deve ser empacotado: `entry` é o ponto de partida da nossa aplicação, `output.filename` é o nome do pacote a ser gerado, `output.path` e `output.publicPath` descrevem a pasta destino e a URL. Nós colocamos o pacote na pasta `dist`, que irá conter os arquivos que são gerados automaticamente (diferentemente do css declarativo que nós criamos anteriormente, que está em `public`). `module.rules` é onde nós dizemos ao Webpack para aplicar um tratamento a alguns tipos de arquivos. Aqui nós dizemos que nós queremos que todos os arquivos `.js` e `.jsx` (para o React), exceto os que estão em `node_modules`, passem por alguma coisa chamado `babel-loader`. Nós também queremos que essas duas extencões em `resolve`. Finalmente, nós declaramos uma porta para o Webpack Dev Server (Servidor de desenvolvimento do Webpack).

**Nota**: A extensão `.babel.js` é uma propriedade do Webpack para aplicar nossas transformações do Babel ao arquivo config

`babel-loader` is a plugin for Webpack that transpiles your code just like we've been doing since the beginning of this tutorial. The only difference is that this time, the code will end up running in the browser instead of your server.

`babel-loader` é uma extensão para o Webpack, que transpila o seu código assim como estamos fazendo desde o início desse tutorial. A unica diferença é que dessa vez, o código irá rodar no navegador invés do servidor.

- Execute `yarn add --dev webpack webpack-dev-server babel-core babel-loader`.
- 
`babel-core` é uma dependência intrinseca de `babel-loader`, então nós o instalamos também.

- Adcione `/dist/` ao seu `.gitignore`.

### Atualização de tarefas

No modo de deselvomvimento, nós vamos usar o `webpack-dev-server` para tirar vantagem do Hot Module Reloading (Atualização de modulos em execução), e em produção nós iremos apenas usar `webpack` para gerar nossos pacotes. Em ambos os casos, a bandeira `--progress`será util para mostrar informações adcionais quando o Webpack está compilando nossos arquivos. Em produção, nós também iremos passar a bandeira `-p` para que o `webpack` minifique nosso código, e colocar o valor da variavel `NODE_ENV` para `production`.

Vamos atualizar nossos `scripts` para implementar tudo isso, e melhorar nossas tarefas também:

```json
"scripts": {
  "start": "yarn dev:start",
  "dev:start": "nodemon -e js,jsx --ignore lib --ignore dist --exec babel-node src/server",
  "dev:wds": "webpack-dev-server --progress",
  "prod:build": "rimraf lib dist && babel src -d lib --ignore .test.js && cross-env NODE_ENV=production webpack -p --progress",
  "prod:start": "cross-env NODE_ENV=production pm2 start lib/server && pm2 logs",
  "prod:stop": "pm2 delete all",
  "lint": "eslint src webpack.config.babel.js --ext .js,.jsx",
  "test": "yarn lint && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test && yarn prod:build"
},
```

Em `dev:start` nós explicitamente declaramos as extensões de arquivos para monitorar, `.js` e `.jsx`, e adcionamos add `dist` nos diretórios ignoráveis.

We created a separate `lint` task and added `webpack.config.babel.js` to the files to lint.

Nós criamos uma tarefa separada ,`link`, e adcionamos `webpack.config.babel.js` aos arquivos em que o lint será executado.

- Em seguida, vamos criar o container para nossa aplicação em `src/server/render-app.js`, e incluir o pacote que nós iremos gerar:

```js
// @flow

import { APP_CONTAINER_CLASS, STATIC_PATH, WDS_PORT } from '../shared/config'
import { isProd } from '../shared/util'

const renderApp = (title: string) =>
`<!doctype html>
<html>
  <head>
    <title>${title}</title>
    <link rel="stylesheet" href="${STATIC_PATH}/css/style.css">
  </head>
  <body>
    <div class="${APP_CONTAINER_CLASS}"></div>
    <script src="${isProd ? STATIC_PATH : `http://localhost:${WDS_PORT}/dist`}/js/bundle.js"></script>
  </body>
</html>
`

export default renderApp
```

Dependendo do nosso ambiente em que nós estamos, nós iremos incluir ou o pacote do Webpack Dev Server, ou o pacote de produção. Veja que o caminho do pacote do Webpack Dev Server é *virtual*, `dist/js/bundle.js` não é realmente lido do disco em desenvolvimento. Também é necessário dar ao Webpack Dev Server uma porta diferente do sua porta principal para web.

- Finalmente, em `src/server/index.js`, mude a mensagem do seu `console.log` para:

```js
console.log(`Server running on port ${WEB_PORT} ${isProd ? '(production)' :
  '(development).\nKeep "yarn dev:wds" running in an other terminal'}.`)
```

Isso vai dar aos outros desenvolvedores uma dica sobre oque fazer no caso deles executarem `yarn start` sem o Webpack Dev Server.

Alright that was a lot of changes, let's see if everything works as expected:

🏁 Run `yarn start` in a terminal. Open an other terminal tab or window, and run `yarn dev:wds` in it. Once Webpack Dev Server is done generating the bundle and its sourcemaps (which should both be ~600kB files) and both processes hang in your terminals, open `http://localhost:8000/` and you should see "Hello Webpack!". Open your Chrome console, and under the Source tab, check which files are included. You should only see `static/css/style.css` under `localhost:8000/`, and have all your ES6 source files under `webpack://./src`. That means sourcemaps are working. In your editor, in `src/client/index.js`, try changing `Hello Webpack!` into any other string. As you save the file, Webpack Dev Server in your terminal should generate a new bundle and the Chrome tab should reload automatically.

- Kill the previous processes in your terminals with Ctrl+C, then run `yarn prod:build`, and then `yarn prod:start`. Open `http://localhost:8000/` and you should still see "Hello Webpack!". In the Source tab of the Chrome console, you should this time find `static/js/bundle.js` under `localhost:8000/`, but no `webpack://` sources. Click on `bundle.js` to make sure it is minified. Run `yarn prod:stop`.

Good job, I know this was quite dense. You deserve a break! The next section is easier.

**Note**: I would recommend to have at least 3 terminals open, one for your Express server, one for the Webpack Dev Server, and one for Git, tests, and general commands like installing packages with `yarn`. Ideally, you should split your terminal screen in multiple panes to see them all.

## React

> 💡 **[React](https://facebook.github.io/react/)** is a library for building user interfaces by Facebook. It uses the **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** syntax to represent HTML elements and components while leveraging the power of JavaScript.

In this section we are going to render some text using React and JSX.

First, let's install React and ReactDOM:

- Run `yarn add react react-dom`

Rename your `src/client/index.js` file into `src/client/index.jsx` and write some React code in it:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- Create a `src/client/app.jsx` file containing:

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

Since we use the JSX syntax here, we have to tell Babel that it needs to transform it as well.

- Run `yarn add --dev babel-preset-react` and add `react` to your `.babelrc` file like so:

```json
{
  "presets": [
    "env",
    "flow",
    "react"
  ]
}
```

🏁 Run `yarn start` and `yarn dev:wds` and hit `http://localhost:8000`. You should see "Hello React!".

Now try changing the text in `src/client/app.jsx` to something else. Webpack Dev Server should reload the page automatically, which is pretty neat, but we are going to make it even better.

## Hot Module Replacement

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) is a powerful Webpack feature to replace a module on the fly without reloading the entire page.

To make HMR work with React, we are going to need to tweak a few things.

- Run `yarn add react-hot-loader@next`

- Edit your `webpack.config.babel.js` like so:

```js
import webpack from 'webpack'
// [...]
entry: [
  'react-hot-loader/patch',
  './src/client',
],
// [...]
devServer: {
  port: WDS_PORT,
  hot: true,
},
plugins: [
  new webpack.optimize.OccurrenceOrderPlugin(),
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NamedModulesPlugin(),
  new webpack.NoEmitOnErrorsPlugin(),
],
```

- Edit your `src/client/index.jsx` file:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'
import { AppContainer } from 'react-hot-loader'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

const rootEl = document.querySelector(APP_CONTAINER_SELECTOR)

const wrapApp = AppComponent =>
  <AppContainer>
    <AppComponent />
  </AppContainer>

ReactDOM.render(wrapApp(App), rootEl)

if (module.hot) {
  // flow-disable-next-line
  module.hot.accept('./app', () => {
    // eslint-disable-next-line global-require
    const NextApp = require('./app').default
    ReactDOM.render(wrapApp(NextApp), rootEl)
  })
}
```

We need to make our `App` a child of `react-hot-loader`'s `AppContainer`, and we need to `require` the next version of our `App` when hot-reloading. To make this  process clean and DRY, we create a little `wrapApp` function that we use in both places it needs to render `App`. Feel free to move the `eslint-disable global-require` to the top of the file to make this more readable.

🏁 Restart your `yarn dev:wds` process if it was still running. Open `localhost:8000`. In the Console tab, you should see some logs about HMR. Go ahead and change something in `src/client/app.jsx` and your changes should be reflected in your browser after a few seconds, without any full-page reload!

Next section: [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

Back to the [previous section](03-express-nodemon-pm2.md#readme) or the [table of contents](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
