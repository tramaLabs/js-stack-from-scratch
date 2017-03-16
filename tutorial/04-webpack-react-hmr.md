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

Se você quiser algumas das maiores novidades do ES no seu cliente, como `Promise`, você vai precisar incluir o [Babel Polyfill](https://babeljs.io/docs/usage/polyfill/) antes de qualquer coisa do seu pacote.

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
    filename: 'js/bundle.js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: isProd ? '/static/' : `http://localhost:${WDS_PORT}/dist/`,
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

O arquivo é usado para descrever como nosso pacote deve ser empacotado: `entry` é o ponto de partida da nossa aplicação, `output.filename` é o nome do pacote a ser gerado, `output.path` e `output.publicPath` descrevem a pasta destino e a URL. Nós colocamos o pacote na pasta `dist`, que irá conter os arquivos que são gerados automaticamente (diferentemente do css declarativo que nós criamos anteriormente, que está em `public`). `module.rules` é onde nós dizemos ao Webpack para aplicar um tratamento a alguns tipos de arquivos. Aqui nós dizemos que nós queremos que todos os arquivos `.js` e `.jsx` (para o React), exceto os que estão em `node_modules`, passem por alguma coisa chamado `babel-loader`. Nós também queremos que essas duas extensões sejam usadas em `resolve` quando nós usamos `import`. Finalmente, nós declaramos uma porta para o Webpack Dev Server (Servidor de desenvolvimento do Webpack).

**Nota**: A extensão `.babel.js` é uma propriedade do Webpack para aplicar nossas transformações do Babel ao arquivo config

`babel-loader` é uma extensão para o Webpack, que transpila o seu código assim como estamos fazendo desde o início desse tutorial. A unica diferença é que dessa vez, o código irá rodar no navegador invés do servidor.

- Execute `yarn add --dev webpack webpack-dev-server babel-core babel-loader`.

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
  "prod:stop": "pm2 delete server",
  "lint": "eslint src webpack.config.babel.js --ext .js,.jsx",
  "test": "yarn lint && flow && jest --coverage",
  "precommit": "yarn test",
  "prepush": "yarn test && yarn prod:build"
},
```

Em `dev:start` nós explicitamente declaramos as extensões de arquivos para monitorar, `.js` e `.jsx`, e adcionamos add `dist` nos diretórios ignoráveis.

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

Ok, essas foram grandes mudanças, vamos ver se tudo funciona como esperado:

🏁 Rode `yarn start` no terminal. Abra outra aba ou janela do terminal, e rode `yarn dev:wds`. Quando o Webpack Dev Server terminar de gerar o pacote e os sourcemaps (oque juntos devem ser ~600kB em arquivos) e ambos os processos estiverem rodando no seu terminal, abra `http://localhost:8000` e você deve ver "Hello Webpack!". Abra o console do seu Chrome, e na aba Source, veja quais arquivos foram incluidos. Você deve ver apenas `static/css/style.css`
em `localhost:8000`, e todos os seus arquivos ES6 em `webpack://./src`. Isso significa que os sourcemaps estão funcionando. No seu editor, em `src/client/index.js`, tente mudar `Hello Webpack!` para outra string. Quando você salvar o arquivo, o Webpack Dev Server, no seu terminal, deve gerar outro pacote e o seu Chrome deve recarregar automaticamente.

- Mate os processos passados no seu terminal, usando Ctrl+C, então rode `yarn prod: build`, e depois `yarn prod:start`. Abra `http://localhost:8000` e você deve continuar vendo "Hello Webpack!". Na aba Source do console do seu Chrome. você deve achar, dessa vez, `static/js/bundle.js` em `localhost:8000/`, mas nenhum source em `webpack://`. Clique em `bundle.js` para ter certeza de que está minificado. Execute `yarn prod:stop`.

Bom trabalho, eu sei que isso foi um pouco denso. Você merece um descanço! A proxima seção será mais facil.

**Nota**: Eu recomendo que você tenha pelo menos 3 terminais abertos, um para o seu servidor Express, outro para o Webpack Dev Server, e mais um para o Git, testes, e comandos em geral, como instar pacotes com `yarn`.

## React

> 💡 **[React](https://facebook.github.io/react/)** é uma biblioteca para criar interfaces de usuário criado pelo Facebook. Ele usas a sintaxe do   **[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)** para representar elementos HTML e componentes enquanto dá mais poder ao JavaScript.

Nessa seção nós vamos renderizar um texto usando React e JSX.

Primeiro, vamos instalar o React e ReactDOM:

- Execute `yarn add react react-dom`

Troque seu arquivo `src/client/index.js` para `src/client/index.jsx` e escreva o código React nele:

```js
// @flow

import 'babel-polyfill'

import React from 'react'
import ReactDOM from 'react-dom'

import App from './app'
import { APP_CONTAINER_SELECTOR } from '../shared/config'

ReactDOM.render(<App />, document.querySelector(APP_CONTAINER_SELECTOR))
```

- Crie um arquivo `src/client/app.jsx` contendo:

```js
// @flow

import React from 'react'

const App = () => <h1>Hello React!</h1>

export default App
```

Já que nos usamos a sintaxe do JSX, nós precisamos falar para o Babel que ele precisa transformar isso usando o preset `babel-preset-react`. E já que estamos nele, nós tambem precisaremos adicionar um plugin de Babel chamado `flow-react-proptypes`, que automaticamente gera PropTypes de anotações Flow para seus componentes React.

- Execute `yarn add --dev babel-preset-react` e adicione react no seu arquivo `.babelrc` dessa forma:

```json
{
  "presets": [
    "env",
    "flow",
    "react"
  ],
  "plugins": [
    "flow-react-proptypes"
  ]
}
```

🏁 Rode `yarn start` e `yarn dev:wds` e vá em `http://localhost:8000`. Você deve ser "Hello React!".

Agora, tente mudar o texto em `src/client/app.jsx` para qualquer outra coisa. O Webpack Dev Server deve recarregar a página automaticamente, o que é muito bom, mas nós vamos deixa-lo ainda melhor.

## Hot Module Replacement

> 💡 **[Hot Module Replacement](https://webpack.js.org/concepts/hot-module-replacement/)** (*HMR*) é uma ferramenta muito poderosa, do Webpack, para trocar os modulos automaticamente sem precisar recarregar a página inteira.

Para fazer o HMR funcionar com o React, nós vamos precisar mudar um pouco as coisas.

- Execute `yarn add react-hot-loader@next`

- Edite o seu `webpack.config.babel.js` dessa forma:

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

Nós precisamos fazer nosso `App` do `AppContainer` do `react-hot-loader`, e nós precisamos pedir, com o `require`, a proxima versão do nosso `App` quando executado o hot-reload. Para fazer esse processo simples e fácil, nós criamos uma pequena função `wrapApp` que nós usamos em todos os casos que foi preciso renderizar o `App`. Você pode mover o `eslint-disable global-require` para o começo do arquivo para deixa-lo mais legível.

🏁 Reinicie o seu processo `yarn dev:wds`, se ele ainda estiver rodando. Abra `localhost:8000`. Na aba de Console, você deve ver alguns logs sobre o HMR. Vá em frente e mude alguma coisa no arquivo `src/client/app.jsx` e suas mudanças devem ser refletidas no browser depois de alguns segundo, sem precisar recarregar a página inteira!

Proxima seção: [05 - Redux, Immutable, Fetch](05-redux-immutable-fetch.md#readme)

Voltar para a [seção anterior](03-express-nodemon-pm2.md#readme) ou para
[tabela de conteúdos](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
