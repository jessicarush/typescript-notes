# tsconfig.json

See [The TSConfig Cheat Sheet](https://www.totaltypescript.com/tsconfig-cheat-sheet).

## Table of contents

<!-- toc -->

- [Target JavaScript version](#target-javascript-version)
- [Set input and output directories](#set-input-and-output-directories)
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

## Export type and function declarations

If you are planning to use a type definition in another file, you can do the following: 

```typescript
export type Employee = {
  name: string;
  email: string;
  salary: number;
};


export function createEmployee(employeeName: string, salary: number): Employee {
  return {
    name: employeeName,
    email: `${employeeName}@company.com`,
    salary: salary
  };
}
```

In your `tsconfig.json`, set the following option:

```json
{
  "compilerOptions": {
    /* Emit */
    "declaration": true
  }
}
```

I'm not entirely clear on the purpose and usage of this so I'll have to test.