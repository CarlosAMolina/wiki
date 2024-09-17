## Contenidos

- [Ejemplo](#ejemplo)
- [Recursos](#recursos)

## Ejemplo

```bash
\usepackage[hidelinks]{hyperref}    % links en el pdf (indice...) [hidelinks] quita la caja roja
\usepackage[dvipsnames]{xcolor}

% configuracion. color enlaces
\newcommand{\colorEnlaces}{blue!80!black}
\hypersetup{
    colorlinks,
    linkcolor={\colorEnlaces},
    citecolor={\colorEnlaces},
    urlcolor={\colorEnlaces}
}

% begin{document}

%En la siguiente web \href{https://github.com}{\textcolor{blue}{github page}}
En la siguiente web \href{https://github.com}{github page}
```

## Recursos

Hyperlinks (índice, url...)

<http://minisconlatex.blogspot.com.es/2012/09/hyperlinks-con-latex.html>
