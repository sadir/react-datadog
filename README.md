# Datadog Tracing in the browser

This repo is to replicate the behaviour described in the issue [here](https://github.com/rochdev/datadog-tracer-js/issues/37#issuecomment-346444521).

Steps taken:

* follow setup of a react-boilerplate application [here](https://github.com/react-boilerplate/react-boilerplate)
* Run `yarn add datadog-tracer`

```
yarn add v1.3.2
$ npm run npmcheckversion

> react-boilerplate@3.5.0 npmcheckversion /Users/morgansadr-hashemi/code/sadir/react-datadog
> node ./internals/scripts/npmcheckversion.js

[1/5] 🔍  Validating package.json...
[2/5] 🔍  Resolving packages...
[3/5] 🚚  Fetching packages...
[4/5] 🔗  Linking dependencies...
[5/5] 📃  Building fresh packages...
success Saved lockfile.
success Saved 18 new dependencies.
├─ @protobufjs/aspromise@1.1.2
├─ @protobufjs/base64@1.1.2
├─ @protobufjs/codegen@2.0.4
├─ @protobufjs/eventemitter@1.1.0
├─ @protobufjs/fetch@1.1.0
├─ @protobufjs/float@1.0.2
├─ @protobufjs/inquire@1.1.0
├─ @protobufjs/path@1.1.2
├─ @protobufjs/pool@1.1.0
├─ @protobufjs/utf8@1.1.0
├─ @types/long@3.0.32
├─ @types/node@7.0.48
├─ datadog-tracer@0.4.0
├─ es6-promise@4.1.1
├─ long@3.2.0
├─ mersenne-twister@1.1.0
├─ opentracing@0.14.1
└─ protobufjs@6.8.0
$ npm run build:dll

> react-boilerplate@3.5.0 build:dll /Users/morgansadr-hashemi/code/sadir/react-datadog
> node ./internals/scripts/dependencies.js

Building the Webpack DLL...
Hash: 0b15acec436a53688b5d
Version: webpack 3.5.5
Time: 2313ms
                      Asset     Size  Chunks                    Chunk Names
reactBoilerplateDeps.dll.js  3.27 MB       0  [emitted]  [big]  reactBoilerplateDeps
chunk    {0} reactBoilerplateDeps.dll.js (reactBoilerplateDeps) 2.72 MB [entry] [rendered]

ERROR in ./node_modules/datadog-tracer/src/propagation/state.proto.js
Module not found: Error: Can't resolve 'protobuf' in '/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation'
 @ ./node_modules/datadog-tracer/src/propagation/state.proto.js 5:8-37
 @ ./node_modules/datadog-tracer/src/propagation/binary.js
 @ ./node_modules/datadog-tracer/src/tracer.js
 @ ./node_modules/datadog-tracer/browser.js
 @ dll reactBoilerplateDeps
✨  Done in 15.47s.
```

* Run `yarn add protobufjs`

```
yarn add v1.3.2
$ npm run npmcheckversion

> react-boilerplate@3.5.0 npmcheckversion /Users/morgansadr-hashemi/code/sadir/react-datadog
> node ./internals/scripts/npmcheckversion.js

[1/5] 🔍  Validating package.json...
[2/5] 🔍  Resolving packages...
[3/5] 🚚  Fetching packages...
[4/5] 🔗  Linking dependencies...
[5/5] 📃  Building fresh packages...
success Saved 1 new dependency.
└─ protobufjs@6.8.0
$ npm run build:dll

> react-boilerplate@3.5.0 build:dll /Users/morgansadr-hashemi/code/sadir/react-datadog
> node ./internals/scripts/dependencies.js

Building the Webpack DLL...
Hash: 30d927b1c38db848cc42
Version: webpack 3.5.5
Time: 2455ms
                      Asset     Size  Chunks                    Chunk Names
reactBoilerplateDeps.dll.js  3.48 MB       0  [emitted]  [big]  reactBoilerplateDeps
chunk    {0} reactBoilerplateDeps.dll.js (reactBoilerplateDeps) 2.91 MB [entry] [rendered]

ERROR in ./node_modules/datadog-tracer/src/propagation/state.proto.js
Module not found: Error: Can't resolve 'protobuf' in '/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation'
 @ ./node_modules/datadog-tracer/src/propagation/state.proto.js 5:8-37
 @ ./node_modules/datadog-tracer/src/propagation/binary.js
 @ ./node_modules/datadog-tracer/src/tracer.js
 @ ./node_modules/datadog-tracer/browser.js
 @ dll reactBoilerplateDeps
✨  Done in 13.27s.
```

* Make these changes to `app/app.js`

```
 import { ConnectedRouter } from 'react-router-redux';
 import createHistory from 'history/createBrowserHistory';
 import 'sanitize.css/sanitize.css';
+import 'protobufjs';
+import DatadogTracer from 'datadog-tracer';

 // Import root app
 import App from 'containers/App';
@@ -51,6 +53,12 @@ const history = createHistory();
 const store = configureStore(initialState, history);
 const MOUNT_NODE = document.getElementById('app');

+const tracer = new DatadogTracer({
+  service: 'example',
+  hostname: window.location.hostname,
+  port: 8080,
+});
+const span = tracer.startSpan('browser.render');
 const render = (messages) => {
   ReactDOM.render(
     <Provider store={store}>
@@ -63,6 +71,7 @@ const render = (messages) => {
     MOUNT_NODE
   );
 };
+span.finish();
```

* Run `yarn start`
* visit `localhost:3000` in your browser

Console output:
```
Uncaught Error: Cannot find module "protobuf"
    at webpackMissingModule (state.proto.js:5)
    at eval (state.proto.js:5)
    at eval (state.proto.js:13)
    at Object../node_modules/datadog-tracer/src/propagation/state.proto.js (reactBoilerplateDeps.dll.js:2707)
    at __webpack_require__ (reactBoilerplateDeps.dll.js:21)
    at eval (binary.js:10)
    at Object../node_modules/datadog-tracer/src/propagation/binary.js (reactBoilerplateDeps.dll.js:2700)
    at __webpack_require__ (reactBoilerplateDeps.dll.js:21)
    at eval (tracer.js:8)
    at Object../node_modules/datadog-tracer/src/tracer.js (reactBoilerplateDeps.dll.js:2755)
```

* Run `webpack --config internals/webpack/webpack.prod.babel.js --color -p --progress --display-error-details --display-optimization-bailout` just for more info

```
Hash: 0c5c6d33c8d076b4da52
Version: webpack 3.5.5
Time: 5188ms
                          Asset       Size  Chunks             Chunk Names
                 icon-96x96.png    8.11 kB          [emitted]
                  .htaccess.bin    1.79 kB          [emitted]
               icon-128x128.png    11.2 kB          [emitted]
               icon-144x144.png    12.7 kB          [emitted]
               icon-152x152.png    13.9 kB          [emitted]
               icon-192x192.png    17.9 kB          [emitted]
               icon-384x384.png      42 kB          [emitted]
               icon-512x512.png    16.7 kB          [emitted]
                 icon-72x72.png    5.89 kB          [emitted]
                    favicon.ico     370 kB          [emitted]
                  manifest.json  975 bytes          [emitted]
0.c0fa4a38b30f80091feb.chunk.js      41 kB       0  [emitted]
1.3dfb0094a8d5ec33d1b3.chunk.js    24.1 kB       1  [emitted]
2.389e8dada8506908429a.chunk.js    1.44 kB       2  [emitted]
3.4c0c7238a384d700a963.chunk.js    1.43 kB       3  [emitted]
   main.cc0b465e76a2d8878b6d.js    2.31 MB       4  [emitted]  main
                     index.html  558 bytes          [emitted]
                          sw.js    17.6 kB          [emitted]
[./app/app.js] ./app/app.js + 56 modules 145 kB {4} [built]
       ModuleConcatenation bailout: Module is referenced from these modules with unsupported syntax: multi ./app/app.js (referenced with single entry)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/babel-polyfill/lib/index.js (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/datadog-tracer/browser.js (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/.htaccess (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/favicon.ico (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-128x128.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-144x144.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-152x152.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-192x192.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-384x384.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-512x512.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-72x72.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-96x96.png (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/file-loader?name=[name].[ext]!./app/manifest.json (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/history/createBrowserHistory.js (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/protobufjs/index.js (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/react-dom/index.js (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/react-router-redux/index.js (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/react/react.js (<- Module is not an ECMAScript module)
       ModuleConcatenation bailout: Cannot concat with ./node_modules/sanitize.css/sanitize.css (<- Module is not an ECMAScript module)
[./app/translations/en.json] ./app/translations/en.json 19 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/.htaccess] ./node_modules/file-loader?name=[name].[ext]!./app/.htaccess 59 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/favicon.ico] ./node_modules/file-loader?name=[name].[ext]!./app/images/favicon.ico 57 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-128x128.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-128x128.png 62 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-144x144.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-144x144.png 62 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-152x152.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-152x152.png 62 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-192x192.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-192x192.png 62 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-384x384.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-384x384.png 62 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-512x512.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-512x512.png 62 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-72x72.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-72x72.png 60 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/images/icon-96x96.png] ./node_modules/file-loader?name=[name].[ext]!./app/images/icon-96x96.png 60 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/file-loader/index.js?name=[name].[ext]!./app/manifest.json] ./node_modules/file-loader?name=[name].[ext]!./app/manifest.json 59 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
[./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 509 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
   [0] multi ./app/app.js 28 bytes {4} [built]
       ModuleConcatenation bailout: Module is not an ECMAScript module
    + 677 hidden modules

ERROR in ./node_modules/datadog-tracer/src/propagation/state.proto.js
Module not found: Error: Can't resolve 'protobuf' in '/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation'
resolve 'protobuf' in '/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation'
  Parsed request is a module
  using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/package.json (relative path: ./src/propagation)
    Field 'browser' doesn't contain a valid alias configuration
  after using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/package.json (relative path: ./src/propagation)
    resolve as module
      /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation/app doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation/node_modules doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/app doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/node_modules doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/app doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/app doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/node_modules doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/app doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/sadir/node_modules doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/app doesn't exist or is not a directory
      /Users/morgansadr-hashemi/code/node_modules doesn't exist or is not a directory
      /Users/morgansadr-hashemi/app doesn't exist or is not a directory
      /Users/morgansadr-hashemi/node_modules doesn't exist or is not a directory
      /Users/app doesn't exist or is not a directory
      /Users/node_modules doesn't exist or is not a directory
      /app doesn't exist or is not a directory
      /node_modules doesn't exist or is not a directory
      looking for modules in /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules
        using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/package.json (relative path: ./node_modules)
          Field 'browser' doesn't contain a valid alias configuration
        after using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/package.json (relative path: ./node_modules)
          using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/package.json (relative path: ./node_modules/protobuf)
            no extension
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf doesn't exist
            .js
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf.js doesn't exist
            .jsx
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf.jsx doesn't exist
            .react.js
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf.react.js doesn't exist
            as directory
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf doesn't exist
      looking for modules in /Users/morgansadr-hashemi/code/sadir/react-datadog/app
        using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/package.json (relative path: ./app)
          Field 'browser' doesn't contain a valid alias configuration
        after using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/package.json (relative path: ./app)
          using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/package.json (relative path: ./app/protobuf)
            no extension
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf doesn't exist
            .js
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf.js doesn't exist
            .jsx
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf.jsx doesn't exist
            .react.js
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf.react.js doesn't exist
            as directory
              /Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf doesn't exist
      looking for modules in /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules
        using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/package.json (relative path: ./node_modules)
          Field 'browser' doesn't contain a valid alias configuration
        after using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/package.json (relative path: ./node_modules)
          using description file: /Users/morgansadr-hashemi/code/sadir/react-datadog/package.json (relative path: ./node_modules/protobuf)
            no extension
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf doesn't exist
            .js
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf.js doesn't exist
            .jsx
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf.jsx doesn't exist
            .react.js
              Field 'browser' doesn't contain a valid alias configuration
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf.react.js doesn't exist
            as directory
              /Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf doesn't exist
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation/app]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/propagation/node_modules]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/app]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/src/node_modules]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/app]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/app]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/node_modules]
[/Users/morgansadr-hashemi/code/sadir/app]
[/Users/morgansadr-hashemi/code/sadir/node_modules]
[/Users/morgansadr-hashemi/code/app]
[/Users/morgansadr-hashemi/code/node_modules]
[/Users/morgansadr-hashemi/app]
[/Users/morgansadr-hashemi/node_modules]
[/Users/app]
[/Users/node_modules]
[/app]
[/node_modules]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf.js]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf.js]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf.js]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf.jsx]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf.jsx]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf.jsx]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf.react.js]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf.react.js]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf.react.js]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/datadog-tracer/node_modules/protobuf]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/app/protobuf]
[/Users/morgansadr-hashemi/code/sadir/react-datadog/node_modules/protobuf]
 @ ./node_modules/datadog-tracer/src/propagation/state.proto.js 5:8-37
 @ ./node_modules/datadog-tracer/src/propagation/binary.js
 @ ./node_modules/datadog-tracer/src/tracer.js
 @ ./node_modules/datadog-tracer/browser.js
 @ ./app/app.js
 @ multi ./app/app.js

ERROR in main.cc0b465e76a2d8878b6d.js from UglifyJs
Unexpected token: name (Endpoint) [main.cc0b465e76a2d8878b6d.js:15203,6]
Child html-webpack-plugin for "index.html":
     1 asset
    [./node_modules/html-webpack-plugin/lib/loader.js!./app/index.html] ./node_modules/html-webpack-plugin/lib/loader.js!./app/index.html 502 bytes {0} [built]
           ModuleConcatenation bailout: Module is not an ECMAScript module
Child __offline_serviceworker:
     1 asset
       3 modules
```
