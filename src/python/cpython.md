## Contenidos

- [Etapas ejecución código](#etapas-ejecución-código)
  - [Parser](#parser)
  - [Compiler](#compiler)
  - [Assembler](#assembler)
  - [Execution](#execution)
  - [Referencias](#referencias)

## Etapas ejecución código

Este apartado explica las etapas para ejecutar un código en python.

### Parser

El código en python a ejecutar es convertido en estructuras lógicas que pueda manejar el compilador.

Estas estructuras lógicas se llaman AST (Abstract Syntax Tree). Posee estructura en árbol. Contiene las operaciones, funciones, clases y namespaces del código a ejecutar.

### Compiler

Convierte el AST en instrucciones que entienda la CPU.

El symbol table explora los nodos y ramas del AST y añade las entradas al symtable. Contiene namespaces, variables globales y locales para el compilador; puede tener tablas hijas.

El Compiler State es un contenedor que contiene un symbol table.

Estas instrucciones están en un CFG (control-flow-graph), representa la secuencia lógica de ejecución. Posee una estructura en gráfico.

Cada nodo del CFG es un frame block, y cada uno contiene un conjunto de instrucciones y apuntan al siguiente bloque.

### Assembler

Convierte los frame blocks en secuencia bytecode.

Los bytecode son operaciones a ejecutar (instrucciones ejecutables secuenciales) contenidas en code objects.

### Execution

La ejecución de los módulos la realiza el core evaluation loop del intérprete CPython.

### Referencias

<https://realpython.com/products/cpython-internals-book/>
