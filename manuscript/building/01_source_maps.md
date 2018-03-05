# Source Maps

![Source maps in Chrome](images/sourcemaps.png)

When your source code has gone through any transformations, debugging becomes a problem. When debugging in a browser, how to tell where the original code is? **Source maps** solve this problem by providing a mapping between the original and the transformed source code. In addition to source compiling to JavaScript, this works for styling as well.

One approach is to simply skip source maps during development and rely on browser support of language features. If you use ES2015 without any extensions and develop using a modern browser, this can work. The advantage of doing this is that you avoid all the problems related to source maps while gaining better performance.

T> If you want to understand the ideas behind source maps in greater detail, [read Ryan Seddon's introduction to the topic](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/).

## Inline Source Maps and Separate Source Maps

Webpack can generate both inline source maps included within bundles or separate source map files. The former are valuable during development due to better performance while the latter are handy for production usage as it keeps the bundle size small. In this case, loading source maps is optional.

It's possible you **don't** want to generate a source map for your production bundle as this makes it effortless to inspect your application. By disabling them you are performing a sort of obfuscation. Whether or not you want to enable source maps for production, they are handy for staging. Skipping source maps entirely speeds up your build as generating source maps at the best quality can be a complex operation.

**Hidden source maps** give stack trace information only. You can connect them with a monitoring service to get traces as the application crashes allowing you to fix the problematic situations. While this isn't ideal, it's better to know about possible problems than not.

T> It's a good idea to study the documentation of the loaders you are using to see loader specific tips. For example, with TypeScript, you have to set a particular flag to make it work as you expect.

## Enabling Source Maps

Webpack provides two ways to enable source maps. There's a `devtool` shortcut field. You can also find two plugins that give more options to tweak. The plugins are going to be discussed briefly at the end of this chapter. Beyond webpack, you also have to enable support for source maps at the browsers you are using for development.

{pagebreak}

### Enabling Source Maps in Webpack

To get started, you can wrap the core idea within a configuration part. You can convert this to use the plugins later if you want:

**webpack.parts.js**

```javascript
exports.generateSourceMaps = ({ type }) => ({
  devtool: type,
});
```

Webpack supports a wide variety of source map types. These vary based on quality and build speed. For now, you can enable `eval-source-map` for development and `source-map` for production. This way you get good quality while trading off performance, especially during development.

Set these up as follows:

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

`eval-source-map` builds slowly initially, but it provides fast rebuild speed. More rapid development specific options, such as `cheap-module-eval-source-map` and `eval`, produce lower quality source maps. All `eval` options emit source maps as a part of your JavaScript code.

`source-map` is the slowest and highest quality option of them all, but that's fine for a production build.

If you build the project now (`npm run build`), you should see source maps in the output:

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

Take a good look at those *.map* files. That's where the mapping between the generated and the original source happens. During development, it writes the mapping information in the bundle itself.

### Enabling Source Maps in Browsers

To use source maps within a browser, you have to enable source maps explicitly as per browser-specific instructions:

* [Chrome](https://developer.chrome.com/devtools/docs/javascript-debugging). Sometimes source maps [will not update in Chrome inspector](https://github.com/webpack/webpack/issues/2478). For now, the temporary fix is to force the inspector to reload itself by using *alt-r*.
* [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)
* [IE Edge](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/f12-devtools-guide/debugger/#source-maps)
* [Safari](https://developer.apple.com/library/safari/documentation/AppleApplications/Conceptual/Safari_Developer_Guide/ResourcesandtheDOM/ResourcesandtheDOM.html#//apple_ref/doc/uid/TP40007874-CH3-SW2)

W> If you want to use breakpoints (i.e., a `debugger;` statement or ones set through the browser), the `eval`-based options won't work in Chrome!

## Source Map Types Supported by Webpack

Source map types supported by webpack can be split into two categories:

* **Inline** source maps add the mapping data directly to the generated files.
* **Separate** source maps emit the mapping data to separate source map files and link the original source to them using a comment. Hidden source maps omit the comment on purpose.

Thanks to their speed, inline source maps are ideal for development. Given they make the bundles big, separate source maps are the preferable solution for production. Separate source maps work during development as well if the performance overhead is acceptable.

{pagebreak}

## Inline Source Map Types

Webpack provides multiple inline source map variants. Often `eval` is the starting point and [webpack issue #2145](https://github.com/webpack/webpack/issues/2145#issuecomment-294361203) recommends `cheap-module-eval-source-map` as it's a good compromise between speed and quality while working reliably in Chrome and Firefox browsers.

To get a better idea of the available options, they are listed below while providing a small example for each. The source code contains only a single `console.log('Hello world')` and `webpack.NamedModulesPlugin` is used to keep the output easier to understand. In practice, you would see a lot more code to handle the mapping.

T> `webpack.NamedModulesPlugin` replaces number based module IDs with paths. It's discussed in the *Adding Hashes to Filenames* chapter.

### `devtool: "eval"`

`eval` generates code in which each module is wrapped within an `eval` function:

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');\n\n//////////////////\n// WEBPACK FOOTER\n// ./app/index.js\n// module id = ./app/index.js\n// module chunks = 1\n\n//# sourceURL=webpack:///./app/index.js?")
  }
}, ["./app/index.js"]);
```

{pagebreak}

### `devtool: "cheap-eval-source-map"`

`cheap-eval-source-map` goes a step further and it includes base64 encoded version of the code as a data url. The result includes only line data while losing column mappings.

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/MGUwNCJdLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLy8vLy8vLy8vLy8vLy8vLy9cbi8vIFdFQlBBQ0sgRk9PVEVSXG4vLyAuL2FwcC9pbmRleC5qc1xuLy8gbW9kdWxlIGlkID0gLi9hcHAvaW5kZXguanNcbi8vIG1vZHVsZSBjaHVua3MgPSAxIl0sIm1hcHBpbmdzIjoiQUFBQSIsInNvdXJjZVJvb3QiOiIifQ==")
  }
}, ["./app/index.js"]);
```

If you decode that base64 string, you get output containing the mapping:

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

`cheap-module-eval-source-map` is the same idea, except with higher quality and lower performance:

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vYXBwL2luZGV4LmpzPzIwMTgiXSwic291cmNlc0NvbnRlbnQiOlsiY29uc29sZS5sb2coJ0hlbGxvIHdvcmxkJyk7XG5cblxuLy8gV0VCUEFDSyBGT09URVIgLy9cbi8vIGFwcC9pbmRleC5qcyJdLCJtYXBwaW5ncyI6IkFBQUEiLCJzb3VyY2VSb290IjoiIn0=")
  }
}, ["./app/index.js"]);
```

Again, decoding the data reveals more:

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

In this particular case, the difference between the options is minimal.

{pagebreak}

### `devtool: "eval-source-map"`

`eval-source-map` is the highest quality option of the inline options. It's also the slowest one as it emits the most data:

```javascript
webpackJsonp([1, 2], {
  "./app/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/ZGFkYyJdLCJuYW1lcyI6WyJjb25zb2xlIiwibG9nIl0sIm1hcHBpbmdzIjoiQUFBQUEsUUFBUUMsR0FBUixDQUFZLGFBQVoiLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLyBXRUJQQUNLIEZPT1RFUiAvL1xuLy8gLi9hcHAvaW5kZXguanMiXSwic291cmNlUm9vdCI6IiJ9")
  }
}, ["./app/index.js"]);
```

This time around there's more mapping data available for the browser:

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

## Separate Source Map Types

Webpack can also generate production usage friendly source maps. These end up in separate files ending with `.map` extension and are loaded by the browser only when required. This way your users get good performance while it's easier for you to debug the application.

`source-map` is a good default here. Even though it takes longer to generate the source maps this way, you get the best quality. If you don't care about production source maps, you can simply skip the setting there and get better performance in return.

### `devtool: "cheap-source-map"`

`cheap-source-map` es parecida a las opciones fáciles de arriba. Las columnas de mapeo van a faltar en el resultado. Además, los mapas fuente de los cargadores, tales como *css-loader*, no van a usar.

Examinar el archivo `.map` muestra la suiguiente salida en este caso:

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

La fuente original contiene el tipo de comentario `//# sourceMappingURL=app.9a...18.js.map` al final para mapear a este archivo.

