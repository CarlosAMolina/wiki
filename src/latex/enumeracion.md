## Contenidos

- [Símbolos](#símbolos)
- [Recursos](#recursos)

## Símbolos

Utilizar diferentes símbolos (puntos con negrita, diamantes, etc.)

Ejemplo:

```bash
\renewcommand{\labelitemi}{$-$}    % simbolo para la enumeracion

\renewcommand{\labelitemi}{$\bullet$}
\renewcommand{\labelitemii}{$\cdot$}
\renewcommand{\labelitemiii}{$\diamond$}
\renewcommand{\labelitemiv}{$\ast$}

\begin{enumerate}[label=(\alph{*})]
\item) this is item a
\item another item
\end{enumerate}
```

## Recursos

Listas

<https://www.sharelatex.com/learn/Lists>

<http://minisconlatex.blogspot.com.es/2011/11/listas-y-enumeraciones.html>

Labels y Cross-referencing

<https://en.wikibooks.org/wiki/LaTeX/Labels_and_Cross-referencing>

Símbolos

<http://texblog.org/2008/10/16/lists-enumerate-itemize-description-and-how-to-change-them/>.
