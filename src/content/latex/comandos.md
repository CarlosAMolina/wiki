## Contenidos

- [Ejemplo](#ejemplo)
- [Recursos](#recursos)

## Ejemplo

```bash
\documentclass[12pt]{article}
\usepackage[spanish]{babel}	    % contenidos en espaniol


\begin{document}

\newcommand{\sectionTitle}[1]{
  \begin{center}
    \begingroup
      \fontsize{15pt}{0em}\selectfont
      \textbf{#1}
    \endgroup
  \end{center}
}

% sin comando
\begin{center}
  \begingroup
    \fontsize{15pt}{0em}\selectfont
    \textbf{Texto de ejemplo}
  \endgroup
\end{center}

% con comando
\sectionTitle{Texto de ejemplo}

\end{document}
```

## Recursos

<https://www.overleaf.com/learn/latex/Commands>
