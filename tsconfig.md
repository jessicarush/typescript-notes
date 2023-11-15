# tsconfig.json

See [The TSConfig Cheat Sheet](https://www.totaltypescript.com/tsconfig-cheat-sheet).

Set the ECMAScript version for the output `.js` files.

```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "es2016"
  }
}
```

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