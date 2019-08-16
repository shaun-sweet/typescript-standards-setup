---
id: setup-and-configuration
title: Setup and configuration
---

## Assumptions

This guide assumes you have a build system capable of parsing Typescript.  Babel can now remove types on compilation which is what I suggest [Typescript Babel Preset](https://babeljs.io/docs/en/babel-preset-typescript)

## Installing Typescript 

```sh
yarn add typescript --dev
```
or npm
```sh
npm i -D typescript
```

## Adding a tsconfig.json

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "plugins": [],
    "baseUrl": "src",
    "outDir": "dist/",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "allowJs": true,
    "checkJs": false,
    "declaration": false,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "forceConsistentCasingInFileNames": true,
    "noEmitHelpers": true,
    "jsx": "preserve",
    "lib": [
      "dom",
      "es2017"
    ],
    "skipLibCheck": true,
    "target": "es5",
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "noEmit": true,
    "noEmitOnError": false,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "strict": true,
    "pretty": true,
    "removeComments": true,
    "sourceMap": true,
    "isolatedModules": true
  },
  "include": [
    "src",
    "typings"
  ]
}
```

## Change your file extensions to .tsx/.ts

In the root of your front end project run this in bash
```sh
tsify () {
  find $1 -name "*.js" -exec sh -c 'mv "$0" "${0%.js}.tsx"' {} \;
}

tsify .
```

This will change all your javascript files to `.tsx`.  I'd suggest manually changing as you go to .ts for non react specific javascript.  

// TODO: add options for peice meal additions instead of converting the entire project

## Installing common type definitions


```sh
yarn add @types/redux @types/react @types/react-dom @types/webpack @types/webpack-env 
@types/redux-thunk @types/history @types/react-router-dom --dev
```
or npm
```sh
npm i -D @types/redux @types/react @types/react-dom @types/webpack 
@types/webpack-env @types/redux-thunk @types/history @types/react-router-dom
```