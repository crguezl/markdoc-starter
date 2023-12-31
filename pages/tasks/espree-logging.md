---
title: "Espree Logging"
key: espree-logging
published: true
date:
delivery:
campus: "https://campusingenieriaytecnologia2223.ull.es/mod/assign/view.php?id=26887"
order: 8
layout: Practica
sidebar: auto
prev: "hello-compiler.md"
next: "scope-intro.md"
rubrica:
  - "Opciones en línea de comandos (-o, -V, -h, etc.)"
  - "Añade mensajes de logs a la entrada de las <code>function()</code>"
  - "Añade mensajes de logs a la entrada de las arrow <code>() => { ... }</code>"
  - "Tutorial README.md y paneles bien presentados"
  - Da información correcta de los números de línea
  - El <code>package.json</code> tiene <a href="#entrega">scripts para ejecutar el programa</a>
  - El paquete está publicado en npmjs con ámbito <code>aluXXX</code>
  - Contiene un ejecutable que se ejecuta correctamente (<code>--help</code>, etc.)
  - El módulo exporta las funciones adecuadas
  - Contiene suficientes tests 
  - "Estudio de covering"
  - Se ha hecho CI con GitHub Actions
  - "La documentación es completa: API, ejecutable, instalación, etc." 
  - Se ha probado que la librería está accesible y funciona 
  - Se ha probado que el ejecutable queda correctamente instalado, puede ser ejecutado con el nombre publicado y produce salidas correctas
  - Se ha hecho un buen uso del versionado semántico en la evolución del módulo
---

# Práctica {{ $frontmatter.title }}

## Descripción de la Tarea