### `devtool: "cheap-module-source-map"`

`cheap-module-source-map` es la misma que las anteriores excepto por los mapas fuente de los cargadores que son simplificados a un solo mapeado por línea. Da el siguiente resultado en este caso:

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

W> `cheap-module-source-map` es [currently broken if minification is used](https://github.com/webpack/webpack/issues/4176) y esta es una buena razón para evitar la opción por ahora.

### `devtool: "hidden-source-map"`

`hidden-source-map` es lo mismo que `source-map` excepto que no escribe las referencias en los mapas fuente en los archivos fuente. Si no quieres exponer los mapas fuente a las herramientas de desarrollo directamente mientras quieres apilar rutas, esto es útil.

T> [The official documentation](https://webpack.js.org/configuration/devtool/#devtool) contiene más finromación de las opciones `devtool`.

{pagebreak}

### `devtool: "source-map"`

`source-map` provee las mejor calidad con el resultado completo, pero es también la opción más lenta. La salida muetra esto:

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

## Otras opciones de mapas fuente

Hay un par de otras opciones que afectan la generación de mapas fuente:

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

T> La [official documentation](https://webpack.js.org/configuration/output/#output-sourcemapfilename) profundiza en detalles `output`.

W> Si estás usando cualquier `UglifyJsPlugin` y aún quieres mapas fuente, necesitas activar `sourceMap: true` para el plugin. De otra forma, el resultado no será lo que esperas porque UglifyJS realizará otra transformación en el código, rompiendo el mapeo. Lo mismo tiene que hacerse con los otros plugins y cargadores realizando transformaciones. *css-loader* y los cargadores relacionados son un buen ejemplo.

## `SourceMapDevToolPlugin` y `EvalSourceMapDevToolPlugin`

Si quieres más control sobre la generación de mapas fuente, es posible usar el [SourceMapDevToolPlugin](https://webpack.js.org/plugins/source-map-dev-tool-plugin/) o `EvalSourceMapDevToolPlugin` en su lugar. El último es una alternativa más limitada, y como lo dice su nombre, es útill para generar mapas fuente basados en `eval`.

Ambos plugins pueden permitir un control más granular sobre a qué partes del código les quieres generar mapas fuente, mientras tiene también un control estricto sobre el resultado con `SourceMapDevToolPlugin`. Usar cualquier plugin nos permite omitir la opción `devtool` completamente.

Dado que webpack sólo empareja archivos `.js` y `.css` por defecto para mapas fuente, puedes usar `SourceMapDevToolPlugin` para solucionar ese problema. Esto puede lograrse poniendo un patrón `test` como `/\.(js|jsx|css)($|\?)/i`.

`EvalSourceMapDevToolPlugin` sólo acepta las opciones `module` y `lineToLine` como se describe arriba. Por lo tanto, puede considerarse como un alias para `devtool: "eval"` mientras permite un nivel más flexible.

## Cambiar prefijos de mapas fuente

Puedes prefijar una opción de mapa fuente con un caracter **pragma** que se inserten en la referencia del mapa fuente. Webpack usa `#` por defecto, el cual es soportado por navegadores modernos, así que no tienes que configurarlo.

Para superar esto, tienes que prefijar tu opción de mapa fuente con él (e.g., `@source-map`). Luego del cambio, deberías ver el tipo de referencia `//@` en el mapa fuente sobre `//#` en tus archivos JavaScript, asumiendo que fue usado un tipo de mapa fuente separado.

## Usar mapas fuente de dependencia

Asumiendo que estás usando un paquete que usa mapas fuente en línea en su distribución, puedes usar [source-map-loader](https://www.npmjs.com/package/source-map-loader) para hacer a webpack consciente de ellos. Sin configurarlo contra el paquete, obtienes una salida de depuración minificada. A menudo, puedes saltar este paso ya que es un caso especial.

## Mapas fuente para diseño

Si quieres activar mapas fuente para archivos de diseño, puedes lograrlo activando la opción `sourceMap`. La misma idea funciona con cargadores de archivo tales como *css-loader*, *sass-loader*, y *less-loader*.

El *css-loader* es [known to have issues](https://github.com/webpack-contrib/css-loader/issues/232) cuando usas rutas relativas para importar. Para solucionar este problema, deberías poner `output.publicPath` para resolver contra el url del servidor.

## Conclusión

Los mapas fuente pueden ser convenientes durante el desarrollo. Ellos proporcionan mejores medios para depurar aplicaciones ya que todavía puedes examinar el código original sobre el generado. Pueden ser valiosos incluso para uso de producción y te permiten depurar problemas mientras se sirve una versión de cliente amigable para tu aplicación.

Recapitulando:

* **Source maps** pueden ser de ayuda durante el desarrollo y la producción. Proporcionan información más precisa sobre qué está pasando y hacerlo más rápido para depurar posibles problemas.
* Webpack soporta una gran variedad de variantes de mapas fuente. Pueden ser separadas en mapas fuente en línea y separadas, basadas en dónde fueron generadas. Mapas fuente en línea son útiles durante el dessarrollo debido a su velocidad. Mapas fuente separados funcionan para producción como luego cargarlos se convierte opcional.
* `devtool: "source-map"` es la opción de mayor calidad, haciéndola valiosa para producción.
* `cheap-module-eval-source-map` es un buen punto de partida para desarrollo.
* Si quieres obtener sólo rastros de pila durante la producción, usa `devtool: "hidden-source-map"`. Puedes capturar la salida y enviarla a un servicio de terceros para que la examines. De esta menera, puedes capturar errores y arreglarlos.
* `SourceMapDevToolPlugin` y `EvalSourceMapDevToolPlugin` proveen un mayor control sobre el resultado que el acceso directo `devtool`.
* *source-map-loader* puede ser útil si tus dependencias proporcionan mapas fuente.
* Activar mapas fuente para diseño requiere de un esfuerzo adicional. Tienes que activar la opción `sourceMap` por cada cargador de diseño relacionado que estés usando.

En el capítulo siguiente, aprenderás a dividir paquetes y a separar el paquete actual en paquetes de aplicación y de comerciante.
