# requirements.md — Amane (Anime Tracker & Discovery)

> **Versión:** 1.0-draft  
> **Metodología:** Spec-Driven Development (SDD) — Nivel 1: Spec First  
> **Estado:** EN REVISIÓN (Cero líneas de código de implementación)  

---

## 1. Contexto y objetivos

**Amane** es una aplicación diseñada para el registro personal ágil de anime y el descubrimiento de nuevo contenido a través del círculo de confianza del usuario. 

El sistema nace como respuesta a dos problemas comunes en las plataformas tradicionales:
1. **Sobrecarga visual y cognitiva:** Sitios como MyAnimeList o AniList poseen interfaces densas, repletas de foros, muros y elementos sociales abiertos que entorpecen la tarea primaria: registrar con rapidez qué se está viendo y saber qué ver a continuación.
2. **Pérdida de recomendaciones personales:** Las sugerencias recibidas mediante aplicaciones de mensajería (WhatsApp, Discord) se extravían en el historial de chat. No existe un flujo centralizado donde recibir, evaluar y aceptar recomendaciones directas de amigos para transformarlas en pendientes de visualización.

Amane centraliza el seguimiento minimalista, un buzón de recomendaciones directas entre amistades, un motor de afinidad algorítmica y un feed inteligente de novedades y secuelas.

---

## 2. Glosario

- **Anime:** Obra de animación seriada o unitaria con metadatos oficiales (título, cantidad total de episodios, géneros, estado de emisión y obras vinculadas).
- **Entrada de Lista (AnimeEntry):** Registro individual que vincula a un Usuario con un Anime específico, rastreando su estado de visualización, progreso de episodios y puntuación.
- **Estado de Visualización:** Situación de un anime en la lista del usuario (`PLANNING`, `WATCHING`, `COMPLETED`, `ON_HOLD`, `DROPPED`).
- **Amistad (Friendship):** Vínculo bilateral consentido entre dos usuarios registrados en el sistema.
- **Recomendación Directa:** Sugerencia formal de un anime enviada por un usuario a un amigo específico, acompañada de una nota con el motivo de la recomendación.
- **Buzón de Recomendaciones:** Sección del perfil del usuario donde se gestionan las recomendaciones recibidas y enviadas.
- **Motor de Afinidad:** Algoritmo que analiza los animes calificados favorablemente por el usuario para descubrir y sugerir nuevos títulos afines no consumidos.
- **Secuela / Franquicia Vinculada:** Obra derivada, temporada posterior o película vinculada canónicamente a un anime preexistente.
- **Rewatch (Revisualización):** Acción de volver a ver una obra que el usuario ya completó en su totalidad al menos una vez.
- **Índice de Revisitado (Rewatchability Index):** Métrica porcentual que mide la propensión o atractivo de una obra para ser vista nuevamente por la comunidad (proporción de usuarios que la completaron y decidieron volver a verla).

---

## 3. Entidades del dominio

### Usuario (User)
- `id`: identificador único en el sistema.
- `username`: nombre único de usuario (3 a 20 caracteres alfanuméricos).
- `email`: correo electrónico válido para registro y notificaciones.
- `biografia`: texto libre de presentación (máximo 250 caracteres).
- `fechaRegistro`: fecha y hora de creación de la cuenta.

### Anime (Catálogo)
- `id`: identificador único de la obra (mapeado al catálogo canónico externo).
- `titulo`: nombre oficial de la obra.
- `totalEpisodios`: cantidad confirmada de capítulos (entero positivo o indefinido si está en emisión abierta).
- `generos`: lista de géneros temáticos (ej. Acción, Ciencia Ficción, Romance).
- `estudios`: estudios de animación responsables.
- `estadoEmision`: situación actual de transmisión (`AIRING`, `FINISHED`, `NOT_YET_AIRED`).
- `secuelas`: lista de identificadores de animes vinculados directamente como secuelas o continuaciones.
- `indiceRewatch`: porcentaje estimado de revisitado (métrica de 0.0% a 100.0% derivada del comportamiento de la comunidad).

