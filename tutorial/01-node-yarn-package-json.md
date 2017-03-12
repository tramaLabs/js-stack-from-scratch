# 01 - Node, Yarn, e `package.json`

O código desse capítulo está disponível [aqui](https://github.com/verekia/js-stack-walkthrough/tree/master/01-node-yarn-package-json).

Nessa sessão nós iremos configurar Node, Yarn, um arquivo `package.json` inicial, e testar um pacote.

## Node

> 💡 **[Node.js](https://nodejs.org/)** é um ambiente em tempo de execução em JavaScript. É usado majoritariamente para desenvolvimento back-end, mas também é muito utilizado para criação de scripts em geral. No contexto do deenvolvimento front-end, pode ser utilizado para executar uma grande sequência de tarefas, como lint, testes e unificação de arquivos assembly.

Iremos utilizar Node para basicamente tudo nesse tutorial, então você com certeza precisará dele. Acesse a [página de download](https://nodejs.org/en/download/current/) para versões binárias de **macOS** ou **Windows**, ou a [página para instalação via package manager](https://nodejs.org/en/download/package-manager/) para distribuições Linux.

Como exemplo, no **Ubuntu / Debian**, você precisa executar os seguintes comandos para instalar o Node:

```sh
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Você irá precisar de qualquer versão > 6.5.0.

## NVM

Se o Node já estiver instalado, ou se você quer mais flexibilidade, instale NVM ([Node Version Manager](https://github.com/creationix/nvm)), rode NVM install e use a última versão disponível do Node para você.

## NPM

NPM é o gereciador de pacotes padrão do Node. Ele é instalado automaticamente durante a instalação do Node. Gerenciadores de pacotes são usados para instalar e gerenciar pacotes (módulos de código que você ou outro alguém escreveu). Nós iremos usar vários pacotes nesse tutorial, mas iremos aprender a usar o Yarn, outro gerenciador de pacotes.

## Yarn

> 💡 **[Yarn](https://yarnpkg.com/)** é um gerenciador de pacotes de Node.JS que é muito mais rápido que o NPM, tem suporte offline e busca por dependências [mais assertivamente](https://yarnpkg.com/en/docs/yarn-lock).

Desde que [foi publicado](https://code.facebook.com/posts/1840075619545360) em outubro de 2016, ele recebeu uma adoção muito rápida e talvez logo se torne o gerenciador de pacote padrão da comunidade JavaScript. Se você quiser continuar usando o NPM simplesmente substitua todos os comandos `yarn add` e `yarn add --dev` desse tutorial por `npm install --save` e `npm install --save-dev`.

Instale Yarn seguindo as [instruções](https://yarnpkg.com/en/docs/install) para o seu sistema operacional. Eu recomendo utilizar o **script de instalação** da aba *Alternatives* se você estiver no macOS ou no Linux, para [evitar](https://github.com/yarnpkg/yarn/issues/1505) depender de outros gerenciadores de pacotes:

```sh
curl -o- -L https://yarnpkg.com/install.sh | bash
```

## `package.json`

> 💡 **[package.json](https://yarnpkg.com/en/docs/package-json)** é o arquivo usado para descrever e configura seu projeto JavaScript. Ele contem informações gerais (o nome do seu projeto, versão, contribuidores, licença, etc), opções de configuração para as ferramentas que você está usando, e até uma sessão para executar *tarefas*.

- Crie uma nova pasta para trabalhar, e entre `cd` nela.
- Execute `yarn init` e responda as questões (`yarn init -y` para pular todas as questões), para gerar um arquivo `package.json` automaticamente.

Aqui está um arquivo básico `package.json` que eu usarei nesse tutorial:

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT"
}
```

## Hello World

- Crie um arquivo `index.js` contendo `console.log('Hello world')`

🏁 Execute `node .` nessa pasta (`index.js` é o arquivo que o NodeJS busca na sua pasta por padrão). Deve ser impresso "Hello world".

**Nota**: Está vendo esse emoji de bandeirinha de corrida 🏁? Eu vou usar toda vez que você chegar em um **checkpoint**. Nós iremos algumas vezes fazer uma série de mudanças em sequência, e seu código pode não funcionar até você chegar ao checkout seguinte.

## O script `start`

Executar `node .` para rodar nosso programa é um pouco baixo-nível. Em vez disso, nós vamos usar um script do NPM/Yarn para disparar a execução desse código. Isso nos dará uma boa abstração para que sempre possamos utilizar `yarn start`, mesmo quando nosso programa ficar mais complicado.

- No `package.json`, adicione um objeto `scripts` como segue:

```json
{
  "name": "your-project",
  "version": "1.0.0",
  "license": "MIT",
  "scripts": {
    "start": "node ."
  }
}
```

`start` é o nome que damos à *tarefa* que rodará nosso programa. Nós iremos criar uma quantidade de diferentes tarefas nesse objeto `scripts` durante o tutorial. Outros nomes de tarefa padrão são `stop` e `test`.

`package.json` precisa ser um arquivo JSON válido, o que significa que não pode conter vírgulas sobrando (trailing commas). Então seja cuidadoso quando editar seu `package.json` manualmente.

🏁 Rode `yarn start`. Isso deve imprimir `Hello world`.

## Git e `.gitignore`

- Inicialize um repositório git com `git init`

- Crie um arquivo `.gitignore` e adicione o seguinte conteúdo nele:

```gitignore
.DS_Store
/*.log
```

Arquivos `.DS_Store` são gerados automaticamente pelo macOS e você não deve querer tê-los em seu repositório.

`npm-debug.log` e `yarn-error.log` são arquivos que são criados sempre que nosso gerenciador de pacotes encontra um error. Nós não queremos eles versionados em nosso repositório.

## Instalando e usando um pacote

Nessa sessão nós iremos instalar e usar um pacote. Um "pacote" é uma simples peça de código que alguém escreveu, e que você pode usar em seu próprio código. Ele pode ser qualquer coisa. Aqui, nós iremos adicionar um pacote que ajuda a manipular cores.

- Instale o pacote criado pela comunidade chamado `color` executando  `yarn add color`

Abra o arquivo `package.json` para ver como o Yarn automaticamente adicionou `color` em `dependencies`.

Uma pasta `node_modules` foi criada para salvar seu pacote.

- Adicione `node_modules/` em seu arquivo `.gitignore`

Você também notará que um arquivo `yarn.lock` foi gerado pelo Yarn. Você deve commitar esse arquivo no seu repositório, já que ele garante que todos no seu time usem a memsa versão dos seus pacotes. Se você estiver usando NPM em vez de Yarn, o equivalente a esse arquivo é o *shrinkwrap*.

- Escreva o seguinte código no seu arquivo `index.js`:

```js
const color = require('color')

const redHexa = color({ r: 255, g: 0, b: 0 }).hex()

console.log(redHexa)
```

🏁 Execute `yarn start`. Isso deve imprimir `#FF0000`.

Parabéns, você instalou e usou um pacote!

`color` somente foi usado nessa sessão para lhe ensinar como usar um pacote simples. Nós não precisaremos dele mais, então você pode desinstalá-lo:

- Execute `yarn remove color`

## Dois tipos de dependências

Há dois tipos de dependências de pacotes, `"dependencies"` e `"devDependencies"`:

**Dependencies** são bibliotecas que você precisa usar para que sua aplicação execute (React, Redux, Lodash, jQuery, etc). Você instala elas com `yarn add [package]`.

**Dev Dependencies** são bibliotecas usadas durante o desenvolvimento ou para compilar sua aplicação (Webpack, SASS, linters, testing frameworks, etc). Você instala elas com `yarn add --dev [package]`.

Próxima sessão: [02 - Babel, ES6, ESLint, Flow, Jest, Husky](02-babel-es6-eslint-flow-jest-husky.md#readme)

De volta ao [índice](https://github.com/verekia/js-stack-from-scratch#table-of-contents).
