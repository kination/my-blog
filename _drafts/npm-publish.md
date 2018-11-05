---
layout: post
title:  NPM publish
date:   2018-11-12
description: 
tags:
- npm
- package
- nodejs
- ci
permalink: npm-publish
---





## ...

This is the last package setting:
```json
{
  "name": "tucson",
  "version": "0.1.1",
  "description": "Convert JSON key/values to make it fit on your project",
  "main": "lib/index.js",
  "scripts": {
    "test": "mocha",
    "build": "tsc",
    "prepublishOnly": "yarn run test && yarn run build"
  },
  "keywords": [
    "tucson",
    "json",
    "convert"
  ],
  "author": "Dennis Jung <inylove82@gmail.com>",
  "repository": "https://github.com/djKooks/tucson",
  "license": "ISC",
  "devDependencies": {
    // ...
  },
  "dependencies": {
    // ...
  }
}
```


## and Release!
```
$ npm publish

> tucson@0.1.1 prepublishOnly .
> yarn run test && yarn run build

yarn run v1.7.0
$ mocha


  to camelcase
    check key
      ✓ key should be changed as camelcase
    check makeDate
      ✓ value with `makeDate` should be converted as Moment


  2 passing (22ms)

✨  Done in 0.67s.
yarn run v1.7.0
$ tsc
✨  Done in 2.96s.
npm notice
npm notice 📦  tucson@0.1.1
npm notice === Tarball Contents ===
npm notice 682B  package.json
npm notice 709B  README.md
npm notice 293B  tsconfig.json
npm notice 5.6kB yarn.lock
npm notice 151B  lib/config.d.ts
npm notice 330B  lib/config.js
npm notice 52B   lib/index.d.ts
npm notice 236B  lib/index.js
npm notice 99B   lib/jsonOption.d.ts
npm notice 77B   lib/jsonOption.js
npm notice 59B   lib/snakeCase.d.ts
npm notice 490B  lib/snakeCase.js
npm notice 234B  lib/tucson.d.ts
npm notice 1.6kB lib/tucson.js
npm notice 65B   lib/types.d.ts
npm notice 77B   lib/types.js
npm notice === Tarball Details ===
npm notice name:          tucson
npm notice version:       0.1.0
npm notice package size:  4.4 kB
npm notice unpacked size: 10.7 kB
npm notice shasum:        d91ca07f1f78aa4a5192b8c5a66e8f99219b8924
npm notice integrity:     sha512-azW6hamx99C82[...]KySN/WYb+1BDw==
npm notice total files:   16
npm notice
+ tucson@0.1.1
```


## Reference
* <https://www.npmjs.com>
* <https://hackernoon.com/how-to-publish-your-package-on-npm-7fc1f5aae600>