### Entrada de Lista (AnimeEntry)
- `id`: identificador único de la entrada.
- `usuarioId`: referencia al Usuario propietario.
- `animeId`: referencia al Anime registrado.
- `estado`: uno de los estados permitidos (`PLANNING`, `WATCHING`, `COMPLETED`, `ON_HOLD`, `DROPPED`).
- `episodiosVistos`: número entero ($0 \le \text{episodiosVistos} \le \text{totalEpisodios}$).
- `vecesRevisto`: cantidad de veces completada nuevamente la obra por el usuario (entero $\ge 0$, por defecto 0).
- `puntuacion`: calificación personal numérica del 1 al 10 (opcional, null si no fue calificado).
- `notas`: texto privado con observaciones o comentarios del usuario (máximo 500 caracteres).
- `fechaActualizacion`: marca temporal del último cambio.

### Amistad (Friendship)
- `id`: identificador del vínculo.
- `solicitanteId`: usuario que emite la solicitud.
- `destinatarioId`: usuario que recibe la solicitud.
- `estado`: `PENDING`, `ACCEPTED`, `REJECTED`.
- `fechaCreacion`: fecha de emisión de la solicitud.

### Recomendacion (Recommendation)
- `id`: identificador único de la recomendación.
- `emisorId`: usuario que envía la sugerencia.
- `destinatarioId`: usuario que recibe la sugerencia.
- `animeId`: anime recomendado.
- `motivo`: justificación o comentario del emisor (10 a 280 caracteres).
- `estado`: `PENDING`, `ACCEPTED`, `DISMISSED`.
- `fechaEnvio`: marca temporal del envío.

### Relaciones entre entidades
- Un **Usuario** posee exactamente una **Lista** compuesta por cero o muchas **Entradas de Lista (AnimeEntry)**.
- Un **Usuario** no puede tener más de una **Entrada de Lista** para el mismo **Anime**.
- Un **Usuario** puede tener múltiples **Amistades** con otros usuarios.
- Una **Recomendación** vincula a un **Usuario Emisor**, un **Usuario Destinatario** y un **Anime**.
- Un **Anime** puede poseer cero o múltiples referencias a **Secuelas**.

---

## 4. Reglas de negocio (RN-xxx)

- **RN-001 (Unicidad en lista):** Un usuario no puede registrar dos veces el mismo anime en su lista personal.
- **RN-002 (Límites de episodios):** El valor de `episodiosVistos` no puede ser menor a 0. Si el anime tiene un `totalEpisodios` determinado, `episodiosVistos` no puede exceder dicho total.
- **RN-003 (Transición a Completado):** Si `episodiosVistos` alcanza el `totalEpisodios` de una obra, el sistema debe solicitar o aplicar la transición del estado a `COMPLETED`.
- **RN-004 (Rango de calificación):** Toda puntuación asignada debe ser un número entero comprendido entre 1 y 10, ambos inclusive.
- **RN-005 (Bilateralidad de amistad):** Dos usuarios solo pueden enviarse recomendaciones directas si existe un registro de `Friendship` con estado `ACCEPTED` entre ambos.
- **RN-006 (Prohibición de auto-recomendación):** Un usuario no puede enviarse una recomendación a sí mismo (`emisorId != destinatarioId`).
- **RN-007 (Recomendación no redundante):** No se permite enviar una recomendación de un anime a un amigo si el destinatario ya posee dicho anime en su lista en estado `WATCHING` o `COMPLETED`.
- **RN-008 (Recomendación no duplicada activa):** No se puede emitir una recomendación para un amigo si ya existe una recomendación previa con estado `PENDING` para el mismo anime y destinatario.
- **RN-009 (Aceptación de recomendación):** Al aceptar una recomendación con estado `PENDING`, el anime se incorpora automáticamente a la lista del destinatario con estado `PLANNING` y la recomendación pasa a estado `ACCEPTED`.
- **RN-010 (Exclusión en motor de afinidad):** El motor de recomendaciones automáticas nunca debe sugerir un anime que el usuario ya tenga registrado en su lista personal en cualquiera de sus estados.
- **RN-011 (Condición de Rewatch):** Para iniciar un rewatch o registrar un incremento en `vecesRevisto`, la entrada debe encontrarse previamente en estado `COMPLETED`. Al iniciar un rewatch, la entrada pasa a estado `WATCHING` con marca de rewatch activa y `episodiosVistos` reiniciado en 0. Al completarse nuevamente, se incrementa `vecesRevisto` en +1.
- **RN-012 (Cálculo del Índice de Revisitado):** El `indiceRewatch` es una métrica comunitaria agregada que mide el porcentaje de usuarios que, tras completar la obra, registraron al menos un rewatch (`vecesRevisto >= 1`). Se presenta únicamente si la obra cuenta con una muestra estadística mínima confiable.

