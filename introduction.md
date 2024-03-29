# TypeScript introduction

TypeScript is a superset of JavaScript that adds *static typing* and other features to the language. It is designed to help developers write more robust and maintainable code for large-scale JavaScript applications, and it compiles down to plain JavaScript, which can be run in any environment that supports JavaScript.

*Static typing* is a feature in programming languages where variable types are explicitly declared and are checked at compile-time. This allows for errors related to type mismatches to be caught early in the development process, enhancing code reliability and maintainability before the program is run.

## Table of contents

<!-- toc -->

- [Install](#install)
- [tsconfig.json](#tsconfigjson)
- [Errors](#errors)
- [Compile](#compile)
- [ts-node](#ts-node)
- [watch](#watch)
- [@types/node](#typesnode)
- [// @ts-commands](#-ts-commands)
- [vscode](#vscode)
  * [navigation](#navigation)
  * [debugging](#debugging)
  * [eslint](#eslint)
- [Resources](#resources)

<!-- tocstop -->

## Install

You can install typescript using npm per project:

```
npm init -y
npm install typescript --save-dev
npx tsc --version
```

or globally: 

```
npm install -g typescript
tsc --version
```

## tsconfig.json

The `tsconfig.json` file specifies the *root files* and the *compiler options* required to compile the project. This configuration file allows developers to control how the TypeScript compiler behaves, defining settings such as the target ECMAScript version, module resolution, and various checks and constraints to enforce during the compilation process.

If TypeScript is installed at the project level run:

```
npx tsc --init
```

or if TypeScript is installed globally: 

```
tsc --init
``` 

This will initialize the `tsconfig.json` in your project folder.

See [tsconfig.md](tsconfig.md).

## Errors 

Vscode will warn you of many errors while you are writing, otherwise type errors will be caught during compile time.

## Compile

To run the compiler if TypeScript is installed in the project:

```
npx tsc filename.ts
npx tsc
```

To run the compiler if TypeScript is installed globally:

```
tsc filename.ts
tsc
```

This will create a `.js` file using the same name.

You can set this command up as a build script in your `package.json`:

```json
{
  "scripts": {
    "build": "npx tsc"
  },
}
```

## ts-node 

```
npm install -g ts-node
```

This allows you to run TypeScript files directly in node without compiling them first.

```
ts-node filename.ts
```

## watch

Run `npx tsc --help` to see the CLI options. One of them is `--watch` which will compile as you make changes to any of your source files:

```
npx tsc --watch
```

## @types/node

```typescript
import { randomBytes } from 'crypto';
```

When using Node.js-specific modules like `crypto` in a TypeScript project, you need to install `@types/node` so that TypeScript can understand the types from these Node.js modules. This is a standard practice for using Node.js modules in TypeScript.

```
npm install @types/node --save-dev
```

It looks like many external libraries will have `@types` package to install, for example:

```
npm install uuid @types/uuid
```

## // @ts-commands

TypeScript may offer you errors which you disagree with, in those cases you can ignore errors on specific lines by adding `// @ts-ignore` or `// @ts-expect-error` on the preceding line.

```typescript
type Testing = string;
// @ts-expect-error
let testing: Testing = true;
```

To enable TypeScript to raise errors in JavaScript files you would add: `// @ts-check` to the first line in your .js files.

Conversely, You can skip checking some files by adding a `// @ts-nocheck` comment to the first line of a file.

I've also seen comments disabling eslint and typescript=eslint, but I haven't looked into these yet:

```typescript
// eslint-disable-next-line @typescript-eslint/no-unused-vars
// eslint-disable-next-line @typescript-eslint/no-explicit-any
```

## vscode

See [TypeScript in Visual Studio Code](https://code.visualstudio.com/docs/languages/typescript).

### navigation 

- control-click function names to go to the definition 
- right-click on a function to find all references
- right-click on a function to change all occurrences
- right-click on a arg and choose `refactor` to do things like `extract to a const`.

Note, sometimes when you rename files, typescript will show an error in your `tsconfig.json`. To fix this, just reload the window. `Command + P`, then type `>reload`, choose `Developer: reload window`.

### debugging

To debug properly in vscode, you should have just the project root directory open as opposed to a workspace with many directories open in the sidebar.

You should also set `"sourceMap": true` in your `tsconfig.json`.

Open the `Run and Debug` tab in the sidebar.

Add breakpoints by clicking to the left of the line numbers. Sometimes it's buggy and you have to deselect and add breakpoint again.

It will say you should create a `launch.json`:

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug local file",
      "runtimeArgs": ["-r", "ts-node/register"],
      "args": ["${relativeFile}"],
      "env": {
        "request": "test"
      }
    }
  ]
}
```

**version: "0.2.0"**  
This specifies the version of the launch configuration file format. The version "0.2.0" is a standard format used by VSCode.

**configurations:**  
This is an array of configuration objects. Each object represents a specific debug configuration.

**type: "node"**  
This tells VSCode that the debug environment is Node.js.

**request: "launch"**  
This specifies the type of debug session. `"launch"` means VSCode will start the debugging session by launching a new instance of the Node.js application.

**name: "Debug local file"**  
A human-readable name for this debug configuration, shown in the VSCode launch configuration dropdown.

**runtimeArgs:**  
These are additional command-line arguments passed to the Node.js runtime. `-r ts-node/register`: This preloads the ts-node/register module, enabling runtime TypeScript support. It's useful when you're debugging TypeScript files directly without compiling them to JavaScript first.

**args:**  
These are the arguments passed to your program. `${relativeFile}`: This is a variable that refers to the currently active file in VSCode. So, this configuration will debug the currently opened file.

**env:**  
This allows you to set environment variables for the program.

**"request": "test":**  
Sets an environment variable named request with the value test. It's not a standard environment variable for Node.js and might be specific to your application's needs.


### eslint 

See the [eslint docs](https://eslint.org/docs/latest/use/getting-started).

```
npm init @eslint/config
```

You can add rules to your `eslintrc.json`.

You can create a script in your `package.json`:

```json
{
  "scripts": {
    "eslint": "npx eslint ."
  },
}
```

## Resources

- [TypeScript Docs (handbook)](https://www.typescriptlang.org/docs/handbook/intro.html) 
- [TypeScript Release Notes](https://devblogs.microsoft.com/typescript/)
- [React TypeScript cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/setup)

