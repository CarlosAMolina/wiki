# AWS Kendra

Es un servicio de búsqueda inteligente que utiliza machine learning; da la sensación de estar tratando con una persona.

[Enlace a la documentación](https://docs.aws.amazon.com/kendra/latest/dg/what-is-kendra.html).

Se integra con otros servicios de AWS. Es un servicio que suele utilizarse para ofrecer nueva funcionalidad a otros.

Conceptos:

- Index. Son los datos a buscar, organizados de una manera eficiente.
- Data source:
  - Donde se alojan los datos, Kendra se conecta y los indexa de aquí. Ejemplo: S3, RDS, OneDrive, Google Workspace, web crawlers etc.
  - [Enlace a la documentación](https://docs.aws.amazon.com/kendra/latest/dg/hiw-data-source.html).
  - Puedes programar que un data source se sincronice con un index.
- Documentos. Pueden ser:
  - Estructurados. Ejemplo: FAQs.
  - No estructurados. Ejemplo: HTML PDFs, texto, etc.