---

## 5. Features (F-xxx-000)

### Módulo: Tracker Personal (TRK)

#### F-TRK-001: Búsqueda y visualización de anime
- **Historia de Usuario:**  
  *Como* usuario registrado o invitado,  
  *quiero* buscar animes por su título o género,  
  *para* encontrar la obra exacta que deseo consultar o añadir a mi lista.
- **Criterios de Aceptación:**
  - **Criterio 1 (Camino feliz - Búsqueda exitosa):**  
    *Dado que* el usuario se encuentra en la barra de búsqueda e ingresa al menos 3 caracteres,  
    *cuando* ejecuta la búsqueda,  
    *entonces* el sistema muestra una lista de resultados coincidentes con título, portada, año, estado de emisión y número total de episodios.
  - **Criterio 2 (Sin coincidencias):**  
    *Dado que* el usuario busca un término que no arroja resultados,  
    *cuando* se procesa la consulta,  
    *entonces* el sistema muestra un mensaje indicando *"No se encontraron animes que coincidan con la búsqueda"* sin arrojar error.

#### F-TRK-002: Agregar anime a lista personal
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* añadir un anime a mi lista especificando su estado inicial,  
  *para* comenzar a llevar el registro de mi progreso.
- **Criterios de Aceptación:**
  - **Criterio 1 (Camino feliz):**  
    *Dado que* el usuario visualiza la ficha de un anime que no está en su lista,  
    *cuando* selecciona el estado `PLANNING` y presiona *"Guardar en mi lista"*,  
    *entonces* se crea una nueva `AnimeEntry` con 0 episodios vistos, fecha actual y queda visible de inmediato en su lista.
  - **Criterio 2 (Intento de duplicado - Regla RN-001):**  
    *Dado que* el anime ya existe en la lista del usuario,  
    *cuando* el usuario intenta agregarlo nuevamente,  
    *entonces* el sistema deshabilita la opción de agregar y ofrece únicamente la opción de editar la entrada existente.

#### F-TRK-003: Incremento rápido de episodio visto
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* sumar un episodio visto con un solo clic desde mi lista de seguimiento,  
  *para* actualizar mi progreso sin fricciones tras terminar un capítulo.
- **Criterios de Aceptación:**
  - **Criterio 1 (Camino feliz - Incremento simple):**  
    *Dado que* un anime en estado `WATCHING` tiene 5 episodios vistos de un total de 12,  
    *cuando* el usuario presiona el botón `+1`,  
    *entonces* el contador de episodios vistos se actualiza inmediatamente a 6.
  - **Criterio 2 (Llegada al límite y completado - Reglas RN-002 y RN-003):**  
    *Dado que* un anime tiene 11 episodios vistos de un total de 12,  
    *cuando* el usuario presiona el botón `+1`,  
    *entonces* el contador pasa a 12 y el sistema cambia automáticamente el estado a `COMPLETED`.
  - **Criterio 3 (Tope alcanzado):**  
    *Dado que* un anime ya tiene sus 12 de 12 episodios vistos,  
    *cuando* el usuario intenta incrementar,  
    *entonces* el botón `+1` permanece deshabilitado.

#### F-TRK-004: Iniciar y registrar rewatch personal
- **Historia de Usuario:**  
  *Como* usuario registrado que completó una obra,  
  *quiero* iniciar una re-visualización (rewatch) del anime,  
  *para* llevar el seguimiento de los capítulos que estoy volviendo a ver y contabilizar cuántas veces lo completé en total.
