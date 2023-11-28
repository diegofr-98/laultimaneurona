---
title: Arquitectura de "n" capas
author: Diego Franco Roy
pubDatetime: 2023-11-25T23:00:00.000Z
postSlug: arquitectura-de-n-capas
featured: false
description: >-
  Aprende que és la arquitectura de "n" capas y si es buena o no para tu
  proyecto.
ogImage: /blog-images/og-image(1).jpeg
tags:
  - Arquitectura Software
---

# Arquitectura de "n" capas

La arquitectura de "N" capas es un modelo de diseño que organiza el código en capas o niveles, cada una con una función específica y bien definida. El término "N" capas representa que el número de capas puede variar según las necesidades del proyecto. Sin embargo, hay algunas capas comunes que se encuentran en muchas implementaciones, como la capa de presentación, la lógica de negocio y la capa de datos.

![](/ntier.png)

### Capa de Presentación:

Esta capa se encarga de la interfaz de usuario y la interacción con el 
usuario final. Puede incluir componentes como la interfaz gráfica de 
usuario (GUI) en aplicaciones de escritorio o la interfaz de usuario web
 en aplicaciones basadas en la web.

### Capa de Lógica de Negocio:

Contiene la funcionalidad principal de la aplicación. Aquí se implementan las reglas de negocio, la lógica de procesamiento y la gestión de datos. La separación de esta capa permite que la lógica del negocio sea independiente de la capa de presentación y la capa de datos.

### Capa de Datos:

La capa de datos se encarga de interactuar con la fuente de datos, ya sea una base de datos, servicios web o cualquier otro medio de almacenamiento. Esta capa gestiona las operaciones de lectura y escritura de datos, proporcionando una interfaz coherente para que la lógica de negocio acceda a la información.

### Pros:

* Mayor velociad de desarrollo al inicio del proyecto.
* Menor complejidad, arquitectura sencilla.
* Fácil de testear.
* Compilación y despliegue sencillos.

### Cons:

* Difícil de mantener a largo plazo.
* Tendencia a una gran dependencia entre componentes.
* Mayor dificultad a la hora de repartir el trabajo.
* Si queremos actualizar el sistema, debemos desplegarlo de nuevo completamente.

### Cuando usar:

* En proyectos pequeños, con pocos requisitos y bien definidos.
* Sistemas con un corto tiempo de vida.
* Si el equipo tiene poca experiencia.
