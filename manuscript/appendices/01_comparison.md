# Comparación de herramientras de Construcción

Anteriormente, era suficiente concatenar los scripts juntos. Sin embargo, los tiempos han cambiado, y ahora distribuir el código JavaScript puede ser un esfuerzo complicado. Este problema se ha intensificado con el aumento de las aplicaciones de página única (AsPU). Ellas suelen confiar en muchas librerias más robustas. 

Por esta razón, existen múltiples estrategias sobre cómo cargarlas. Se podrían cargar todas a la vez o considerar cargarlas a medida que vayan siendo necesarias. El paquete web apoya muchos de estos tipos de estrategias.  

La popularidad de Node y [npm](https://www.npmjs.com/), es el administrador de paquete, que provee más contexto. Antes de que npm se hiciera más popular, era difícil consumir dependencias. Hubo un periodo donde las personas desarrollaron administradores de paquetes de específicos, pero al final npm ganó. Ahora el manejo de dependencias es mucho más fácil que antes, aunque aún hay desafíos por superar.

## Corredores de Tareas y Conjuntos de Paquetes

Historicamente hablando, han existido muchas herramientas. El más conocido es tal vez *Make*, y es todavía una opción viable. *Corredores de Tareas* especializados, tales como Grunt y Gunt fueron creados particurlarmente con desarrolladores JavaScript en mente. Los complementos disponibles a través de npm hace que ambos corredores de tareas sean poderosos y extensibles. Es incluso posible usar npm `scripts` como un corredor de tareas. Eso es común, particularmente con paquete web.

Los corredores de tareas son grandes herramientas en un alto nivel. Te permiten realizar operaciones en una plataforma cruzada. El problema inicia cuando se necesita unir varios puntos fuertes y producir conjuntos de paquetes. *Empaquetadores*, como Browserify, Brunch o paquete web, existen por esta razón.

{salto de página}

Por un tiempo, [RequireJS](http://requirejs.org/) fue popular. La idea era proveer una definición de módulo asincrónico y construir sobre eso. El formato, AMD, se encuentra cubierto con mayor detalle luego en este capítulo. Afortunadamente, los estándares se han alcanzado, y RequireJS parece ahora más una curiosidad.

## Make

[Make](https://en.wikipedia.org/wiki/Make_%28software%29)  va mucho más atrás, dado que fue lanzado en 1977. Aunque es una herramienta vieja, se ha mantenido relevante. Make permite escribir tareas separadas para diferentes propósitos. Por ejemplo, se puede tener diferentes tareas para la creación de una producción de construcción, minificando el JavaScript o proceso de pruebas. Se puede encontrar la misma idea en muchas otras herramientas.

Aún cuando Makees mayormente usado con proyectos en C, no se encuentra atado a C de ninguna forma. James Coglan discute en detalle [how to use Make with JavaScript](https://blog.jcoglan.com/2014/02/05/building-javascript-projects-with-make/). Considera el código abreviado basado en la publicación de James de abajo:

**Archivo Make**

```makefile
PATH  := node_modules/.bin:$(PATH)
SHELL := /bin/bash

source_files := $(wildcard lib/*.coffee)
build_files  := $(source_files:%.coffee=build/%.js)
app_bundle   := build/app.js
spec_coffee  := $(wildcard spec/*.coffee)
spec_js      := $(spec_coffee:%.coffee=build/%.js)

libraries    := vendor/jquery.js

.PHONY: all clean test

all: $(app_bundle)

build/%.js: %.coffee
    coffee -co $(dir $@) $<

$(app_bundle): $(libraries) $(build_files)
    uglifyjs -cmo $@ $^

test: $(app_bundle) $(spec_js)
    phantomjs phantom.js

clean:
    rm -rf build
```

 Con Make, se modela las tareas usando la sintaxis específica Make- y los terminales de comando haciendo posible integrarlo con paquete web.

## RequireJS

[RequireJS](http://requirejs.org/) was perhaps the first script loader that became genuinely popular. It gave the first proper look at what modular JavaScript on the web could be. Its greatest attraction was AMD. It introduced a `define` wrapper:

```javascript
define(["./MyModule.js"], function (MyModule) {
  return function() {}; // Export at module root
});

// or
define(["./MyModule.js"], function (MyModule) {
  return {
    hello: function() {...}, // Export as a module function
  };
});
```

{pagebreak}

Incidentally, it's possible to use `require` within the wrapper:

```javascript
define(["require"], function (require) {
  var MyModule = require("./MyModule.js");

  return function() {...};
});
```

This latter approach eliminates a part of the clutter. You still end up with code that feels redundant. ES2015 and other standards solve this.

T> Jamund Ferguson has written an excellent blog series on how to port from [RequireJS to webpack](https://gist.github.com/xjamundx/b1c800e9282e16a6a18e).

## npm `scripts` as a Task Runner

Even though npm CLI wasn't primarily designed to be used as a task runner, it works as such thanks to *package.json* `scripts` field. Consider the example below:

**package.json**

```json
"scripts": {
  "stats": "webpack --env production --json > stats.json",
  "start": "webpack-dev-server --env development",
  "deploy": "gh-pages -d build",
  "build": "webpack --env production"
},
```

These scripts can be listed using `npm run` and then executed using `npm run <script>`. You can also namespace your scripts using a convention like `test:watch`. The problem with this approach is that it takes care to keep it cross-platform.

Instead of `rm -rf`, you likely want to use utilities such as [rimraf](https://www.npmjs.com/package/rimraf) and so on. It's possible to invoke other tasks runners here to hide the fact that you are using one. This way you can refactor your tooling while keeping the interface as the same.

## Grunt

![Grunt](images/grunt.png)

[Grunt](http://gruntjs.com/) was the first popular task runner for frontend developers. Its plugin architecture contributed towards its popularity. Plugins are often complex by themselves. As a result, when configuration grows, it can become difficult to understand what's going on.

Here's an example from [Grunt documentation](http://gruntjs.com/sample-gruntfile). In this configuration, you define a linting and a watcher tasks. When the *watch* task gets run, it triggers the *lint* task as well. This way, as you run Grunt, you get warnings in real-time in the terminal as you edit the source code.

{pagebreak}

**Gruntfile.js**

```javascript
module.exports = grunt => {
  grunt.initConfig({
    lint: {
      files: ["Gruntfile.js", "src/**/*.js", "test/**/*.js"],
      options: {
        globals: {
          jQuery: true,
        },
      },
    },
    watch: {
      files: ["<%= lint.files %>"],
      tasks: ["lint"],
    },
  });

  grunt.loadNpmTasks("grunt-contrib-jshint");
  grunt.loadNpmTasks("grunt-contrib-watch");

  grunt.registerTask("default", ["lint"]);
};
```

In practice, you would have many small tasks for specific purposes, such as building the project. An important part of the power of Grunt is that it hides a lot of the wiring from you.

Taken too far, this can get problematic. It can become hard to understand what's going on under the hood. That's the architectural lesson to take from Grunt.

T> [grunt-webpack](https://www.npmjs.com/package/grunt-webpack) plugin allows you to use webpack in a Grunt environment while you leave the heavy lifting to webpack.

## Gulp

![Gulp](images/gulp.png)

[Gulp](http://gulpjs.com/) takes a different approach. Instead of relying on configuration per plugin, you deal with actual code. If you are familiar with Unix and piping, you'll like Gulp. You have *sources* to match files, *filters* to operate on these sources, and *sinks* to pipe the build results.

Here's an abbreviated sample *Gulpfile* adapted from the project's README to give you a better idea of the approach:

**Gulpfile.js**

```javascript
const gulp = require("gulp");
const coffee = require("gulp-coffee");
const concat = require("gulp-concat");
const uglify = require("gulp-uglify");
const sourcemaps = require("gulp-sourcemaps");
const del = require("del");

const paths = {
  scripts: ["client/js/**/*.coffee", "!client/external/**/*.coffee"],
};

// Not all tasks need to use streams.
// A gulpfile is another node program
// and you can use all packages available on npm.
gulp.task("clean", () => del(["build"]));
gulp.task("scripts", ["clean"], () =>
  // Minify and copy all JavaScript (except vendor scripts)
  // with sourcemaps all the way down.
  gulp
    .src(paths.scripts)
    // Pipeline within pipeline
    .pipe(sourcemaps.init())
    .pipe(coffee())
    .pipe(uglify())
    .pipe(concat("all.min.js"))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest("build/js"))
);
gulp.task("watch", () => gulp.watch(paths.scripts, ["scripts"]));

// The default task (called when you run `gulp` from CLI).
gulp.task("default", ["watch", "scripts"]);
```

Given the configuration is code, you can always hack it if you run into troubles. You can wrap existing Node packages as Gulp plugins, and so on. Compared to Grunt, you have a clearer idea of what's going on. You still end up writing a lot of boilerplate for casual tasks, though. That is where newer approaches come in.

T> [webpack-stream](https://www.npmjs.com/package/webpack-stream) allows you to use webpack in a Gulp environment.

{pagebreak}

## Browserify

![Browserify](images/browserify.png)

Dealing with JavaScript modules has always been a bit of a problem. The language itself didn't have the concept of modules till ES2015. Ergo, the language was stuck in the '90s when it comes to browser environments. Various solutions, including [AMD](http://requirejs.org/docs/whyamd.html), have been proposed.

[Browserify](http://browserify.org/) is one solution to the module problem. It allows CommonJS modules to be bundled together. You can hook it up with Gulp, and you can find smaller transformation tools that allow you to move beyond the basic usage. For example, [watchify](https://www.npmjs.com/package/watchify) provides a file watcher that creates bundles for you during development saving effort.

The Browserify ecosystem is composed of a lot of small modules. In this way, Browserify adheres to the Unix philosophy. Browserify is easier to adopt than webpack, and is, in fact, a good alternative to it.

T> [Splittable](https://www.npmjs.com/package/splittable) is a Browserify wrapper that allows code splitting, supports ES2015 out of the box, tree shaking, and more.

T> [transform-loader](https://www.npmjs.com/package/transform-loader) allows you to use Browserify transforms with webpack.

## JSPM

![JSPM](images/jspm.png)

Using [JSPM](http://jspm.io/) is quite different than previous tools. It comes with a command line tool of its own that is used to install new packages to the project, create a production bundle, and so on. It supports [SystemJS plugins](https://github.com/systemjs/systemjs#plugins) that allow you to load various formats to your project.

## Brunch

![Brunch](images/brunch.png)

Compared to Gulp, [Brunch](http://brunch.io/) operates on a higher level of abstraction. It uses a declarative approach similar to webpack's. To give you an example, consider the following configuration adapted from the Brunch site:

```javascript
module.exports = {
  files: {
    javascripts: {
      joinTo: {
        "vendor.js": /^(?!app)/,
        "app.js": /^app/,
      },
    },
    stylesheets: {
      joinTo: "app.css",
    },
  },
  plugins: {
    babel: {
      presets: [react", "env"],
    },
    postcss: {
      processors: [require("autoprefixer")],
    },
  },
};
```

Brunch comes with commands like `brunch new`, `brunch watch --server`, and `brunch build --production`. It contains a lot out of the box and can be extended using plugins.

T> There is an experimental [Hot Module Reloading runtime](https://www.npmjs.com/package/hmr-brunch) for Brunch.

## Webpack

![webpack](images/webpack.png)

You could say [webpack](https://webpack.js.org/) takes a more monolithic approach than Browserify. Whereas Browserify consists of multiple small tools, webpack comes with a core that provides a lot of functionality out of the box.

Webpack core can be extended using specific *loaders* and *plugins*. It gives control over how it *resolves* the modules, making it possible to adapt your build to match specific situations and workaround packages that don't work correctly out of the box.

Compared to the other tools, webpack comes with initial complexity, but it makes up for this through its broad feature set. It's an advanced tool that requires patience. But once you understand the basic ideas behind it, webpack becomes powerful.

## Other Options

You can find more alternatives as listed below:

* [pundle](https://www.npmjs.com/package/pundle) advertises itself as a next generation bundler and notes particularly its performance.
* [Rollup](https://www.npmjs.com/package/rollup) focuses on bundling ES2015 code. *Tree shaking* is one of its selling points. You can use Rollup with webpack through [rollup-loader](https://www.npmjs.com/package/rollup-loader).
* [AssetGraph](https://www.npmjs.com/package/assetgraph) takes an entirely different approach and builds on top of HTML semantics making it ideal for [hyperlink analysis](https://www.npmjs.com/package/hyperlink) or [structural analysis](https://www.npmjs.com/package/assetviz). [webpack-assetgraph-plugin](https://www.npmjs.com/package/webpack-assetgraph-plugin) bridges webpack and AssetGraph together.
* [FuseBox](https://www.npmjs.com/package/fuse-box) is a bundler focusing on speed. It uses a zero-configuration approach and aims to be usable out of the box.
* [StealJS](https://stealjs.com/) is a dependency loader and a build tool which has focused on performance and ease of use.
* [Flipbox](https://www.npmjs.com/package/flipbox) wraps multiple bundlers behind a uniform interface.
* [Blendid](https://www.npmjs.com/package/blendid) is a blend of Gulp and bundlers to form an asset pipeline.

## Conclusion

Historically there have been a lot of build tools for JavaScript. Each has tried to solve a specific problem in its own way. The standards have begun to catch up and less effort is required around basic semantics. Instead, tools can compete on a higher level and push towards better user experience. Often you can use a couple of separate solutions together.

To recap:

* **Task runners** and **bundlers** solve different problems. You can achieve similar results with both but often it's best to use them together to complement each other.
* Older tools, such as Make or RequireJS, still have influence even if they aren't as popular in web development as they once were.
* Bundlers like Browserify or webpack solve an important problem and help you to manage complex web applications.
* A number of emerging technologies approach the problem from different angles. Sometimes they build on top of other tools and at times they can be used together.
