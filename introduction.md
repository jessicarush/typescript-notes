# TypeScript intro notes

TypeScript is a superset of JavaScript that adds *static typing* and other features to the language. It is designed to help developers write more robust and maintainable code for large-scale JavaScript applications, and it compiles down to plain JavaScript, which can be run in any environment that supports JavaScript.

*Static typing* is a feature in programming languages where variable types are explicitly declared and are checked at compile-time. This allows for errors related to type mismatches to be caught early in the development process, enhancing code reliability and maintainability before the program is run.

## Install

You can install npm per project:

```
npm install typescript --save-dev
npx tsc --version
```

or globally: 

```
npm install -g typescript
tsc --version
```

## Errors 

Vscode will warn you of many errors while you are writing, otherwise type errors will be caught during compile time.

## Compile

To run the compiler if TypeScript is installed in the project:

```
npx tsc filename.ts
```

To run the compiler if TypeScript is installed globally:

```
tsc filename.ts
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

## tsconfig.json

ChatGPT

The `tsconfig.json` file specifies the *root files* and the *compiler options* required to compile the project. This configuration file allows developers to control how the TypeScript compiler behaves, defining settings such as the target ECMAScript version, module resolution, and various checks and constraints to enforce during the compilation process.

Run `npx tsc --init` or `tsc --init` to initialize the `tsconfig.json` in your project folder.

See [tsconfig.md](tsconfig.md).



