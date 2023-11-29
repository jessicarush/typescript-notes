# Creating an npm package

## Table of contents

<!-- toc -->

- [Setup](#setup)
- [Build and set main, types](#build-and-set-main-types)
- [.npmignore](#npmignore)
- [Pack](#pack)
- [Installing a local package](#installing-a-local-package)
- [Next steps](#next-steps)

<!-- tocstop -->

## Setup 

```
npm init -y
npm install typescript @types/node --save-dev
npx tsc --init
```

Add a build script to your `package.json`.

```json
{
  "scripts": {
    "build": "npx tsc"
  },
}
```

Set up your `src` directory.

Set up your `tsconfig.json` file, at the very least:

- target
- module
- rootDir
- outDir

In addition set `"declaration": true`.

> Tip: If you are using an external package, consider setting all references to that package in one file, then export/import the function that uses that package as needed.

## Build and set main, types 

Once your package is ready to be shared in another project, build it, then set the output index.js as your main file. In addition, add the typescript types file (generated from the `"declaration": true` option).

```json
{
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
}
```

## .npmignore

Use this file in the root directory to identify which files should be excluded from the package:

```json
src
tsconfig.json
notes.txt
node_modules
```

## Pack 

Run the command to create a tarball:

```
npm pack
```

It will create a `-1.0.0.tgz` file of your package in the root directory.

## Installing a local package

Let's say our package is in the parent directory of our project:

```
npm install ../my-package-1.0.0.tgz
```

## Next steps

If you want to publish your package on npm, here's some guides:

- [Creating and publishing scoped public packages](https://docs.npmjs.com/creating-and-publishing-scoped-public-packages)
- [Publish to npm](https://zellwk.com/blog/publish-to-npm/)