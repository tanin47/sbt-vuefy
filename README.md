sbt-vuefy
==========

[![CircleCI](https://circleci.com/gh/GIVESocialMovement/sbt-vuefy/tree/master.svg?style=shield)](https://circleci.com/gh/GIVESocialMovement/sbt-vuefy/tree/master)
[![codecov](https://codecov.io/gh/GIVESocialMovement/sbt-vuefy/branch/master/graph/badge.svg)](https://codecov.io/gh/GIVESocialMovement/sbt-vuefy)
[![Gitter chat](https://badges.gitter.im/GIVE-asia/gitter.png)](https://gitter.im/GIVE-asia/Lobby)

sbt-vuefy integrates Vue's single components into Playframework. It hot-reloads the changes of Vue components while running Playframework with `sbt run`. It also works with `sbt stage`, which triggers the production build.

Both Typescript and Javascript components are supported and can be mixed. Please see the example project in the folder `test-play-project`.

This plugin is currently used at [GIVE.asia](https://give.asia), which has more than 200 Vue components in both Javascript and Typescript.


Requirements
-------------

* __[Webpack 5.x](https://webpack.js.org/) and [vue-loader 16.x](https://github.com/vuejs/vue-loader):__ you'll need to specify the webpack binary location and webpack's configuration localtion. This enables you to choose your own version of Webpack and your own Webpack's configuration. You can see an example in the folder `test-play-project`.
* __Playframework 2.8.x__ 
* __Scala >= 2.12.x and SBT 1.x:__ Because the artifact is only published this setting (See: https://search.maven.org/artifact/io.github.givesocialmovement/sbt-vuefy/6.0.0/jar). If you would like other combinations of Scala and SBT versions, please open an issue.

How to use
-----------

### 1. Install the plugin

Add the below line to `project/plugins.sbt`:

For Playframework 2.8.x and Vue 3:
```
addSbtPlugin("io.github.tanin47" % "sbt-vuefy" % "6.0.0")
```

The artifacts are published to Maven Central here: https://search.maven.org/artifact/io.github.givesocialmovement/sbt-vuefy/6.0.0/jar

### 2. Configure Webpack config file.

Create `webpack.config.js` with `vue-loader`. Below is a working minimal example:

```
"use strict";

const {VueLoaderPlugin} = require('vue-loader');

module.exports = {
  plugins: [new VueLoaderPlugin()],
  stats: 'minimal',
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
      },
      {
        test: /\.scss$/,
        exclude: /node_modules/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ],
      },
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/],
        }
      }
    ]
  },
  resolve: {
    extensions: ['.ts', '.js', '.vue']
  }
};
```

You should NOT specify `module.exports.output` because sbt-vuefy will automatically set the field.

Your config file will be copied and added with some required additional code. Then, it will used by sbt-vuefy when compiling Vue components.

When running sbt-vuefy, we print the webpack command with the modified `webpack.config.js`, so you can inspect the config that we use.

To make it work with Typescript, `tsconfig.json` is also needed to be setup. Please see `test-play-project` for a working example.


### 3. Configure `build.sbt`

Specifying necessary configurations:

```
lazy val root = (project in file(".")).enablePlugins(PlayScala, SbtWeb, SbtVuefy) // Enable the plugin

// The commands that triggers production build when running Webpack, as in `webpack --mode production`.
Assets / VueKeys.vuefy / VueKeys.prodCommands := Set("stage")

// The location of the webpack binary. For windows, it might be `webpack.cmd`.
Assets / VueKeys.vuefy / VueKeys.webpackBinary := "./node_modules/.bin/webpack"

// The location of the webpack configuration.
Assets / VueKeys.vuefy / VueKeys.webpackConfig := "./webpack.config.js"
```

### 4. Find out where the output JS file is and how to use it

The plugin compiles `*.vue` within `app/assets`.

For the path `app/assets/vue/components/some-component.vue`, the output JS should be at `http://.../assets/vue/components/some-component.js`.
It should also work with `@routes.Assets.versioned("vue/components/some-component.js")`.

The exported module name is the camel case of the file name. In the above example, the module name is `SomeComponent`.

Therefore, we can use the component as shown below:

```
<script src="https://unpkg.com/vue@@3.0.4/dist/vue.runtime.global.js"></script>
<script src='@routes.Assets.versioned("vue/components/some-component.js")'></script>

<div id="app"></div>
<script>
  Vue
      .createApp({
        render: function() {
          return Vue.h(SomeComponent.default, {
            someProp: "SomePropValue"
          });
        }
      })
      .mount('#app');
</script>
```

Please see the folder `test-play-project` for a complete example.


Interested in using the plugin?
--------------------------------

Please feel free to open an issue to ask questions. Let us know how you want to use the plugin. We want to help you use the plugin successfully.


Contributing
---------------

The project welcomes any contribution. Here are the steps for testing when developing locally:

1. Run `yarn install` in order to install packages needed for the integration tests.
2. Run `sbt test` to run all tests.
3. To test the plugin on an actual Playframework project, go to `test-play-project`, run `yarn install`, and run `sbt run`.

Publish
--------
1. Get the latest master by running `git fetch`.
2. Run `sbt clean publishSigned`
3. Tag the current commit with the current version: `git tag -a v[VERSION] -m "Version v[VERSION]"` and `git push origin --tags`.


Future improvement
--------------------

* Currently, the plugin doesn't track CSS dependencies (e.g. using `@import`) because webpack/vue-loader doesn't track these dependencies. We need to find a way. See the ongoing issue: https://github.com/GIVESocialMovement/sbt-vuefy/issues/20
* `VueKeys.prodCommands` is hacky. I use this approach because I don't have good understanding in SBT's scoping. There must be a better way of implementing the production build setting.
