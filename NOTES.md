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
    - [Configure package source maps](#configure-package-source-maps)
      - [Fix Problem, debugging package ts files and add source maps](#fix-problem-debugging-package-ts-files-and-add-source-maps)
      - [Fix Problem of debugging without environment variables](#fix-problem-of-debugging-without-environment-variables)
    - [Fix npm run build on nestjs-package-jwt-authentication-consumer](#fix-npm-run-build-on-nestjs-package-jwt-authentication-consumer)
    - [Commit project](#commit-project)
    - [Publish NPM package](#publish-npm-package)
    - [Publish component to private registry](#publish-component-to-private-registry)
  - [Useful stuff implemented in other based package projects](#useful-stuff-implemented-in-other-based-package-projects)
    - [Use Environment variables](#use-environment-variables)
  - [Problems](#problems)
    - [Package.json "dist/test.js"](#packagejson-disttestjs)
    - [Error: Cannot find module 'reflect-metadata'](#error-cannot-find-module-reflect-metadata)
  - [Use custom HttpExceptionFilter in custom nest js packages](#use-custom-httpexceptionfilter-in-custom-nest-js-packages)
    - [NestJs Exception filters](#nestjs-exception-filters)
    - [Solution(s)](#solutions)

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
```

```json
{
  "name": "@koakh/nestjs-package-yaml-config",
  "description": "Koakh NestJS YAML Config Package",
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
$ curl localhost:3000
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

lauch attach debugger with `F5` and it will stop in both breakpoint's, package and consumer

> Notice that in terminal window 2, since we're linked to the local package, the dev server will automatically restart as the package is rebuilt :)

```shell
$ curl localhost:3000
Buon Giorno!
```

### Configure package source maps

to debug with ts files we must do some magic

first edit package `nestjs-package-starter/tsconfig.json` and enable sourceMap's

change `"sourceMap": false` to `"sourceMap": true`

`nestjs-package-starter/tsconfig.json`

```json
{
  "compilerOptions": {
    "sourceMap": false,
  }
}
```

now when we build package we have new map files

- `nestjs-package-starter/tsconfig.json`
- `nestjs-package-starter/dist/test.js.map`
- `nestjs-package-starter/dist/index.js.map`

#### Fix Problem, debugging package ts files and add source maps

```shell
Could not load source '/media/mario/Storage/Documents/Development/@CriticalLinksBitBucket/c3next/src/test.ts': Unable to retrieve source content.
```

> Note: to fix this in a elegant way, we must create a `.vscode/launch.json` on root of `workspaceFolder`, and it starts to work, seems that the trick is back on dir, and debug in workspaceFolder and not in `nestjs-package-starter-consumer` folder

add `.vscode/launch.json`

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": [
        "<node_internals>/**"
      ],
      "program": "${workspaceFolder}/nestjs-package-starter-consumer/src/main.ts",
      "outFiles": [
        "${workspaceFolder}/**/*.js"
      ],
      "outputCapture": "std"
    }
  ]
}
```

Note: `"outputCapture": "std"` to show output in **debug console** tab

now launch F5 it works without issues, great, we land in `debugger;`

`nestjs-package-starter/src/test.ts` at debugger directive

```typescript
export function getHello(): string {
  debugger;
  return 'Buon Giorno!';
}
```

now we have our development environment ready to roll

#### Fix Problem of debugging without environment variables

side note, this seems not working with nestjs authentication package `nestjs-package-jwt-authentication` project, it won't work, after some debug like launch debug with F5 and both projects `nestjs-package-starter` and `nestjs-package-jwt-authentication` and change to vscode `debug console tab`, noted that it fails with below output

```shell
# nestjs-package-starter
/usr/local/bin/node ./nestjs-package-starter-consumer/dist/main.js
ok

# nestjs-package-jwt-authentication
/usr/local/bin/node ./nestjs-package-jwt-authentication-consumer/src/main.ts
Process exited with code 1
null: Uncaught /media/mario/Storage/Documents/Development/Node/@NestJsPackages/TypescriptNestJsPackageJwtAuthentication/nestjs-package-jwt-authentication-consumer/src/main.ts:1
import { NestFactory } from '@nestjs/core';
       ^

SyntaxError: Unexpected token {
KO
```

first we can't launch `/usr/local/bin/node ./nestjs-package-jwt-authentication-consumer/src/main.ts`, it seems that dist folder is missing, the right command is `/usr/local/bin/node ./nestjs-package-jwt-authentication-consumer/dist/main.js`

to debug this problem, we build `nestjs-package-jwt-authentication-consumer`, and with `dist` folder builded we can proceeed

```shell
# build dist folder
$ cd nestjs-package-jwt-authentication-consumer/
$ npm run build
$ cd ..
# try launch the command to catch the real problem
$ /usr/local/bin/node ./nestjs-package-jwt-authentication-consumer/dist/main.js
[Nest] 5748   - 10/07/2020, 3:08:36 PM   [ExceptionHandler] JwtStrategy requires a secret or key +2ms
TypeError: JwtStrategy requires a secret or key
```

it seems that we miss the `.env` file, add it to `launch.json`

```json
{
  ...
  "configurations": [
    {
      "program": "${workspaceFolder}/nestjs-package-jwt-authentication-consumer/src/main.ts",
      "outFiles": [
        "${workspaceFolder}/**/*.js"
      ],
      "envFile": "${workspaceFolder}/nestjs-package-jwt-authentication-consumer/.env"
    }
  ]
}
```

test with

```shell
$ curl -X POST http://localhost:3000/auth/login -d '{"username": "admin", "password": "12345678"}' -H "Content-Type: application/json"
```

now launch debugger with F5 and we are done, great we have debug ts, one more step to glory

another side note, seems that `debugger;` won't work and **breakpoints work**, better

### Fix npm run build on nestjs-package-jwt-authentication-consumer

- [Error “Cannot write file … because it would overwrite input file.”](https://medium.com/@salifyataala/error-cannot-write-file-because-it-would-overwrite-input-file-e17e1deba5e3)

```shell
$ npm run build
Debugger listening on ws://127.0.0.1:37661/8ae4d366-0073-4636-b17c-4519e1893250
For help, see: https://nodejs.org/en/docs/inspector
Debugger attached.

> @koakh/nestjs-package-jwt-authentication-consumer@1.0.0 build /media/mario/Storage/Documents/Development/Node/@NestJsPackages/TypescriptNestJsPackageJwtAuthentication/nestjs-package-jwt-authentication-consumer
> tsc -p tsconfig.build.json

error TS5055: Cannot write file '/media/mario/Storage/Documents/Development/Node/@NestJsPackages/TypescriptNestJsPackageJwtAuthentication/nestjs-package-jwt-authentication-consumer/dist/app.controller.d.ts' because it would overwrite input file.
...
```

to fix edit `nestjs-package-jwt-authentication-consumer/tsconfig.build.json` and add `"dist"` to `"exclude"`

```json
{
  "extends": "./tsconfig.json",
  "exclude": ["node_modules", "test", "**/*spec.ts", "dist"]
}
```

done now we can build project :)

### Commit project

commit id [8b0737b](https://github.com/koakh/NestJsPackageStarter/commit/8b0737b24454bad1641c0182190824f1b2cc54aa)

```shell
$ git add .
[main 8b0737b] now we have our development environment ready to roll
```

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

### Publish component to private registry

```shell
# enter react-component-greeting path
$ cd react-component-greeting
# check package.json, id is first time change  "description" and other properties
$ code package.json
# @critical-links/react-component-greeting
$ npm publish
+ @critical-links/react-component-greeting@1.0.0
# done we have published our component
```









## Useful stuff implemented in other based package projects

### Use Environment variables

to use Environment variables, read [notes](https://github.com/koakh/NestJsPackageJwtAuthentication/blob/main/NOTES.md) from NestJsPackageJwtAuthentication

## Problems

### Package.json "dist/test.js"

problems detected in authentication package

```shell
$ npm run start
TSError: ⨯ Unable to compile TypeScript:
src/app.module.ts:4:28 - error TS2307: Cannot find module '@koakh/nestjs-package-jwt-authentication' or its corresponding type declarations.

4 import { AuthModule } from '@koakh/nestjs-package-jwt-authentication';
```

fuck the problem is this `"main": "dist/test.js"`, change to "main": `"dist/index.js"`

```json
{
  "name": "@koakh/nestjs-package-jwt-authentication",
  "version": "1.0.0",
  "description": "Koakh NestJS Jwt Authentication Package",
  "author": "Mário Monteiro <marioammonteiro@gmail.com>",
  "license": "MIT",
  "readmeFilename": "README.md",
  "main": "dist/test.js",
```

and re-install `npm i ../nestjs-package-jwt-authentication`

### Error: Cannot find module 'reflect-metadata'

```shell
 cd nestjs-package-jwt-authentication-consumer
$ npm run start
Error: Cannot find module 'reflect-metadata'
required in `nestjs-package-jwt-authentication`
Require stack:
- /media/mario/Storage/Documents/Development/Node/@NestJsPackages/TypescriptNestJsPackageJwtAuthentication/nestjs-package-jwt-authentication/node_modules/@nestjs/common/index.js
```

- [Cannot find module 'reflect-metadata](https://github.com/nestjs/nest/issues/1211)

It's a `peerDependency`, You need to install it alongside `rxjs` aswell, in package library `nestjs-package-jwt-authentication`

```shell
# install in package
$ cd nestjs-package-jwt-authentication
$ npm install --save reflect-metadata rxjs
```

## Use custom HttpExceptionFilter in custom nest js packages

we have some problems using `HttpExceptionFilter` inside our custom nestjs controllers, always receiving the default `{ "statusCode": 500, "message": "Internal server error" }`, event when we used exceptions like `throw new NotFoundException(`userId not found`);`, it seems that using `app.useGlobalFilters(new HttpExceptionFilter());` in `nestjs-package-starter-consumer/src/main.ts` won't work, explanation is:

### NestJs Exception filters

- [Exception filters](https://docs.nestjs.com/exception-filters)

> WARNING: The `useGlobalFilters()` method does not set up filters for gateways or hybrid applications.

> Global-scoped filters are used across the whole application, for every controller and every route handler. In terms of dependency injection, global filters registered from outside of any module (with `useGlobalFilters()` as in the example above) cannot inject dependencies since this is done outside the context of any module. In order to solve this issue, you can register a global-scoped filter directly from any module using the following construction:

`app.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```

### Solution(s)

1. we must have a custom `HttpExceptionFilter` filter at controller, controller method or **global-scoped filter directly from any module**

ex ate controller method `@UseFilters(new HttpExceptionFilter())`

```typescript
@Put(':id/password')
@UseGuards(JwtAuthGuard)
@UseFilters(new HttpExceptionFilter())
async updatePassword(
  @Param('id') id: string,
  @Body() updateUserPasswordDto: UpdateUserPasswordDto
): Promise<User> {
  return await this.userService.updatePassword(id, updateUserPasswordDto);
}
```

we used in our custom nestjs package in both modules with **global-scoped filter directly from any module**

- `nestjs-package-jwt-authentication/src/auth/auth.module.ts`
- `nestjs-package-jwt-authentication/src/user/user.module.ts`

```typescript
@Module({
  ...
  providers: [
    // register a global-scoped filter directly from any module
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
    AuthService, UserService, LocalStrategy, JwtStrategy],
    ...
})
```

and now we have custom `HttpExceptionFilter`

```shell
HTTP/1.1 401 Unauthorized
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 101
ETag: W/"65-HVww6OPX1UL3DhHrIRJHBd/YUb8"
Date: Fri, 09 Oct 2020 11:57:07 GMT
Connection: close

{
  "statusCode": 401,
  "path": "/auth/login",
  "timestamp": "2020-10-09T11:57:07.873Z",
  "error": "Unauthorized"
}
```

best practice is using same filter in main app modules too

- `nestjs-package-jwt-authentication-consumer/src/main.ts`

```typescript
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { HttpExceptionFilter } from '@koakh/nestjs-package-jwt-authentication/src/common/filters';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // pipes middleware
  app.useGlobalPipes(new ValidationPipe({ transform: true }));
  // global-scoped filter, custom nestjs packages use global-scoped filter directly in modules
  app.useGlobalFilters(new HttpExceptionFilter());

  await app.listen(3000);
}

bootstrap();
```
