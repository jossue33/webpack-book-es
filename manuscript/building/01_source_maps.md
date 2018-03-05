# Mapas fuente

![Source maps in Chrome](images/sourcemaps.png)

Cuando tu código fuente ha pasado por alguna transformación, depurar se convierte en un problema. Al hacer depuración en un navegador, ¿cómo decir dónde está el código original? **Source maps** resolver este problema proporcionando un mapeado entre el código fuente original y el modificado. En edición a la compilación fuente a JavaScript, esto funciona para diseño también.

Una aproximación es simplemente omitir los mapas fuente durante el desarrollo y confiar en el soporte del navegador de características de lenguaje. Si usas ES2015 sin ninguna extensión y desarrollas usando un navegador moderno, esto puede funcionar. La ventaja de hacer esto es que evitas todos los problemas relacionados a mapas fuente mientras ganas un mejor rendimiento.

T> Si quieres entender las ideas detras de los mapas fuente en gran detalle, [read Ryan Seddon's introduction to the topic](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/).

## Mapas fuente en línea y mapas fuente separados

Webpack puede generar mapas fuente, incluidos los que están dentro de paquetes o archivos de mapa fuente separados. Los primeros son importantes en el proceso de desarrollo debido al rendimiento mejorado mientras que los últimos son útiles para uso de producción mientras se mantenga pequeño el tamaño del paquete. En este caso, es opcional cargar los mapas fuente.

Es posible que tú **don't** quieras generar un mapa fuente para tu paquete de producción ya que esto hace que examinar tu aplicación sea fácil. Desactivándolas, estás realizando una especie de confusión. Quieras o no activar mapas fuente para producción, son útiles para preparar. Omitir mapas fuente completamente acelera tu versión al igual que generar mapas fuente en la mejor calidad puede ser una operación compleja.

**Hidden source maps** da solamente información de rastreo de pila. Puedes conectarlos con un servicio de monitoreo para conseguir rastros cuando la aplicación se cuelgue, permitiéndote arreglar las situaciones problemáticas. Si bien esto no es lo ideal, es mejor saber sobre los problemas posibles a que no.

T> Es una buena idea estudiar los manueales de los cargadores que estás usando para ver los consejos específicos del cargador. Por ejemplo, con TypeScript, tienes que poner una flag en particular para hacerlo funcionar como esperas.

## Activar mapas fuente

Webpack proporciona dos formas de activar los mapas fuente. Hay un acceso directo `devtool`. También puedes encontrar dos plugins que dan más opciones para modificar. Los plugins van a ser discutidos brévemente al final de este capítulo. Más allá de webpack, también tienes que activar el soporte para mapas fuente en los navegadores que estás usando para el desarrollo.

{pagebreak}

### Activando mapas fuente en Webpack

Para empezar, puedes envolver la idea principal dentro de una pieza de configuración. Puedes convertir esto para usar los plugins luego si quieres:

**webpack.parts.js**

```javascript
exports.generateSourceMaps = ({ type }) => ({
  devtool: type,
});
```

Webpack soporta una amplia variedad de tipos de mapas fuente. Estos varían, basados en la calidad y velocidad de la versión. Por ahora, Puedes activar `eval-source-map` para desarrollo y `source-map` para producción. De esta menera, obtienes buena calidad mientras sacrificas rendimiento, especialmente durante el dessarrollo.

Configúrelos de la siguiente manera:

**webpack.config.js**

```javascript
const productionConfig = merge([
leanpub-start-insert
  parts.generateSourceMaps({ type: "source-map" }),
leanpub-end-insert
  ...
]);

const developmentConfig = merge([
leanpub-start-insert
  {
    output: {
      devtoolModuleFilenameTemplate:
        "webpack:///[absolute-resource-path]",
    },
  },
  parts.generateSourceMaps({
    type: "cheap-module-eval-source-map"
  }),
leanpub-end-insert
  ...
]);
```

`eval-source-map` construye lentamente al inicio, pero provee de una rápida velocidad de reconstrucción. Más opciones rápidas de desarrollo específicas, tales como `cheap-module-eval-source-map` y `eval`, producen mapas fuente de menor calidad. Todas las opciones `eval` emiten mapas fuente como parte de tu código JavaScript.

`source-map` es la opción más lenta y de mayor calidad de todas, pero eso está bien para una construcción de producción.

Si construyes el proyecto ahora (`npm run build`), deberías ver mapas fuente en la salida:

```bash
Hash: 28afad28226615c16e12
Version: webpack 3.8.1
Time: 2006ms
        Asset       Size  Chunks                    Chunk Names
  ...font.eot     166 kB          [emitted]
...font.woff2    77.2 kB          [emitted]
 ...font.woff      98 kB          [emitted]
  ...font.svg     444 kB          [emitted]  [big]
  ...font.ttf     166 kB          [emitted]
       app.js    4.46 kB       0  [emitted]         app
      app.css    3.89 kB       0  [emitted]         app
leanpub-start-insert
   app.js.map    4.15 kB       0  [emitted]         app
  app.css.map   84 bytes       0  [emitted]         app
leanpub-end-insert
   index.html  218 bytes          [emitted]
   [0] ./app/index.js 160 bytes {0} [built]
   [3] ./app/main.css 41 bytes {0} [built]
   [4] ./app/component.js 275 bytes {0} [built]
...
```

Échale un buen vistazo a esos archivos *.map*. Ahí es donde el mapeado entre la fuente original y la generada ocurre. Durante el desarrollo, hace la información de mapeado en el paquete en sí mismo.

### Activar mapas fuente en navegadores

Para usar mapas fuente dentro de un navegador, tienes que activar los mapas fuente axplícitamente como instrucciones específicas de navegador:

* [Chrome](https://developer.chrome.com/devtools/docs/javascript-debugging). Sometimes source maps [will not update in Chrome inspector](https://github.com/webpack/webpack/issues/2478). For now, the temporary fix is to force the inspector to reload itself by using *alt-r*.
* [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)
* [IE Edge](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/f12-devtools-guide/debugger/#source-maps)
* [Safari](https://developer.apple.com/library/safari/documentation/AppleApplications/Conceptual/Safari_Developer_Guide/ResourcesandtheDOM/ResourcesandtheDOM.html#//apple_ref/doc/uid/TP40007874-CH3-SW2)

W> Si quieres usar puntos de quiebre (i.e., una declaración `debugger;` o las establecidas a través del navegador), ¡las opciones `eval`-based no funcionarán en Chrome!

## Tipos de mapa fuente soportados por Webpack

Los tipos de mapa fuente soportados por webpack pueden ser separados en dos categorías:

* **Inline** los mapas fuente añaden la información de mapeo directamente a los archivos generados.
* **Separate** los mapas fuente emiten la información de mapeo para separar los archivos del mapa fuente y enlazar la fuente original a ellos usando un comentario. Los mapas fuente escondidos omiten el comentario a propósito.

Gracias a su velocidad, mapas fuente en línea son ideales para desarrollar. Dado que hacen a los paquetes grandes, los mapas fuente separados son la opción preferible para producción. Los mapas fuente separados funcionan durante el desarrollo también si la sobrecarga de rendimiento es permitida.

{pagebreak}

## Tipos de mapas fuente en línea

Webpack provee múltiples variantes de mapas fuente en línea. A menudo, el punto de partida es `eval` y [webpack issue #2145](https://github.com/webpack/webpack/issues/2145#issuecomment-294361203) recomienda `cheap-module-eval-source-map` ya que es una buena promesa entre velocidad y calidad, trabajando confiablemente en los navegadores Chrome y Firefox.

Para tener una mejor idea de las opciones disponibles, están listadas abajo mientras se da un pequeño ejemplo de cada una. El código fuente condiente un solo `console.log('Hello world')` y `webpack.NamedModulesPlugin` se usa para mentener la salida más fácil de entender. En la práctica, verías mucho más código para manejar el mapeado.

T> `webpack.NamedModulesPlugin` reemplaza las ID de módulos basados en números con rutas. Eso es discutido en el capítulo *Adding Hashes to Filenames*.

### `devtool: "eval"`

`eval` genera código en el cual cada módulo es envuelto dentro de una función `eval`:

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');\n\n//////////////////\n// WEBPACK FOOTER\n// ./app/index.js\n// module id = ./app/index.js\n// module chunks = 1\n\n//# sourceURL=webpack:///./app/index.js?")
  }
}, ["./app/index.js"]);
```

{pagebreak}

### `devtool: "cheap-eval-source-map"`

`cheap-eval-source-map` va un paso adelante e incluye versiones codificadas base64 del código como una url de información. El resultado incluye sólo datos de línea mientras pierde mapeos de columna.

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/MGUwNCJdLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLy8vLy8vLy8vLy8vLy8vLy9cbi8vIFdFQlBBQ0sgRk9PVEVSXG4vLyAuL2FwcC9pbmRleC5qc1xuLy8gbW9kdWxlIGlkID0gLi9hcHAvaW5kZXguanNcbi8vIG1vZHVsZSBjaHVua3MgPSAxIl0sIm1hcHBpbmdzIjoiQUFBQSIsInNvdXJjZVJvb3QiOiIifQ==")
  }
}, ["./app/index.js"]);
```

Si decodificas esa cadena base64, obtienes salidas que contienen el mapeo:

```json
{
  "file": "./app/index.js.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///./app/index.js?0e04"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n//////////////////\n// WEBPACK FOOTER\n// ./app/index.js\n// module id = ./app/index.js\n// module chunks = 1"
  ],
  "version": 3
}
```

{pagebreak}

### `devtool: "cheap-module-eval-source-map"`

`cheap-module-eval-source-map` es la misma idea, exepto con una mayor calidad y un rendimiento más bajo:

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vYXBwL2luZGV4LmpzPzIwMTgiXSwic291cmNlc0NvbnRlbnQiOlsiY29uc29sZS5sb2coJ0hlbGxvIHdvcmxkJyk7XG5cblxuLy8gV0VCUEFDSyBGT09URVIgLy9cbi8vIGFwcC9pbmRleC5qcyJdLCJtYXBwaW5ncyI6IkFBQUEiLCJzb3VyY2VSb290IjoiIn0=")
  }
}, ["./app/index.js"]);
```

De nuevo, decodificar la información revela más:

```json
{
  "file": "./app/index.js.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///app/index.js?2018"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// app/index.js"
  ],
  "version": 3
}
```

En este caso en particular, la diferencia entre las opciones es mínima.

{pagebreak}

### `devtool: "eval-source-map"`

`eval-source-map` es la opción de mayor calidad de las que están en línea. También es la más lenta ya que es la que emite más información:

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/ZGFkYyJdLCJuYW1lcyI6WyJjb25zb2xlIiwibG9nIl0sIm1hcHBpbmdzIjoiQUFBQUEsUUFBUUMsR0FBUixDQUFZLGFBQVoiLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLyBXRUJQQUNLIEZPT1RFUiAvL1xuLy8gLi9hcHAvaW5kZXguanMiXSwic291cmNlUm9vdCI6IiJ9")
  }
}, ["./app/index.js"]);
```