Esta práctica usa como template el repo [ULL-ESIT-PL/espree-logging-template](https://github.com/ull-esit-pl/espree-logging-template).

En el repo [Repo ULL-ESIT-GRADOII-PL/esprima-pegjs-jsconfeu-talk](https://github.com/ULL-ESIT-GRADOII-PL/esprima-pegjs-jsconfeu-talk) encontrará el programa `logging-espree.js`  el cual implementa una función `addLogging` que:

* cuando se llama analiza el código JS que se la da como entrada 
* produciendo como salida un código JS equivalente que inserta  mensajes de `console.log` a la entrada de cada función.

Por ejemplo, cuando se llama con esta entrada:

```js
addLogging(`
function foo(a, b) {   
  var x = 'blah';   
  var y = (function () {
    return 3;
  })();
}     
foo(1, 'wut', 3);
`);
```

produce una salida como esta:

```
[~/javascript-learning/esprima-pegjs-jsconfeu-talk(private)]$ node logging-espree.js 

```

```js
function foo(a, b) {
    console.log('Entering foo()');
    var x = 'blah';
    var y = function () {
        console.log('Entering <anonymous function>()');
        return 3;
    }();
}
foo(1, 'wut', 3);
```

Este es el código de `logging-espree.js`: 

```
[~/javascript-learning/esprima-pegjs-jsconfeu-talk(private)]$ cat logging-espree.js 

```

```js
const escodegen = require('escodegen');
const espree = require('espree');
const estraverse = require('estraverse');

function addLogging(code) {
    const ast = espree.parse(code);
    estraverse.traverse(ast, {
        enter: function(node, parent) {
            if (node.type === 'FunctionDeclaration' ||
                node.type === 'FunctionExpression') {
                addBeforeCode(node);
            }
        }
    });
    return escodegen.generate(ast);
}

function addBeforeCode(node) {
    const name = node.id ? node.id.name : '<anonymous function>';
    const beforeCode = "console.log('Entering " + name + "()');";
    const beforeNodes = espree.parse(beforeCode).body;
    node.body.body = beforeNodes.concat(node.body.body);
}

console.log(addLogging(`
function foo(a, b) {   
  var x = 'blah';   
  var y = (function () {
    return 3;
  })();
}
foo(1, 'wut', 3);
`));
```

Le ayudarán a entender el código estos recursos:

* [Patrick Dubroy: Parsing, Compiling, and Static Metaprogramming -- JSConf EU 2013](https://youtu.be/UqTlToUYK1E)
* [Trasparencias explicando este código](https://github.com/ULL-ESIT-GRADOII-PL/esprima-pegjs-jsconfeu-talk/blob/master/jsconfeu-logging.pdf)
* [AST de la función de entrada usada como ejemplo](https://astexplorer.net/#/gist/ee28a7c80b48178607f998b8242746bc/72adda89322262c0089b79c77023912d87754dbf) en [https://astexplorer.net/](https://astexplorer.net/)

## Requisitos

En esta práctica, se pide:

::: tip Requisitos
1. Acepte la asignación Classroom de esta tarea
2. En la tarea del Campus  entregue el zip con su repositorio
3. Ejecute paso a paso el código de `logging.js` usando el debugger de chrome, intentando comprender el funcionamiento de la transformación realizada. Haga un resumen de lo que ha aprendido en el fichero Markdown: `README.md` 
4. Modifique el programa para que los `console.log` insertados informen de los valores de los parámetros pasados a la función.
5. Añada la capacidad de procesar funciones con sintáxis ECMA6 flecha gorda con bloque
6. Añada el número de línea a la información de log de la función en la que se entra
7. Procese la línea de argumentos y escriba el resultado en un fichero de salida.
8. Añada pruebas e integración continua con GitHub Actions
9. [Documente](/temas/introduccion-a-javascript/documentation)
el módulo incorporando un `README.md` y la documentación de la función exportada usando [JsDoc](/temas/introduccion-a-javascript/documentation.html)
10. Publicar el paquete en npmjs con ámbito `aluXXX`
::: 

## Argumentos de la línea de comandos y salida

Vea el siguiente ejemplo de como debe funcionar una solución:

```
$ ./p0-t0-esprima-logging-sol.js 
Usage: p0-t0-esprima-logging-sol [options] <filename> [...]
 
Options:
  -V, --version            output the version number
  -o, --output <filename>  
  -h, --help               output usage information
[~/javascript-learning/esprima-pegjs-jsconfeu-talk(private)]$ cat input.js 
```

El programa usado hace un parsing de la línea de comandos mediante el módulo npm [commander.js](https://www.npmjs.com/package/commander). Puede encontrar ejemplos en el directorio
[examples](https://github.com/tj/commander.js/tree/master/examples) del repo del modulo
[commander.js](https://www.npmjs.com/package/commander).

Cuando lo ejecutamos con la opción `-V` nos da la versión:

```
$ ./p0-t0-esprima-logging-sol.js -V
0.1.0
```

Con la  opción `-o input-log.js` especificamos el fichero de salida. El programa 
muestra los contenidos del fichero de entrada:

```
$ ./p0-t0-esprima-logging-sol.js -o input-log.js input.js 
input:
```
```js
function foo(a, b) {
  var x = 'blah';
  var y = (function (z) {
    return z+3;
  })(2);
}
foo(1, 'wut', 3);
```

Al volcar la salida, vemos que el fichero de entrada ha sido transformado correctamente:

```
---
Output in file 'input-log.js'
$ cat input-log.js
```
```js
function foo(a, b) {
    console.log(`Entering foo(${ a },${ b })`);
    var x = 'blah';
    var y = function (z) {
        console.log(`Entering <anonymous function>(${ z })`);
        return z + 3;
    }(2);
}
foo(1, 'wut', 3);
---
```

Si ejecutamos la salida obtenemos la traza esperada:

```
$ node input-log.js 
Entering foo(1,wut)
Entering <anonymous function>(2)
```


## Question: Backticks in espree

Un alumno pregunta:

> Trabajando y experimentando con el método `parse()` del compilador `espree`, he comprobado que es incapaz de procesar cadenas de caracteres que posean en su interior el signo  \`,  que es usado en JS para crear cadenas de caracteres que pueden aprovecharse de la interpolación de expresiones. 

> En concreto, y a modo de ejemplo, el error me ha surgido al intentar ejecutar 
`parse()` pasando como argumento:  

```js
"console.log(`prueba`)"
```

> Me preguntaba si el analizador léxico carece verdaderamente de la capacidad para interpretar dicho símbolo y, en caso afirmativo, cómo aprovechar la mecánica de interpolación de expresiones al utilizar el analizador.
En concreto, el error que se obtiene es:  

```
SyntaxError: Unexpected character '`'.
```

**Answer: Use option `{ecmaVersion:6}`**

```js{8}
[~/javascript-learning/esprima-pegjs-jsconfeu-talk(private)]$  node
Welcome to Node.js v12.10.0.
Type ".help" for more information.
> code3 = "console.log(`prueba`)"
'console.log(`prueba`)'
> const { parse } = require('espree')
undefined
> parse(code3, {ecmaVersion:6})
Node {
  type: 'Program',
  start: 0,
  end: 21,
  body: [
    Node {
      type: 'ExpressionStatement',
      start: 0,
      end: 21,
      expression: [Node]
    }
  ],
  sourceType: 'script'
}
```

## Reto 1: Funciones Flecha Gorda

Añada la capacidad de procesar funciones con sintáxis ECMA6 *flecha gorda* con bloque 
como en este ejemplo:

```js
let z = (e => { 
  return e +1 
})(4);
```

Ejemplo de ejecución:

```js
[~/.../eval/p0-t0-esprima-logging(master)]$ ./logging-espree.js input.js -o output.js 
input:
function foo(a, b, c) {
  let x = 'tutu';
  let y = (function (x) { return x*x })(2);
  let z = (e => { return e +1 })(4);
  console.log(x,y,z);
}
foo(1, 'wut', 3);
---
```

```
[~/.../eval/p0-t0-esprima-logging(master)]$ cat output.js 
```
```js
function foo(a, b, c) {
    console.log(`Entering foo(${ a }, ${ b }, ${ c }) at line 1`);
    let x = 'tutu';
    let y = function (x) {
        console.log(`Entering <anonymous function>(${ x }) at line 3`);
        return x * x;
    }(2);
    let z = (e => {
        console.log(`Entering <anonymous function>(${ e }) at line 4`);
        return e + 1;
    })(4);
    console.log(x, y, z);
}
```

Ejecución de la salida:

```
foo(1, 'wut', 3);[~/.../eval/p0-t0-esprima-logging-CristoNavarro(master)]$ node output.js 
Entering foo(1, wut, 3) at line 1
Entering <anonymous function>(2) at line 3
Entering <anonymous function>(4) at line 4
tutu 4 5
```

Vea aquí [El AST Espree del ejemplo](https://astexplorer.net/#/gist/30fd4865621d99718672b1cd53d6c3c9/latest) usado como entrada en la ejecución anterior. Use el parser de `espree` pasándole la opción `ecmaVersion`:

```js
const ast = espree.parse(code, {ecmaVersion:6});
```

## Reto 2: Número de Línea

Añada el número de línea a la información de log de la función en la que se entra:

```
[~/javascript-learning/esprima-pegjs-jsconfeu-talk(develop)]$ ./p0-t0-esprima-logging-sol.js input.js -o salida.js
input:
```
```js
function foo(a, b) {
  var x = 'blah';
  var y = (function (z) {
    return z+3;
  })(2);
}
foo(1, 'wut', 3);
```
```
---
Output in file 'salida.js'
[esprima-pegjs-jsconfeu-talk(develop)]$ cat salida.js
```
```js
function foo(a, b) {
    console.log(`Entering foo(${ a },${ b }) at line 1`);
    var x = 'blah';
    var y = function (z) {
        console.log(`Entering <anonymous function>(${ z }) at line 3`);
        return z + 3;
    }(2);
}
foo(1, 'wut', 3);
```

```
[esprima-pegjs-jsconfeu-talk(develop)]$ node salida.js 
Entering foo(1,wut) at line 1
Entering <anonymous function>(2) at line 3
```

## Publicación como paquete npm 

Construya un paquete npm y 
publíquelo en [npmjs](https://www.npmjs.com/) con ámbito `@aluXXX` y con nombre `espree-logging` 

Para publicar en npm haga público el repositorio en GitHub.

El módulo además de exportar la función `addLogging` provee un ejecutable `funlog`.

Una parte de los conceptos y habilidades a adquirir con esta práctica se explican en la sección [Creating and publishing a node.js module en GitHub y en NPM](/temas/introduccion-a-javascript/creating-and-publishing-npm-module). Léala con detenimiento.

### Ámbitos

Deberá publicar el paquete en [npmjs](https://www.npmjs.com/) con ámbito `@aluXXX` y con nombre `constant-folding`.

Para saber sobre ámbitos, vea la sección [Scopes and Registries](/temas/introduccion-a-javascript/creating-and-publishing-npm-module#scopes-and-registries).

## Pruebas, Covering e Integración Continua

Escriba las pruebas, haga un estudio de cubrimiento y añada integración continua usando GitHub Actions.

Lea las secciones [Testing with Mocha and Chai](/temas/introduccion-a-javascript/creating-and-publishing-npm-module.html#testing-with-mocha-and-chai) y [Jest](/temas/introduccion-a-javascript/jest).

## Documentación

[Documente](/temas/introduccion-a-javascript/documentation)
el módulo incorporando un `README.md` y la documentación de la función exportada usando [JsDoc](/temas/introduccion-a-javascript/documentation.html).
Lea la sección [Documenting the JavaScript Sources](/temas/introduccion-a-javascript/creating-and-publishing-npm-module.html#documenting-the-javascript-sources)



## Recursos

###  The Shape of the AST for console.log

!!!include(practicas/the-shape-of-the-ast.md)!!!

### Material para la Práctica

* En el [Repo ULL-ESIT-GRADOII-PL/esprima-pegjs-jsconfeu-talk](https://github.com/ULL-ESIT-GRADOII-PL/esprima-pegjs-jsconfeu-talk) encontrará material para esta práctica
* [AST de la función de entrada usada como ejemplo](https://astexplorer.net/#/gist/ee28a7c80b48178607f998b8242746bc/72adda89322262c0089b79c77023912d87754dbf)
* [Trasparencias explicando este código](https://github.com/ULL-ESIT-GRADOII-PL/esprima-pegjs-jsconfeu-talk/blob/master/jsconfeu-logging.pdf)

### Debugging

* [Debugging Client Code with Chrome](https://javascript.info/debugging-chrome)
* [Debugging NodeJS with Chrome and Visual Studio Code](/temas/introduccion-a-javascript/debugging)

### Commander

* El módulo npm [commander.js](https://www.npmjs.com/package/commander)
  - [Examples](https://github.com/tj/commander.js/tree/master/examples) en el repo del modulo
[commander.js](https://www.npmjs.com/package/commander)

## References

### AST Transformations

* Repo [ULL-ESIT-PL/espree-logging-template](https://github.com/ULL-ESIT-PL/espree-logging-template) usado como template para esta práctica. <!--  * Puede encontrar releases (.zip, .tar.gz) en [crguezl/espree-logging-template/releases/tag/v1](https://github.com/crguezl/espree-logging-template/releases/tag/v1)-->
* [Tipos de Nodos del AST](/assets/temas/tema0-introduccion-a-pl/espree-visitorkeys)
* [Espree](https://github.com/eslint/espree)
  * [Options for parse and tokenize methods](https://github.com/eslint/espree#options)
  * [API doc at ull-esit-pl.github.io/espree](https://ull-esit-pl.github.io/espree/#/)
* [Escodegen repo en GitHub](https://github.com/estools/escodegen)
  - [Escodegen API Doc](https://github.com/estools/escodegen/wiki/API)
* [Estraverse Usage](https://github.com/estools/estraverse/wiki/Usage)
  * [Estraverse README.md](https://github.com/estools/estraverse)

### Packages

* [Creating and Publishing a node.js Module in GitHub and NPM Registries](/temas/introduccion-a-javascript/creating-and-publishing-npm-module)
* [Módulos](/temas/introduccion-a-javascript/modulos)
* [Node.js Packages](/temas/introduccion-a-javascript/nodejspackages)
* [Instalación de Módulos desde GitHub](/temas/introduccion-a-javascript/nodejspackages.html#instalaci%C3%B3n-desde-github)
* [Introducción a los Módulos en JS](https://lenguajejs.com/automatizadores/introduccion/commonjs-vs-es-modules/) por Manz
* [@ull-esit-dsi-1617/scapegoat](https://www.npmjs.com/package/@ull-esit-dsi-1617/scapegoat) en npm
* [How to install an npm package from GitHub directly?](https://stackoverflow.com/questions/17509669/how-to-install-an-npm-package-from-github-directly) in StackOverflow
* [Working with scoped packages](https://docs.npmjs.com/getting-started/scoped-packages)
* [npm-scope manual: Scoped packages](https://docs.npmjs.com/misc/scope#publishing-public-scoped-packages-to-the-public-npm-registry)
* [Package.json documentation en npm site](https://docs.npmjs.com/files/package.json)

### Testing and Documentation

* [Jest](/temas/introduccion-a-javascript/jest)
* [Mocha](/temas/introduccion-a-javascript/mocha)
* [Documentation](/temas/introduccion-a-javascript/documentation)

### Semantic versioning and npm

* [Semantic versioning and npm](https://docs.npmjs.com/getting-started/semantic-versioning)
* [Semantic Versioning: Why You Should Be Using it](https://www.sitepoint.com/semantic-versioning-why-you-should-using/) SitePoint
* [YouTube Video: Semantic versioning and npm](https://youtu.be/kK4Meix58R4)
* [El comando npm version](https://docs.npmjs.com/cli/version)
