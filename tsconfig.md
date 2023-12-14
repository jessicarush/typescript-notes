# tsconfig.json

For complete reference see:

- [TSConfig Reference](https://www.typescriptlang.org/tsconfig)
- [TSConfig base options](https://github.com/tsconfig/bases/tree/main/bases)
- [The TSConfig Cheat Sheet](https://www.totaltypescript.com/tsconfig-cheat-sheet)

## Table of contents

<!-- toc -->

- [Target and lib](#target-and-lib)
- [Set input and output directories](#set-input-and-output-directories)
- [Output ES6 module syntax](#output-es6-module-syntax)
- [Verbatim module syntax](#verbatim-module-syntax)
- [Include and exclude](#include-and-exclude)
- [skipLibCheck](#skiplibcheck)
- [declaration](#declaration)
- [sourcemap](#sourcemap)
- [decorators](#decorators)

<!-- tocstop -->

## Target and lib

Set the ECMAScript version for the output `.js` files. See [target](https://www.typescriptlang.org/tsconfig#target).

```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "es2016"
  }
}
```

While `target` determines the JavaScript version your TypeScript code is compiled down to, `lib` allows you to specify which libraries (or sets of features) you want TypeScript to recognize and use in your project. 

If you don't set the `lib` option, it gets automatically set based on your `target` option. 

You can use `lib` to add support for features that are not included in your `target`. For example, if your target is `"ES5"` but you want to use ES2015 collection features (like `Map` or `Set`), you can specify both high level and individual library components: 

```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "ES5",
    "lib": ["ES5", "ES2015.Collection"],
  }
}
```

You may “mix and match” target and lib settings as desired, but you could just set target for convenience.

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
    "module": "ES6",
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

## Include and exclude 

`include` is used to specify an array of filenames or patterns to include in the program. These filenames are resolved relative to the directory containing the `tsconfig.json` file. `incldue` and `exclude` are [top level options](https://www.typescriptlang.org/tsconfig) in the `tsconfig.json` file:

```json
{
  "compilerOptions": {
    /* ... */
  },
  "include": ["src/**/*", "tests/**/*"]
}
```

You can use `exclude` to specify an array of filenames or patterns that should be skipped when resolving `include`.

```json
{
  "compilerOptions": {
    /* ... */
  },
  "exclude": [
    "node_modules",
    "config.ts",
    "**/*.spec.ts"
  ]
}
```

## skipLibCheck

By default TypeScript will not type check external npm installed libraries, however, if you're having issues you could turn it on to check:

```json
{
  "compilerOptions": {
    /* Completeness */
    "skipLibCheck": false
  }
}
```

## declaration

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

## sourcemap 

Enables the generation of sourcemap files. These files allow debuggers and other tools to display the original TypeScript source code when actually working with the emitted JavaScript files. Source map files are emitted as `.js.map` (or `.jsx.map`) files next to the corresponding `.js` output file.

```json
{
  "compilerOptions": {
    /* Emit */
    "sourceMap": true
  }
}
```

This is extremely valuable during debugging because it allows you to see and debug your TypeScript code directly in the browser developer tools or Node.js debuggers, even though the runtime is executing JavaScript.

## decorators

To ensure you are using TypeScript 5 (stage 3) decorators. See [decorators.md](decorators.md). This is the default if not set.

```typescript
{
  "compilerOptions": {
    /* Language and Environment */
    "experimentalDecorators": false
  }
}
```