Esta vez, hay más datos de mapeo disponibles para el navegador:

```json
{
  "file": "./app/index.js.js",
  "mappings": "AAAAA,QAAQC,GAAR,CAAY,aAAZ",
  "names": [
    "console",
    "log"
  ],
  "sourceRoot": "",
  "sources": [
    "webpack:///./app/index.js?dadc"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./app/index.js"
  ],
  "version": 3
}
```

{pagebreak}

## Tipos de mapas fuente separados

Webpack también puede generar mapas fuente amigables de uso de producción. Estos finalizan en archivos separados que terminan con la extensión `.map` y que son cargadas por el navegador sólo cuando es requerido. De esta forma, tus usuarios obtienen un buen rendimiento al mismo tiempo que es más fácil para ti depurar la aplicación.

`source-map` es un buen valor por defecto aquí. Aunque tarda más generar los mapas fuente de esta manera, obtienes la mejor calidad. Si no te importan los mapas fuente de producción, puedes simplemente omitir las configuraciones ahí y tener un mejor renidmiento a cambio.

### `devtool: "cheap-source-map"`

`cheap-source-map` is similar to the cheap options above. The result is going to miss column mappings. Also, source maps from loaders, such as *css-loader*, are not going to be used.

Examining the `.map` file reveals the following output in this case:

