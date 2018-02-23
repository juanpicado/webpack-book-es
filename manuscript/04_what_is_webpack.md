# Qué es Webpack


Webpack es un **empaquetador de modulos**. Webpack se encargará de empaquetar en paralelo a un ejecutador de tareas, sin embargo, la línea entre un empaquetador y un ejecutador de tareas es algo borrosa gracias a la comunidad que ha desarrollado extensiones para webpack. A veces estas extensiones son usadas para ejecutar tareas que usualmente habian sido echas fuera de webpack, tales como limpiar directorio de construcción o el despliegue del proyecto.

React con  **Hot Module Replacement** (HMR) ayudó enormemente a popularizar webpack y guió el uso de webpack en otros ambientes, tales como [Ruby on Rails](https://github.com/rails/webpacker). A pesar del nombre, webpack no se ha limitado solo a la web. Puede empaquetar muchos otros objectivos también, como se discute en el capítulo de *Construir Objetivos*.

T> Si tu quieres entender las herramientras de construcción y su historia en detalle, echa un vistazo a el apéndice de *Comparación de Herramientras de Construcción*.

## Webpack Depende de Modulos

El proyecto más pequeño que puedas empaquetar con webpack consiste en una **entrada** y una **salida**. El proceso de empaquetado empieza por un serie de **entradas**. Las entradas en su son **modulos** y pueden apuntar a otros modulos a través de **importaciones**.

Cuando empaquetas un proyecto usando webpack, se recorre a través de importaciones, construyendo un **gráfico de dependencias** del proyecto y generando un **resultado** basado en la configuración. Adicionalmente, es posible definir **puntos de separación** generado diferentes paquetes dentro del código de proyecto en sí mismo.

Webpack soporta formatos ES2015, CommonJS y modulos AMD listos para usar. El mecanismo funciona para CSS tambien, con `@import` y `url()` soportados usando *css-loader*. También puedes encontar extensiones para tareas específicas, como minificación, internacionalización, HMR y demás.

T> Un grafo de dependendencias es un gráfo dirigido que describe como nodos se relacionan unos a los otros. En este caso el grafo dirigido es definido a traves de referencias (`require`, `import`) entre archivos. Webpack recorre esta información en una manera estática sin ejecutar el origen y generando el grafo que es necesitado para crear los paquetes.

## Ejecución de Proceso de Webpack

![Webpack's execution process](images/webpack-process.png)

Webpack empieza su trabajo desde las **entradas**. Amenudo estas son modulos de JavaScript donde webpack empieza el proceso de recorrido. Durante este proceso, webpack evalua los emparejamientos a traves de configuracion de **loader** que le dice a webpack como transformar cada coincidencia.

{pagebreak}

### Proceso de Resolución

Una entrada en su misma es un módulo. Cuando webpack encuentra una, webpack trata de coincidir la entrada en contra de un archivo del sistema usando la entrada en la configuración `resolve`. Puedes decirle a webpack que ejecute una busqueda a traves de diferentes directorios en adición de *node_modules*. También es posible ajustar la manera que webpack empareja con extensiones de archivos, y puedes definir alias específicos para directorios. El capítulo de *Técnicas de Consumir Paquetes* cubre esas ideas en gran detalle.

Si la resolución del archivo falla, webpack reacciona con un error de ejecución. Si webpack consigue resolver el archivo correctamente, webpack ejecuta el proceso sobre el archivo coincidente basado en al definición del loader. Cada loader aplica a una transformación específica contra el contenido de cada modulo.

The way a loader gets matched against a resolved file can be configured in multiple ways, including by file type and by location within the file system. Webpack's flexibility even allows you to apply specific transformation against a file based on *where* it was imported to the project.

The same resolution process is performed against webpack's loaders. Webpack allows you to apply similar logic when determining which loader it should use. Loaders have resolve configurations of their own for this reason. If webpack fails to perform a loader lookup, it will raise a runtime error.

### Webpack Resolves Against Any File Type

Webpack will resolve each module it encounters while constructing the dependency graph. If an entry contains dependencies, the process will be performed recursively against each dependency until the traversal has completed. Webpack can perform this process against any file type, unlike specialized tools like the Babel or Sass compiler.

Webpack gives you control over how to treat different assets it encounters. For example, you can decide to **inline** assets to your JavaScript bundles to avoid requests. Webpack also allows you to use techniques like CSS Modules to couple styling with components, and to avoid issues of standard CSS styling. This flexibility is what makes webpack so valuable.

Although webpack is used mainly to bundle JavaScript, it can capture assets like images or fonts and emit separate files for them. Entries are only a starting point of the bundling process. What webpack emits depends entirely on the way you configure it.

### Evaluation Process

Assuming all loaders were found, webpack evaluates the matched loaders from bottom to top and right to left (`styleLoader(cssLoader('./main.css'))`), running the module through each loader in turn. As a result, you get output which webpack will inject in the resulting **bundle**. The *Loader Definitions* chapter covers the topic in detail.

If all loader evaluation completed without a runtime error, webpack includes the source in the last bundle. **Plugins** allow you to intercept **runtime events** at different stages of the bundling process.

Although loaders can do a lot, they don’t provide enough power for advanced tasks. Plugins can intercept **runtime events** provided by webpack. A good example is bundle extraction performed by the `ExtractTextPlugin` which, when used with a loader, extracts CSS files out of the bundle and into a separate file. Without this step, CSS would be inlined in the resulting JavaScript, as webpack treats all code as JavaScript by default. The extraction idea is discussed in the *Separating CSS* chapter.

### Finishing

After every module has been evaluated, webpack writes **output**. The output includes a bootstrap script with a manifest that describes how to begin executing the result in the browser. The manifest can be extracted to a file of its own, as discussed later in the book. The output differs based on the build target you are using (targeting web is not the only option).

That’s not all there is to the bundling process. For example, you can define specific **split points** where webpack generates separate bundles that are loaded based on application logic. This idea is discussed in the *Code Splitting* chapter.

{pagebreak}

## Webpack Is Configuration Driven

At its core, webpack relies on configuration. Here is a sample configuration adapted from [the official webpack tutorial](https://webpack.js.org/get-started/) that covers the main points:

**webpack.config.js**

```javascript
const webpack = require("webpack");

module.exports = {
  // Where to start bundling
  entry: {
    app: "./entry.js",
  },

  // Where to output
  output: {
    // Output to the same directory
    path: __dirname,

    // Capture name from the entry using a pattern
    filename: "[name].js",
  },

  // How to resolve encountered imports
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.js$/,
        use: "babel-loader",
        exclude: /node_modules/,
      },
    ],
  },

  // What extra processing to perform
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
  ],

  // Adjust module resolution algorithm
  resolve: {
    alias: { ... },
  },
};
```

Webpack's configuration model can feel a bit opaque at times as the configuration file can appear monolithic. It can be difficult to understand what webpack is doing unless you understand the ideas behind it. Providing means to tame configuration is one of the main purposes why this book exists.

T> To understand webpack on the source code level, check out [the artsy webpack tour](https://github.com/TheLarkInn/artsy-webpack-tour).

## Asset Hashing

With webpack, you can inject a hash to each bundle name (e.g., *app.d587bbd6.js*) to invalidate bundles on the client side as changes are made. Bundle-splitting allows the client to reload only a small part of the data in the ideal case.

{pagebreak}

## Hot Module Replacement

You are likely familiar with tools, such as [LiveReload](http://livereload.com/) or [BrowserSync](http://www.browsersync.io/), already. These tools refresh the browser automatically as you make changes. *Hot Module Replacement* (HMR) takes things one step further. In the case of React, it allows the application to maintain its state without forcing a refresh. While this does not sound all that special, it can make a big difference in practice.

HMR is also available in Browserify via [livereactload](https://github.com/milankinen/livereactload), so it’s not a webpack exclusive feature.

## Code Splitting

In addition to HMR, webpack’s bundling capabilities are extensive. Webpack allows you to split code in various ways. You can even load code dynamically as your application gets executed. This sort of lazy loading comes in handy especially for larger applications, as dependencies can be loaded on the fly as needed.

Even small applications can benefit from code splitting, as it allows the users to get something useable in their hands faster. Performance is a feature, after all. Knowing the basic techniques is worthwhile.

{pagebreak}

## Conclusion

Webpack comes with a significant learning curve. However, it’s a tool worth learning, given how much time and effort it can save over the long term. To get a better idea how it compares to other tools, check out [the official comparison](https://webpack.js.org/get-started/why-webpack/#comparison).

Webpack won’t solve everything, however, it does solve the problem of bundling. That’s one less worry during development. Using *package.json* and webpack alone can take you far.

To summarize:

* Webpack is a **module bundler**, but you can also use it for task running as well.
* Webpack relies on a **dependency graph** underneath. Webpack traverses through the source to construct the graph, and it uses this information and configuration to generate bundles.
* Webpack relies on **loaders** and **plugins**. Loaders operate on a module level, while plugins rely on hooks provided by webpack and have the best access to its execution process.
* Webpack’s **configuration** describes how to transform assets of the graphs and what kind of output it should generate. Part of this information can be included in the source itself if features like **code splitting** are used.
* **Hot Module Replacement** (HMR) helped to popularize webpack. It's a feature that can enhance the development experience by updating code in the browser without needing a full page refresh.
* Webpack can generate **hashes** for filenames allowing you to invalidate past bundles as their contents change.

In the next part of the book you'll learn to construct a development configuration using webpack while learning more about its basic concepts.
