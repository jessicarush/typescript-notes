# tsconfig.json

For complete reference see:

- [TSConfig Reference](https://www.typescriptlang.org/tsconfig)
- [The TSConfig Cheat Sheet](https://www.totaltypescript.com/tsconfig-cheat-sheet).

## Table of contents

<!-- toc -->

- [Target JavaScript version](#target-javascript-version)
- [Set input and output directories](#set-input-and-output-directories)
- [Output ES6 module syntax](#output-es6-module-syntax)
- [Verbatim module syntax](#verbatim-module-syntax)
- [Export type and function declarations](#export-type-and-function-declarations)

<!-- tocstop -->

## Target JavaScript version 

Set the ECMAScript version for the output `.js` files.

```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "es2016"
  }
}
```

## Set input and output directories

Use these options to specify the source and output directories:

```json
{
  "compilerOptions": {
    /* Modules */
    "rootDir": "./src",
    /* Emit */
    "outDir": "./dist"
  }
}
```

Note, this only works if you run `npx tsc`. If you run `npx tsc filename.ts`, you will have to navigate to `src` first and it will put the output in the same directory.

## Output ES6 module syntax

See [modules_and_bundling.md](modules_and_bundling.md).

```json
{
  "compilerOptions": {
    /* Modules */
    "module": "ES6", /* Specify what module code is generated. */
  }
}
```

## Verbatim module syntax 

See [modules_and_bundling.md](modules_and_bundling.md).

```json
{
  "compilerOptions": {
    /* Interop Constraints */
    "verbatimModuleSyntax": true,
  }
}
```

## Export type and function declarations

When you are creating a library or a package that will be used by other TypeScript projects, there is an option that tells the compiler to generate `.d.ts` files along with the compiled JavaScript files. These `.d.ts` files are known as declaration files or type definition files. They are used by TypeScript to perform type checking when other TypeScript projects import your module. 

In your `tsconfig.json`, set the following option:

```json
{
  "compilerOptions": {
    /* Emit */
    "declaration": true
  }
}
```

If you're working on a project where the code is not intended to be consumed as a library or package by other projects, you typically don't need to set the declaration option.