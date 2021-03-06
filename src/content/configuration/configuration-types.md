---
title: Configuration Types
sort: 3
contributors:
  - sokra
  - skipjack
  - kbariotis
  - simon04
  - fadysamirsadek
  - byzyk
  - EugeneHlushko
  - dhurlburtusa
  - anshumanv
---

Besides exporting a single config object, there are a few more ways that cover other needs as well.


## Exporting a Function

Eventually you will find the need to disambiguate in your `webpack.config.js` between [development](/guides/development) and [production builds](/guides/production). You have (at least) two options:

One option is to export a function from your webpack config instead of exporting an object. The function will be invoked with two arguments:

- An environment as the first parameter. See the [environment options CLI documentation](/api/cli/#environment-options) for syntax examples.
- An options map (`argv`) as the second parameter. This describes the options passed to webpack, with keys such as [`output-filename`](/api/cli/#output-options) and [`optimize-minimize`](/api/cli/#optimize-options).

```diff
-module.exports = {
+module.exports = function(env, argv) {
+  return {
+    mode: env.production ? 'production' : 'development',
+    devtool: env.production ? 'source-maps' : 'eval',
     plugins: [
       new TerserPlugin({
         terserOptions: {
+          compress: argv['optimize-minimize'] // only if -p or --optimize-minimize were passed
         }
       })
     ]
+  };
};
```


## Exporting a Promise

webpack will run the function exported by the configuration file and wait for a Promise to be returned. Handy when you need to asynchronously load configuration variables.

T> It is possible to export multiple promises by wrapping them into `Promise.all([/* Your promises */]).`

```js
module.exports = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({
        entry: './app.js',
        /* ... */
      });
    }, 5000);
  });
};
```

W> Returning a `Promise` only works when using webpack via CLI. [`webpack()`](/api/node/#webpack) expects an object.


## Exporting multiple configurations

Instead of exporting a single configuration object/function, you may export multiple configurations (multiple functions are supported since webpack 3.1.0). When running webpack, all configurations are built. For instance, this is useful for [bundling a library](/guides/author-libraries) for multiple [targets](/configuration/output/#outputlibrarytarget) such as AMD and CommonJS:

```js
module.exports = [{
  output: {
    filename: './dist-amd.js',
    libraryTarget: 'amd'
  },
  name: 'amd',
  entry: './app.js',
  mode: 'production',
}, {
  output: {
    filename: './dist-commonjs.js',
    libraryTarget: 'commonjs'
  },
  name: 'commonjs',
  entry: './app.js',
  mode: 'production',
}];
```

T> If you pass a name to [`--config-name`](/api/cli/#config-options) flag, webpack will only build that specific configuration.