- **Criterios de Aceptación:**
  - **Criterio 1 (Inicio de rewatch exitoso - Regla RN-011):**  
    *Dado que* un anime se encuentra en la lista del usuario con estado `COMPLETED`,  
    *cuando* el usuario presiona la acción *"Iniciar Rewatch"*,  
    *entonces* el estado pasa a `WATCHING` con indicador visual de rewatch, `episodiosVistos` se reinicia a 0 y `vecesRevisto` conserva su valor actual.
  - **Criterio 2 (Completar un rewatch):**  
    *Dado que* el usuario se encuentra en rewatch y alcanza el total de episodios de la serie,  
    *cuando* se registra el último episodio,  
    *entonces* el estado retorna a `COMPLETED` y el contador `vecesRevisto` se incrementa automáticamente en +1.
  - **Criterio 3 (Acción no permitida):**  
    *Dado que* un anime se encuentra en estado `PLANNING` o `DROPPED`,  
    *cuando* el usuario visualiza sus opciones,  
    *entonces* la opción de *"Iniciar Rewatch"* no se encuentra disponible.

#### F-TRK-005: Visualización de índice de rewatchability (Métrica de Revisitado)
- **Historia de Usuario:**  
  *Como* usuario que explora el catálogo o recomendaciones,  
  *quiero* consultar el índice de revisitado de un anime,  
  *para* identificar series que la comunidad considera altamente rejugables y valiosas para ver más de una vez.
- **Criterios de Aceptación:**
  - **Criterio 1 (Visualización de métrica comunitaria):**  
    *Dado que* el usuario consulta la ficha de un anime con volumen representativo de visualizaciones,  
    *cuando* carga la información detallada,  
    *entonces* el sistema muestra el valor de `indiceRewatch` como porcentaje (ej. *"68% de revisitado"* / *"Alta rejugabilidad"*).
  - **Criterio 2 (Muestra insuficiente):**  
    *Dado que* el anime es un estreno en curso o cuenta con muy pocos usuarios que lo hayan completado,  
    *cuando* el usuario consulta su ficha,  
    *entonces* el sistema indica *"Índice no disponible (datos insuficientes)"* sin arrojar errores ni valores ficticios.

---

### Módulo: Social y Recomendaciones (SOC)

#### F-SOC-001: Solicitud de amistad
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* enviar una solicitud de amistad mediante el nombre de usuario de otra persona,  
  *para* habilitar el intercambio directo de recomendaciones.
- **Criterios de Aceptación:**
  - **Criterio 1 (Envío exitoso):**  
    *Dado que* el usuario busca a "Sakura" y no existe un vínculo previo,  
    *cuando* presiona *"Enviar solicitud de amistad"*,  
    *entonces* se registra una solicitud con estado `PENDING` y Sakura la visualiza en su lista de solicitudes entrantes.
  - **Criterio 2 (Solicitud duplicada o autorreferencial):**  
    *Dado que* el usuario ingresa su propio nombre de usuario o un usuario con solicitud pendiente,  
    *cuando* intenta enviar la solicitud,  
    *entonces* el sistema rechaza la operación con el mensaje *"No puedes enviarte una solicitud a ti mismo ni repetir solicitudes pendientes"*.

#### F-SOC-002: Enviar recomendación a un amigo
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* recomendar un anime a un amigo con una nota explicativa,  
  *para* que conozca una obra que considero relevante para sus gustos.
- **Criterios de Aceptación:**
  - **Criterio 1 (Camino feliz):**  
    *Dado que* el usuario tiene amistad aceptada con "Kuro" y "Kuro" no tiene el anime "Steins;Gate" en su lista,  
    *cuando* selecciona a Kuro, redacta un motivo de 50 caracteres y presiona *"Enviar recomendación"*,  
    *entonces* se crea el registro con estado `PENDING` y aparece en el buzón de Kuro.
  - **Criterio 2 (Amigo ya vio el anime - Regla RN-007):**  
    *Dado que* Kuro ya tiene "Steins;Gate" marcado como `COMPLETED` o `WATCHING`,  
    *cuando* el usuario intenta seleccionarlo como destinatario para esa obra,  
    *entonces* el sistema impide el envío con el mensaje *"Tu amigo ya tiene este anime en su lista"*.

#### F-SOC-003: Aceptar o descartar recomendación en buzón
- **Historia de Usuario:**  
  *Como* usuario receptor,  
  *quiero* gestionar las recomendaciones recibidas en mi buzón,  
  *para* incorporar a mis pendientes las que me interesen o archivar las que no.
