# TypeScript intro notes

TypeScript is a superset of JavaScript that adds *static typing* and other features to the language. It is designed to help developers write more robust and maintainable code for large-scale JavaScript applications, and it compiles down to plain JavaScript, which can be run in any environment that supports JavaScript.

*Static typing* is a feature in programming languages where variable types are explicitly declared and are checked at compile-time. This allows for errors related to type mismatches to be caught early in the development process, enhancing code reliability and maintainability before the program is run.

## Table of contents

<!-- toc -->

- [Install](#install)
- [tsconfig.json](#tsconfigjson)
- [Errors](#errors)
- [Compile](#compile)
- [ts-node](#ts-node)
- [@types/node](#typesnode)
- [// @ts-commands](#-ts-commands)

<!-- tocstop -->

## Install

You can install npm per project (`npm init`):

```
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

## ts-node 

```
npm install -g ts-node
```

This allows you to run TypeScript files directly in node without compiling them first.

```
ts-node filename.ts
  ```

## @types/node

```typescript
import { randomBytes } from 'crypto';
```

When using Node.js-specific modules like `crypto` in a TypeScript project, you need to install `@types/node` so that TypeScript can understand the types from these Node.js modules. This is a standard practice for using Node.js modules in TypeScript.

```
npm install @types/node --save-dev
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