```json
{
  "file": "app.9aff3b1eced1f089ef18.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///app.9aff3b1eced1f089ef18.js"
  ],
  "sourcesContent": [
    "webpackJsonp([1,2],{\"./app/index.js\":function(o,n){console.log(\"Hello world\")}},[\"./app/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// app.9aff3b1eced1f089ef18.js"
  ],
  "version": 3
}
```

The original source contains `//# sourceMappingURL=app.9a...18.js.map` kind of comment at its end to map to this file.

### `devtool: "cheap-module-source-map"`

`cheap-module-source-map` is the same as previous except source maps from loaders are simplified to a single mapping per line. It yields the following output in this case:

```json
{
  "file": "app.9aff3b1eced1f089ef18.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///app.9aff3b1eced1f089ef18.js"
  ],
  "version": 3
}
```

W> `cheap-module-source-map` is [currently broken if minification is used](https://github.com/webpack/webpack/issues/4176) and this is a good reason to avoid the option for now.

### `devtool: "hidden-source-map"`

`hidden-source-map` is the same as `source-map` except it doesn't write references to the source maps to the source files. If you don't want to expose source maps to development tools directly while you want proper stack traces, this is handy.

T> [The official documentation](https://webpack.js.org/configuration/devtool/#devtool) contains more information about `devtool` options.

{pagebreak}

### `devtool: "source-map"`

`source-map` provides the best quality with the complete result, but it's also the slowest option. The output reflects this:

```json
{
  "file": "app.9aff3b1eced1f089ef18.js",
  "mappings": "AAAAA,cAAc,EAAE,IAEVC,iBACA,SAAUC,EAAQC,GCHxBC,QAAQC,IAAI,kBDST",
  "names": [
    "webpackJsonp",
    "./app/index.js",
    "module",
    "exports",
    "console",
    "log"
  ],
  "sourceRoot": "",
  "sources": [
    "webpack:///app.9aff3b1eced1f089ef18.js",
    "webpack:///./app/index.js"
  ],
  "sourcesContent": [
    "webpackJsonp([1,2],{\n\n/***/ \"./app/index.js\":\n/***/ (function(module, exports) {\n\nconsole.log('Hello world');\n\n/***/ })\n\n},[\"./app/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// app.9aff3b1eced1f089ef18.js",
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./app/index.js"
  ],
  "version": 3
}
```

{pagebreak}

## Other Source Map Options

There are a couple of other options that affect source map generation:

```javascript
{
  output: {
    // Modify the name of the generated source map file.
    // You can use [file], [id], and [hash] replacements here.
    // The default option is enough for most use cases.
    sourceMapFilename: '[file].map', // Default

    // This is the source map filename template. It's default
    // format depends on the devtool option used. You don't
    // need to modify this often.
    devtoolModuleFilenameTemplate:
      'webpack:///[resource-path]?[loaders]'
  },
}
```

T> The [official documentation](https://webpack.js.org/configuration/output/#output-sourcemapfilename) digs into `output` specifics.

W> If you are using any `UglifyJsPlugin` and still want source maps, you need to enable `sourceMap: true` for the plugin. Otherwise, the result isn't be what you expect because UglifyJS will perform a further transformation on the code, breaking the mapping. The same has to be done with other plugins and loaders performing transformations. *css-loader* and related loaders are a good example.

## `SourceMapDevToolPlugin` and `EvalSourceMapDevToolPlugin`

If you want more control over source map generation, it's possible to use the [SourceMapDevToolPlugin](https://webpack.js.org/plugins/source-map-dev-tool-plugin/) or `EvalSourceMapDevToolPlugin` instead. The latter is a more limited alternative, and as stated by its name, it's handy for generating `eval` based source maps.

Both plugins can allow more granular control over which portions of the code you want to generate source maps for, while also having strict control over the result with `SourceMapDevToolPlugin`. Using either plugin allows you to skip the `devtool` option altogether.

Given webpack matches only `.js` and `.css` files by default for source maps, you can use `SourceMapDevToolPlugin` to overcome this issue. This can be achieved by passing a `test` pattern like `/\.(js|jsx|css)($|\?)/i`.

`EvalSourceMapDevToolPlugin` accepts only `module` and `lineToLine` options as described above. Therefore it can be considered as an alias to `devtool: "eval"` while allowing a notch more flexibility.

## Changing Source Map Prefix

You can prefix a source map option with a **pragma** character that gets injected to the source map reference. Webpack uses `#` by default that is supported by modern browsers so you don't have to set it.

To override this, you have to prefix your source map option with it (e.g., `@source-map`). After the change, you should see `//@` kind of reference to the source map over `//#` in your JavaScript files assuming a separate source map type was used.

## Using Dependency Source Maps

Assuming you are using a package that uses inline source maps in its distribution, you can use [source-map-loader](https://www.npmjs.com/package/source-map-loader) to make webpack aware of them. Without setting it up against the package, you get minified debug output. Often you can skip this step as it's a special case.

## Source Maps for Styling

If you want to enable source maps for styling files, you can achieve this by enabling the `sourceMap` option. The same idea works with style loaders such as *css-loader*, *sass-loader*, and *less-loader*.

The *css-loader* is [known to have issues](https://github.com/webpack-contrib/css-loader/issues/232) when you are using relative paths in imports. To overcome this problem, you should set `output.publicPath` to resolve against the server url.

## Conclusion

Source maps can be convenient during development. They provide better means to debug applications as you can still examine the original code over a generated one. They can be valuable even for production usage and allow you to debug issues while serving a client-friendly version of your application.

To recap:

* **Source maps** can be helpful both during development and production. They provide more accurate information about what's going on and make it faster to debug possible problems.
* Webpack supports a large variety of source map variants. They can be split into inline and separate source maps based on where they are generated. Inline source maps are handy during development due to their speed. Separate source maps work for production as then loading them becomes optional.
* `devtool: "source-map"` is the highest quality option making it valuable for production.
* `cheap-module-eval-source-map` is a good starting point for development.
* If you want to get only stack traces during production, use `devtool: "hidden-source-map"`. You can capture the output and send it to a third party service for you to examine. This way you can capture errors and fix them.
* `SourceMapDevToolPlugin` and `EvalSourceMapDevToolPlugin` provide more control over the result than the `devtool` shortcut.
* *source-map-loader* can come in handy if your dependencies provide source maps.
* Enabling source maps for styling requires additional effort. You have to enable `sourceMap` option per styling related loader you are using.

In the next chapter, you'll learn to split bundles and separate the current bundle into application and vendor bundles.
