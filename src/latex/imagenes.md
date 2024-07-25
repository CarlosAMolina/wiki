## Contenidos

- [Consideraciones](#consideraciones)
  - [Espacio antes de la figura](#espacio-antes-de-la-figura)
- [Recursos](#recursos)

## Consideraciones

### Espacio antes de la figura

Al insertar una imagen, no puede especificarse de la siguiente manera:

```bash
La siguiente imagen:

\\
\begin{figure}[H]
\includegraphics[width=\textwidth,height=\textheight,keepaspectratio]{./imagenes/image.jpg}
\end{figure}
```

Debe ser así:

```bash
La siguiente imagen:
\\
\begin{figure}[H]
\includegraphics[width=\textwidth,height=\textheight,keepaspectratio]{./imagenes/image.jpg}
\end{figure}
```

## Recursos

Bordes

<http://tex.stackexchange.com/questions/20640/how-to-add-border-for-an-image>

Subfiguras

<http://tex.stackexchange.com/questions/55640/trying-to-put-a-box-around-a-collection-of-subfigures>

Tutorial imágenes

<https://es.sharelatex.com/learn/Inserting_Images#A.C3.B1adir_etiquetas.2C_leyendas_y_referencias>
