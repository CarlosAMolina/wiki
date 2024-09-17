## Contenidos

- [Ejemplo de paquetes](#ejemplo-de-paquetes)

## Ejemplo de paquetes

```bash
% Paquetes
\usepackage[spanish]{babel}	    % contenidos en espaniol
\usepackage[utf8]{inputenc}    % acentos
\usepackage{graphicx}
\usepackage{caption}
\usepackage[labelfont=bf]{caption} % los 'Figura x.x' de los caption, ponerlo en negrita
\usepackage{float}
\usepackage{fancyhdr}    % pagina X de Y
\usepackage{lastpage}
\usepackage{cite}    % para contraer referencias
\usepackage[hidelinks]{hyperref}    % links en el pdf (indice...) [hidelinks] quita la caja roja. si da error usar \documentclass[hidelinks,12pt]{report} (https://tex.stackexchange.com/questions/823/remove-ugly-borders-around-clickable-cross-references-and-hyperlinks)
\usepackage{enumerate}    % enumeracion
\renewcommand{\labelitemi}{$-$}    % simbolo para la enumeracion
%\renewcommand{\baselinestretch}{1.5}    % interlineado, defecto=1
\usepackage{booktabs}    %usar tablas
% \usepackage{lscape}    % begin-end:landscape. permite introducir paginas horizontales entre verticales. no modifica ancho de tablas y figuras automaticamente (usar rotating)
%\resizebox{\linewidth}{!}{ en lugar de textwidth para que se ajuste al ancho total
\usepackage{pdflscape} % exporta la hoja del pdf ya girada
\usepackage{rotating}   % begin-end:sidewaysfigure. utiliza el ancho de la pagina
\usepackage{amsmath}    % referencias a la secuaciones \eqref{eq:solve} pone enter parentesis el numero de la ecuacion al hacer referencia en el texto
\usepackage[titles]{tocloft}    % para indice de ecuaciones (crear uno propio). % [titles]: tamaño igual que el reesto
\usepackage{gensymb} % poder usar º
\usepackage{subcaption}    % subplot
\usepackage{adjustbox}    % subplot rodeados por un cuadro
\usepackage{eurosym}    % simbolo euro
\usepackage{longtable} % continuar tabla en siguiente pagina
\usepackage{array}    % definir nueva columna en una tabla http://tex.stackexchange.com/questions/66123/how-do-i-set-right-alignment-in-a-longtable-column-with-a-fixed-width
% \usepackage{pdfpages} http://elclubdelautodidacta.es/wp/2013/05/insercion-de-documentos-pdf-en-latex/
\usepackage{wrapfig} % imagen al lado del texto
http://tex.stackexchange.com/questions/118602/how-to-text-wrap-an-image-in-latex
\usepackage{anysize}
% \marginsize{3cm}{3cm}{2,5cm}{2,5cm} %{izquierda}{derecha}{arriba}{abajo}
http://www.espaciolinux.com/foros/software/latex-como-definen-correctamente-los-margenes-t34057.html
\usepackage{color} % colores https://en.wikibooks.org/wiki/LaTeX/Colors ejm: \textcolor{blue}{mi blog}
%\pagenumbering{gobble}% Remove page numbers (and reset to 1) https://tex.stackexchange.com/questions/54333/no-page-numbering
\usepackage{subfiles} % permite ejecutar cada .tex independiente si hacerlo desde el principal. en cada .tex debe incluirse \documentclass[ruta/archivo_principal.tex]{subfiles}
\begin{document} ..... \end{document} https://tex.stackexchange.com/questions/94414/how-to-best-use-and-compile-multiple-tex-files-as-part-of-same-final-document
```
