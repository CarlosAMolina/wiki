## Contenidos

- [Compilación](#compilación)
  - [Texmaker](#texmaker)
  - [Terminal](#terminal)
- [Paths](#paths)
  - [plainnat.bst](#plainnat.bst)
- [Caracteres no ASCII](#caracteres-no-ascii)
- [Orden](#orden)
  - [Ordenar alfabéticamente](#ordenar-alfabéticamente)
  - [Orden apellido, nombre](#orden-apellido-nombre)
- [URLs](#urls)
  - [Mostrar URL](#mostrar-url)
  - [Cortar URLs](#cortar-urls)
- [Errores](#errores)
  - [Texmaker muestra interrogación en lugar de número](#texmaker-muestra-interrogación-en-lugar-de-número)
  - [Al compilar siempre se tiene error](#al-compilar-siempre-se-tiene-error)
- [Recursos](#recursos)

## Compilación

### Texmaker

Dar a menú herramientas > BibTex y LaTex y PDFLatex(no preocuparse si este da error): F2(error, no problema), F6, F11 y luego compilación rápida (F1)

Link <http://tex.stackexchange.com/questions/67431/bib-file-does-not-update-in-texmakerx-when-i-build-the-file>

### Terminal

Para que el índice aparezca correctamente, ejecutar el siguiente comandos dos veces:

```bash
lualatex archivo.tex
```

## Paths

### plainnat.bst

Ruta en Ubuntu a plainnat.bst: `/usr/share/texlive/texmf-dist/bibtex/bst/natbib`

## Caracteres no ASCII

Escribir del siguiente modo:

- Acentos: \'o -> ó
- letras mayúsculas: {P}
- ñ: \~n
- º: \textordmasculine{ } ({ } para que haya espacio después)

## Orden

### Ordenar alfabéticamente

Para ordenar alfabéticamente, escribir siempre el autor entre doble llave. Ejm `{{ xxx }}`.

### Orden apellido, nombre

Link <https://tex.stackexchange.com/questions/131087/displaying-authors-name-in-a-bibliographic-entry-in-the-form-surname-first-in>.

## URLs

### Mostrar URL

Mostrar URL por ejemplo de un tipo @ article:

- Importar paquete URL: `\usepackage{url}`
- Utilizar: `\bibliographystyle{plainurl}`

Link <https://tex.stackexchange.com/questions/21633/url-for-article-and-other-entries-of-the-bib-file>.

### Cortar URLs

```bash
\usepackage[hyphens]{url}
\usepackage[hidelinks]{hyperref}
\hypersetup{breaklinks=true}
```

Link <https://tex.stackexchange.com/questions/115690/urls-in-bibliography-latex-not-breaking-line-as-expected>

## Errores

### Texmaker muestra interrogación en lugar de número

Menú `opciones` > `configurar texmaker` > `compilacion rapida`: `pdflatx+bi(la)tex+pdflatex(x2)+viewpdf`.

### Al compilar siempre se tiene error

Si nunca deja de aparecer un error, compilar solo la biografía (Bibtex), aunque de error, después compilar como siempre.

## Recursos

Alternativas software

<http://alternativeto.net/software/jabref/>

Tutorial bibliografía

<http://minisconlatex.blogspot.com.es/2011/03/bibliografia.html>
