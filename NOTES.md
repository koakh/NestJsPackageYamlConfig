# NOTES

- [NOTES](#notes)
  - [Links](#links)
  - [TLDR](#tldr)
  - [Start NestJs NPM Package](#start-nestjs-npm-package)
    - [Clone and change metadata](#clone-and-change-metadata)
    - [Init Git/Remote Repository](#init-gitremote-repository)
    - [Create remote repository NestJsPackageStarter](#create-remote-repository-nestjspackagestarter)
    - [Install dependencies](#install-dependencies)
    - [Build the package](#build-the-package)
    - [Use the package in the test app](#use-the-package-in-the-test-app)
    - [Do some changes in package and test debugger](#do-some-changes-in-package-and-test-debugger)
  - [Implement YAML Service](#implement-yaml-service)
  - [Publish Package [WIP]](#publish-package-wip)
    - [Publish NPM package](#publish-npm-package)

## Links

- based on [Koakh/NestJsPackageStarter](https://github.com/koakh/NestJsPackageStarter), please follow read and notes from base repo

## TLDR

used node version `node/v12.8.1`

## Start NestJs NPM Package

### Clone and change metadata

```shell
$ git clone https://github.com/koakh/NestJsPackageStarter.git TypescriptNestJsPackageYamlConfig
$ cd TypescriptNestJsPackageYamlConfig
# change package name to @koakh/nestjs-package-yaml-config
$ code nestjs-package-starter/package.json
```npm install js-yaml
  "repository": {
    "url": "https://github.com/koakh/NestJsPackageYamlConfig"
  },
  "bugs": "https://github.com/koakh/NestJsPackageYamlConfig",
```

```shell
# change package name to @koakh/nestjs-package-yaml-config-consumer
$ code nestjs-package-starter-consumer/package.json
```

```json
{
  "name": "@koakh/nestjs-package-yaml-config-consumer",
  "description": "Koakh NestJS YAML Config Package Consumer App",
},
"dependencies": {
  "@koakh/nestjs-package-yaml": "file:../nestjs-package-starter",
},
```

### Init Git/Remote Repository

```shell
$ rm .git -R
$ code README.md
$ git init
$ git add .
$ git commit -am "first commit"
$ git remote add origin https://github.com/koakh/NestJsPackageYamlConfig.git
$ git branch -M main
$ git push -u origin main
```

### Create remote repository NestJsPackageStarter

```shell
git branch -M main
git remote add origin https://github.com/koakh/NestJsPackageStarter.git
git push -u origin main
```

### Install dependencies

```shell
$ cd nestjs-package-starter/
$ npm i
$ cd ../nestjs-package-starter-consumer/
$ npm i
```

### Build the package

Take a moment to poke around in the `nestjs-package-starter/nestjs-package-starter` folder

```shell
$ cd ../nestjs-package-starter
$ npm run build
# commit changes
$ git add . && git commit -am "finished dependencies update"
$ git push
```

### Use the package in the test app

The template package exports a single simple test function. Examine `code ../nestjs-package-starter/src/test.ts` to see it:

```typescript
export function getHello(): string {
  return 'Hello from the new package!';
}
```

Open `nestjs-package-starter-consumer/src/app.controller.ts` and chanhe import from `@koakh/nestjs-package-starter` to `@koakh/nestjs-package-yaml-config`

```shell
import { Controller, Get } from '@nestjs/common';
import { getHello } from '@koakh/nestjs-package-yaml-config';

@Controller()
export class AppController {
  @Get()
  getHello(): string {
    return getHello();
  }
}
```

In terminal window 1, start `nestjs-package-starter` (using `start:dev` is recommended here so we can make iterative changes):

```shell
# terminal 1
$ cd nestjs-package-starter
$ npm run start:dev
[4:32:02 PM] Starting compilation in watch mode...
# terminal 2
$ cd nestjs-package-starter-consumer
$ npm run start:debug
# done we have access to package
$ curl localhost:3010
Hello from the new package!
```

### Do some changes in package and test debugger

```shell
# in window 1 : nestjs-package-starter-consumer
$ npm run start:debug
# in window 2 : nestjs-package-starter
$ npm run start:dev
```

Make a simple change to the `nestjs-package-starter/src/test.ts` `getHello()` function exported by the package, change it to return 'Buon Giorno!'

> Note: use `npm run start:debug` and add a `breakpoint/debugger;`

```typescript
export function getHello(): string {
  debugger;
  return 'Buon Giorno!';
}
```

add a breakpoint to consumer app to

```typescript
@Controller()
export class AppController {
  @Get()
  getHello(): string {
    debugger;
    return getHello();
  }
}
```

launch attach debugger with `F5` and it will stop in both breakpoint's, package and consumer

> Notice that in terminal window 2, since we're linked to the local package, the dev server will automatically restart as the package is rebuilt

```shell
$ curl localhost:3010
Buon Giorno!
```

## Implement YAML Service


## Publish Package [WIP]

### Publish NPM package

ex for `TypescriptNestJsPackageJwtAuthentication`

```shell
$ cd nestjs-package-jwt-authentication
$ npm run build
# before publish check if we are connected to private registry
$ npm get registry
https://hub.critical-links.com:543/
# we are change to default npm public registry
# change to private registry
$ NPM_REGISTRY=https://registry.npmjs.org/
# change default npm registry
$ npm set registry $NPM_REGISTRY
# confirm changes
$ npm get registry
https://registry.npmjs.org/
# login
$ npm login
Logged in as koakh on https://registry.npmjs.org/.
# notes
configuration is saved in ~/.npmrc


# check package.json, id is first time change  "description" and other properties
$ code package.json
# @critical-links/react-component-greeting
$ npm publish
+ @koakh/nestjs-package-jwt-authentication@1.0.0
# update versions with patch, minor and major
$ npm version patch
v1.0.1
```
