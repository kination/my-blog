---
layout: post
title:  NPM publish
date:   2018-12-21
description: 
tags:
- npm
- package
- nodejs
- ci
permalink: npm-publish
---


One way to show your output as a developer, is releasing a module which offers specific function to help development. Most of modern programming language has its own package manager(Python-pip, Rust-cargo, Java-mvn repo, ...), so it helps to release and introduce your stuff and make it simple to be used by engineers.
I once write a note about releasing project in [bower](https://bower.io/) package manager. But because of several problems, most of people suggest to go over to [npm](https://www.npmjs.com/), and even bower project itself suggest to do as that way.
So in this section, I'm going t


## Why I did this
When I was developing front-end side(React + TypeScript), back-end side was developed in PHP and Python. Our team wanted to keep front-end side code to follow [camelCase](https://en.wikipedia.org/wiki/Camel_case) rule, but parameters sent from back-end side was [snake_case](https://en.wikipedia.org/wiki/Snake_case). It forced some of codes to follow snake case, so I thought of wrapping the response data, and convert parameter style directly.


## So, it is useful?
I really want to suggest this
It is important to think the usefulness, but actually, thinking too deeply will hesitate your work to create something, as I did. It does not need to be so great.


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


## ...


## Reference
* <https://www.npmjs.com>
* <https://hackernoon.com/how-to-publish-your-package-on-npm-7fc1f5aae600>