- **Criterios de Aceptación:**
  - **Criterio 1 (Aceptación de recomendación - Regla RN-009):**  
    *Dado que* el usuario tiene una recomendación pendiente en su buzón,  
    *cuando* presiona *"Aceptar recomendación"*,  
    *entonces* la recomendación cambia a `ACCEPTED`, y el anime se incorpora automáticamente a la lista personal en estado `PLANNING`.
  - **Criterio 2 (Descarte de recomendación):**  
    *Dado que* el usuario no desea ver el anime recomendado,  
    *cuando* presiona *"Descartar"*,  
    *entonces* la recomendación pasa a estado `DISMISSED` y no se agrega ninguna entrada a su lista personal.

---

### Módulo: Descubrimiento y Afinidad (REC)

#### F-REC-001: Sugerencias automáticas por afinidad
- **Historia de Usuario:**  
  *Como* usuario con historial de visualización,  
  *quiero* recibir sugerencias generadas por el sistema basadas en mis mejores calificaciones,  
  *para* descubrir series afines sin depender exclusivamente de recomendaciones externas.
- **Criterios de Aceptación:**
  - **Criterio 1 (Generación de recomendaciones con explicación):**  
    *Dado que* el usuario tiene al menos 3 animes calificados con nota $\ge 8$ de género "Cyberpunk",  
    *cuando* consulta su pestaña de descubrimiento,  
    *entonces* el sistema le presenta títulos afines que NO están en su lista (RN-010), indicando el motivo explícito (ej. *"Sugerido por tu alta calificación en obras Cyberpunk"*).
  - **Criterio 2 (Historial insuficiente):**  
    *Dado que* el usuario es nuevo y no tiene animes calificados,  
    *cuando* ingresa a la pestaña de descubrimiento,  
    *entonces* el sistema muestra las tendencias globales de la temporada actual invitándolo a calificar sus primeros animes.

---

### Módulo: Feed de Novedades y Secuelas (NOT)

#### F-NOT-001: Alerta de secuelas de animes seguidos
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* ser alertado cuando se anuncie o estrene una nueva temporada de un anime que completé o estoy viendo,  
  *para* no perder la continuidad de mis historias favoritas.
- **Criterios de Aceptación:**
  - **Criterio 1 (Detección de nueva temporada):**  
    *Dado que* el usuario tiene "Frieren" como `COMPLETED` y el catálogo detecta una secuela confirmada en estado `AIRING` o fecha de estreno fijada,  
    *cuando* el usuario abre su feed de novedades,  
    *entonces* visualiza una tarjeta destacada informando sobre la secuela con la opción *"Agregar a pendientes"* en un solo paso.

---

## 6. Alcance del proyecto (Scope Management)

### DENTRO del alcance — Versión 1.0
- Registro, login y gestión básica de perfil de usuario.
- Búsqueda de anime contra catálogo externo oficial.
- Gestión completa de lista personal de animes (5 estados canónicos, progreso de episodios, nota 1-10, notas privadas).
- Sistema de amistad bilateral consentida.
- Buzón de recomendaciones entre amigos con notas personalizadas y aceptación directa hacia `PLANNING`.
- Algoritmo de afinidad por géneros/estudios con explicabilidad de sugerencia.
- Feed de tendencias de temporada y detección de secuelas de títulos seguidos.

### FUERA del alcance (Explícito para Versión 1.0)
- **Streaming y reproducción de video:** Amane no es una plataforma de contenido audiovisual.
- **Red social abierta:** Sin foros de discusión, comentarios públicos en fichas de animes ni muros globales.
- **Redacción editorial de noticias:** Las novedades son procesadas automáticamente desde los metadatos de las APIs externas.
- **Catálogo de Manga y Novelas Ligeras:** Exclusivo para anime en la V1.0.
- **Integración con Discord Bot:** Planificada formalmente para la versión 2.0 (se diseñará el núcleo en el `architect.md` desacoplado mediante arquitectura hexagonal para habilitar el bot en V2 sin rehacer el dominio).
- **Importación/Exportación de listas (XML/JSON de MAL):** Postergado para versión posterior a la v1.0.

---

