# Modules and Bundling

If you need a refresher on modules and bundling in JavaScript, see:

- [javascript-notes/modules.md](https://github.com/jessicarush/javascript-notes/blob/master/modules.md)
- [javascript-notes/node_package_manager.md](https://github.com/jessicarush/javascript-notes/blob/master/node_package_manager.md)

## Table of contents

<!-- toc -->

- [Introduction](#introduction)
- [Output ES6 modules](#output-es6-modules)
- [type keyword in imports](#type-keyword-in-imports)
- [Bundlers](#bundlers)
- [Webpack](#webpack)
  * [Multiple configurations: web and node with external packages](#multiple-configurations-web-and-node-with-external-packages)
- [Esbuild](#esbuild)
- [Vite](#vite)

<!-- tocstop -->

## Introduction

The first thing to note regarding modules in TypeScript is that by default TypeScript is configured to compile to *CommonJS* (`module.exports`, `require()`) as opposed to *ES6* (`import`, `export`). 

You can see this when you initialize a `tsconfig.json` with `npx tsc --init`:

```json
{
  "compilerOptions": {
    /* Modules */
    "module": "commonjs", /* Specify what module code is generated. */
  }
}
```

This doesn't mean you have to use *CommonJS* style modules when writing your `.ts` files, it just means its going to compile to that. 

One thing that is slightly misleading is normally we would need to add `"type": "module"` in the `package.json` to use *ES6* `import`s, but this is not the case with TypeScript. If `"module": "commonjs"` is set in the `tsconfig.json` then we **don't** want `"type": "module"` in the `package.json.`. Repeat: you can still use `import`s and `export`s in your `.ts` files with this setup.

However, if you plan on using your compiled output in the browser:

```html
<body>
  <p>hello</p>
  <script src="./dist/main.js"></script>
</body>
```

It won't work, because in order to use *CommonJS* modules in the browser, you need a bundler. See my [summary table here](https://github.com/jessicarush/javascript-notes/blob/master/modules.md#summary-1).

The other option of course is to switch to *ES6* modules as the compiled output.

## Output ES6 modules

First change your `tsconfig.json`:

```json
{
  "compilerOptions": {
    /* Modules */
    "module": "ES6", /* Specify what module code is generated. */
  }
}
```

Then add to your `package.json`:

```json
{
  "type": "module",
}
```

Then your html might look like:

```html
<body>
  <p>hello</p>
  <script src="./dist/main.js" type="module"></script>
</body>
```

One last thing, ensure all your own module imports include the `.js` file extension. For some season TypeScript leaves it off when compiling and then the browser can find the file. Not sure way and too busy to investigate at this point.

```typescript
import { items } from "./utils.js";
```

## type keyword in imports

Note that when *importing* a type such as:

```typescript
export type User = {
    name: string,
    age: number
}

export function printUser(user: User) {
    console.log(`${user.name} is ${user.age} years old`)
}
```

While it is perfectly fine to do this:

```typescript
import { User, printUser } from './Utils';
```

You also have the option to use the `type` keyword:

```typescript
import { type User, printUser } from './Utils';
```

There are a couple of benefits to doing this:

- Clarity and Intent: By using type in the import, you're explicitly stating that `User` is a type and not a value (like a function or a variable). This can improve the readability of your code and make the intent clearer to anyone reviewing it.
- Tree Shaking and Bundling: In some build tools and bundling processes, explicitly importing types can assist in "tree shaking," which is the process of removing unused code in the final bundle. By specifying that `User` is a type, it becomes clear to the bundler that it can be stripped out, as types do not exist at runtime and are only used during type checking.

To use this as a rule, you should set this option in your `tsconfig.json`:

```json
{
  "compilerOptions": {
    /* Interop Constraints */
    "verbatimModuleSyntax": true,
  }
}
```

> TypeScript 5.0 introduces a new option called --verbatimModuleSyntax to simplify the situation. The rules are much simpler - any imports or exports without a type modifier are left around. Anything that uses the type modifier is dropped entirely. [source](https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax)

So I should note if you're not doing your export statements inline then you should add the `type` keyword to the export too:

```typescript
export { type User, printUser };
```

In the docs, they also say something about the `verbatimModuleSyntax` option not being compatible with CommonJS modules so I think they're saying to use `"module": "ES6"`, but it's not super clear.

## Bundlers 

The [State of JavaScript](https://stateofjs.com/en-US) surveys have a section on [build tools](https://2022.stateofjs.com/en-US/libraries/build-tools/) so you can get a sense of how many there are and the popularity. For example:

- webpack
- Parcel
- Gulp
- Rollup
- Browserify
- tsc CLI
- Rome
- Snowpack
- SWC
- esbuild
- Vite
- WMR
- Turbopack

Vite and esbuild seem to be teh post popular these days. Technically Vite isn't an actual bundler though. It uses esbuild and Rollup under the hood.

## Webpack 

```
npm install webpack webpack-cli ts-node ts-loader @types/webpack @types/node --save-dev
```

Note you don't need to have configured `src` as your `rootDir` in `tsconfig.json`, because webpack will be doing this now.

In the project dir create a `webpack.config.ts` file.

Note: The following config is for running in `node` and only works with `"module": "commonjs"` (`tsconfig.json`). 

```typescript
import { Configuration } from 'webpack';
import { resolve } from 'path';

const config: Configuration = {
  mode: 'none',
  entry: {
    bundle: './src/main.ts'
  },
  target: 'node',
  module: {
    rules: [
      {
        exclude: /node_modules/,
        use: {
          loader: 'ts-loader',
          options: {
            transpileOnly: true
          }
        }
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js']
  },
  output: {
    filename: '[name].js',
    path: resolve(__dirname, 'dist')
  }
};

export default config;
```

To build, just run `webpack`. It will create a `bundle.js`.

### Multiple configurations: web and node with external packages

Normally you would be building for the browser or for node, bot both, but just as an exercise, lets see how you could do both.

You will need need a second `tsconfig.json` for the web build. Call the second file `tsconfig.web.json` and have it extend the main `tsconfig.json` file:

```json
{
  "extends":"./tsconfig.json",
  "compilerOptions": {
    "module": "ES6"
  }
}
```

Then we will need two webpack config files, one for node and one for web.

**webpack.config.node.ts**:

```typescript
import { Configuration } from 'webpack';
import { resolve } from 'path';

const config: Configuration = {
  mode: 'none',
  entry: {
    'bundle-node': './src/main.ts'      // creates a bundle-node.js
  },
  target: 'node',                       // node!
  module: {
    rules: [
      {
        exclude: /node_modules/,
        use: {
          loader: 'ts-loader',
          options: {
            transpileOnly: true
          }
        }
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js']
  },
  output: {
    filename: '[name].js',
    path: resolve(__dirname, 'dist')
  }
};

export default config;
```

**webpack.config.web.ts**:

```typescript
import { Configuration } from 'webpack';
import { resolve } from 'path';

const config: Configuration = {
  mode: 'none',
  entry: {
    'bundle-web': './src/main.ts'          // creates a bundle-web.js
  },
  target: 'web',                           // web!
  module: {
    rules: [
      {
        exclude: /node_modules/,
        use: {
          loader: 'ts-loader',
          options: {
            transpileOnly: true,
            configFile: 'tsconfig.web.json' // ES6 modules config!
          }
        }
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js']
  },
  output: {
    filename: '[name].js',
    path: resolve(__dirname, 'dist')
  }
};

export default config;
```

To build, just run:

```
webpack --config webpack.config.node.ts
webpack --config webpack.config.web.ts
```

It will create a `bundle.js`.

Note: you can set these these build commands up as scripts in your `package.json`:

```json
{
  "scripts": {
    "build-node": "webpack --config webpack.config.node.ts",
    "build-web": "webpack --config webpack.config.web.ts"
  }
}
```

## Esbuild

```
npm install esbuild –save-exact --save-dev
```

Create a `build.js` file in the project root dir:

```JavaScript
const { build } = require('esbuild');

async function buildAll(){
    await build({
        entryPoints:['./src/main.ts'],
        bundle: true,
        platform: 'node',
        logLevel: 'info',
        outfile: 'dist/bundle-node.js'
    });
    
    await build({
        entryPoints:['./src/main.ts'],
        bundle: true,
        platform: 'browser',
        logLevel: 'info',
        outfile: 'dist/bundle-web.js'
    })
}

buildAll();
```

You can also use the `import` keyword, but then you need to save the file with `.mjs` extension.

> The .mjs file extension is used to explicitly mark a file as an ES Module in Node.js environments. This is particularly relevant in contexts where there's potential ambiguity about the module system being used (CommonJS or ES Modules), such as in a Node.js application.

```JavaScript
import * as esbuild from 'esbuild';

async function buildAll(){
    await esbuild.build({
        entryPoints:['./src/Main.ts'],
        bundle: true,
        platform: 'node',
        logLevel: 'info',
        outfile: 'dist/bundle-node.js'
    });
    
    await esbuild.build({
        entryPoints:['./src/Main.ts'],
        bundle: true,
        platform: 'browser',
        logLevel: 'info',
        outfile: 'dist/bundle-web.js'
    })
}

buildAll();
```

To build, just run the file with node:

```
node build.mjs
```

## Vite 

[Vite](https://vitejs.dev/guide/#trying-vite-online) has presets for create vanilla TypeScript or React + TypeScript projects using `npm create vite@latest` 

Note SWC..
