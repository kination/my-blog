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


One way to show your output as a developer, is releasing a module which offers specific function to help development. Most of modern programming language has its own package manager(Python-pip, Rust-cargo, Java-maven, ...), so it helps to release and introduce your stuff and make it simple to be used by engineers.
I once write a note about releasing project in [bower](https://bower.io/) package manager. But because of several problems, most of people suggest to go over to [npm](https://www.npmjs.com/), and even bower project itself suggest to do as that way.
So in this section, I'm going t


## Why I did this
When I was developing front-end side(React + TypeScript), back-end side was developed in PHP and Python. Our team wanted to keep front-end side code to follow [camelCase](https://en.wikipedia.org/wiki/Camel_case) rule, but parameters sent from back-end side was [snake_case](https://en.wikipedia.org/wiki/Snake_case). It forced some of codes to follow snake case, so I thought of wrapping the response data, and convert parameter style directly.

Above this, I also added type converter logic. It supports to change integer value `0 or 1` to boolean, and date string to `Date` type. Because our project is based on TypeScript, this became quite useful.


## So, it is useful?
This works are just focusing to reduce the code. It is small project, with 2~300 lines of code only, and no high-level mathematical logic. Somebody could think they can be do it by themself. But actually, more than half of the modules can be refused by same reason.

I really want to suggest that, don't become so serious for starting up. It will hesitate your work to create something, as I did. It is important to think the usefulness, but it does not need to be so great.

So if somebody ask about this 'Is it useful?', than I would say 'Not sure, but it could be to somebody.'


## Start from empty space
If you are JavaScript(or TypeScript) user, just start with `npm init` for now. This command will generate skeleton code for project.
```
$ npm init
...
...

Press ^C at any time to quit.
package name: (your-package)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /path/to/your-package/package.json:

{
  "name": "your-package",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

If you are TypeScript user, using `tsc` will help quick setting.
```
$ tsc --init
message TS6071: Successfully created a tsconfig.json file.
```

Now you will have `tsconfig.json` file for typescript compile, and `package.json` file for node project configuration. Look on `tsconfig` first.

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "declaration": true,
    "outDir": "./lib",
    "strict": true,
    "moduleResolution": "node",
    "suppressImplicitAnyIndexErrors": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "**/__tests__/*"]
}
```

This is configuration for compiling typescript files. TypeScript files for project will be included inside '/src' directory, so defined as `"include": ["src"]`. `compilerOptions/outDir` is the location which compiled JavaScript file will be located. It's your decision, but I setup here to be go on inside '/lib'.
Compiled result is a build output, not coded by developer. So it is better to be included in `.gitignore` file.


## Make some function
Let's make very simple thing here. This is function reversing letters of words inside phrase.
```typescript
export function reverseWordsInPhrase(phrase: string): string {
  let wordList = phrase.split(/\b\s+/)
  let reversed = wordList.map((word) => {
    return reverseWord(word)
  })

  return reversed.join(' ')
}

function reverseWord(word: string): string {
  let newString = ''
  for (let a = word.length - 1; a >= 0; a--) {
    newString += word[a]
  }

  return newString
}
```



## Release setup


This is the package setting:
```json
{
  "name": "tucson",
  "version": "0.2.1",
  "description": "Convert JSON key/values to make it fit on your project",
  "main": "lib/index.js",
  "scripts": {
    "test": "mocha",
    "lint": "tslint -p ./",
    "build": "tsc",
    "prepublishOnly": "yarn run build && yarn run lint && yarn run test"
  },
  "keywords": [
    "tucson",
    "json",
    "convert"
  ],
  "author": "Dennis Jung <inylove82@gmail.com>",
  "repository": "https://github.com/djKooks/tucson",
  "license": "MIT",
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

> tucson@0.2.1 prepublishOnly .
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
npm notice 📦  tucson@0.2.1
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
npm notice version:       0.2.0
npm notice package size:  4.4 kB
npm notice unpacked size: 10.7 kB
npm notice shasum:        d91ca07f1f78aa4a5192b8c5a66e8f99219b8924
npm notice integrity:     sha512-azW6hamx99C82[...]KySN/WYb+1BDw==
npm notice total files:   16
npm notice
+ tucson@0.2.1
```


## ...


## Reference
* <https://www.npmjs.com>
* <https://hackernoon.com/how-to-publish-your-package-on-npm-7fc1f5aae600>
