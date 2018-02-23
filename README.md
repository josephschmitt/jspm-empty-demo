# JSPM `@empty` Error Demo App

This repository contains an application (`jspm-empty-demo-app`) and two packages
(`jspm-empty-dependency` and `jspm-empty-other-dependency`) that together demonstrate a bug in
`jspm` when setting a dependency to map to `@empty`.

## Setup

1. `jspm-empty-demo-app` -- an application using `jspm` to manage dependencies and build a bundled
   javascript file meant to be loaded by the browser
2. `jspm-empty-dependency` -- an `npm` package that exports a function which displays a console.log.
   This package is depended on by `jspm-empty-demo-app`.
2. `jspm-empty-other-dependency` -- an locally-defined package inside the `jspm-dempty-demo-app`
   which is declared as a `peerDependency` in `jspm-empty-dependency`. The peer is imported and
   executed by `jspm-empty-dependency`. However, this package is not published to `npm`. It instead
   exists completely inside `jspm-empty-demo-app` and uses `jspm` config's path mapping to correctly
   route the dependency and load the file

## Steps to reproduce the bug

1. Check out this repository
2. cd into `jspm-empty-demo-app` and run `npm install`
3. Bundle the application by running `npm run bundle`

## Expected outcome

`jspm` successfully bundles the application

## Actual outcome

`jspm` fails to bundle the application with the following error:

```sh
> @joe-sh/jspm-empty-demo-app@1.0.0 prepublish /.../jspm-empty-demo/jspm-empty-demo-app
> jspm bundle src/index.js ${npm_package_main}

     Building the bundle tree for src/index.js...

err  Error on fetch for @empty/index.js at file:///.../jspm-empty-demo/jspm-empty-demo-app/public/@empty/index.js
        Loading npm:@joe-sh/jspm-empty-dependency@1.0.4/index.js
        Loading src/index.js
        Error: ENOENT: no such file or directory, open '/.../jspm-empty-demo/jspm-empty-demo-app/public/@empty/index.js'
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! @joe-sh/jspm-empty-demo-app@1.0.0 prepublish: `jspm bundle src/index.js ${npm_package_main}`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the @joe-sh/jspm-empty-demo-app@1.0.0 prepublish script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /.../.npm/_logs/2018-02-23T20_10_41_989Z-debug.log
```

## Observations

It seems there's a bug with the mapping of `jspm-empty-other-dependency` in the `package.json` of
`jspm-empty-dependency`. Instead of correctly mapping the dependency to the location in the
filesystem, it's actually renaming the `@joe-sh/jspm-empty-other-dependency` to `@empty`, which then
produces an `ENOENT` error since it cannot load `@empty/index.js`.
