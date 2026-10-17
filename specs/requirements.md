# requirements.md — Amane (Anime Tracker & Discovery)

> **Versión:** 1.0-final  
> **Metodología:** Spec-Driven Development (SDD) — Nivel 1: Spec First  
> **Estado:** CERRADO Y APROBADO (Fase 0 de Especificación Completada al 100% — Cero TBDs)  

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
- `id`: identificador único interno en el sistema (UUID).
- `username`: nombre único de usuario en Amane (3 a 20 caracteres alfanuméricos).
- `email`: correo electrónico provisto y verificado por el proveedor de autenticación.
- `authProvider`: proveedor de autenticación (`GOOGLE`, `DISCORD`, o proveedor SaaS OIDC configurado).
- `externalId`: identificador único inmutable del sujeto (`sub`) provisto por el proveedor de identidad.
- `estadoCuenta`: situación de la cuenta (`PENDING_ONBOARDING`, `ACTIVE`, `PENDING_DELETION`, `DELETED`).
- `biografia`: texto libre de presentación (máximo 250 caracteres).
- `fechaRegistro`: fecha y hora de creación de la cuenta.
- `fechaUltimoLogin`: marca temporal del último acceso exitoso.
- `fechaProgramadaEliminacion`: marca temporal límite para la purga física definitiva si se encuentra en período de gracia (nullable).
- `visibilidadLista`: control de acceso a la lista (`PUBLIC`, `FRIENDS_ONLY`, `PRIVATE`, por defecto `PUBLIC`).
- `visibilidadNotas`: control de acceso a las notas personales (`PUBLIC`, `FRIENDS_ONLY`, `PRIVATE`, por defecto `PRIVATE`).

### Anime (Catálogo)
- `id`: identificador único de la obra (mapeado al catálogo canónico externo).
- `titulo`: nombre oficial de la obra.
- `formato`: tipo o medio de la obra (`TV`, `MOVIE`, `OVA`, `ONA`, `SPECIAL`).
- `totalEpisodios`: cantidad confirmada de capítulos (entero positivo, o `null` cuando la obra se encuentra en emisión abierta o sin confirmación oficial de capítulos).
- `duracionMinutos`: duración de referencia en minutos por episodio, o metraje total cuando el formato es `MOVIE` (entero positivo, opcional).
- `generos`: lista de géneros temáticos (ej. Acción, Ciencia Ficción, Romance).
- `estudios`: estudios de animación responsables.
- `estadoEmision`: situación actual de transmisión (`AIRING`, `FINISHED`, `NOT_YET_AIRED`).
- `secuelas`: lista de identificadores de animes vinculados directamente como secuelas o continuaciones.
- `indiceRewatch`: porcentaje estimado de revisitado (métrica de 0.0% a 100.0% derivada del cálculo bayesiano comunitario).

### Entrada de Lista (AnimeEntry)
- `id`: identificador único de la entrada.
- `usuarioId`: referencia al Usuario propietario.
- `animeId`: referencia al Anime registrado.
- `estado`: uno de los estados permitidos (`PLANNING`, `WATCHING`, `COMPLETED`, `ON_HOLD`, `DROPPED`).
- `episodiosVistos`: número entero ($0 \le \text{episodiosVistos} \le \text{totalEpisodios}$).
- `vecesRevisto`: cantidad de veces completada nuevamente la obra por el usuario (entero $\ge 0$, por defecto 0).
- `rewatchActivo`: indicador booleano que marca si la entrada se encuentra en una re-visualización en curso (por defecto `false`).
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

### Notificacion (Notification)
- `id`: identificador único de la notificación (UUID).
- `usuarioId`: referencia al Usuario destinatario.
- `tipo`: tipo de evento (`FRIEND_REQUEST_RECEIVED`, `FRIEND_REQUEST_ACCEPTED`, `RECOMMENDATION_RECEIVED`, `SEQUEL_AIRING_ALERT`).
- `titulo`: texto breve resumen de la notificación (máximo 80 caracteres).
- `mensaje`: texto detallado explicativo (máximo 280 caracteres).
- `enlaceAccion`: URI interna para interacción directa (ej. `/buzon`, `/amigos`, `/anime/{id}`).
- `leida`: booleano que indica si el usuario ya vio la alerta (por defecto `false`).
- `fechaCreacion`: marca temporal de emisión de la notificación.

### PreferenciasNotificacion (NotificationPreferences)
- `id`: identificador de las preferencias.
- `usuarioId`: referencia al Usuario propietario (relación 1 a 1).
- `emailRecomendaciones`: booleano (por defecto `false` - opt-in estricto).
- `emailSolicitudesAmistad`: booleano (por defecto `false` - opt-in estricto).
- `emailSecuelas`: booleano (por defecto `false` - opt-in estricto).
- `fechaActualizacion`: marca temporal del último cambio de configuración.

### Bloqueo (UserBlock)
- `id`: identificador único del bloqueo (UUID).
- `bloqueadorId`: referencia al Usuario que impone el bloqueo.
- `bloqueadoId`: referencia al Usuario bloqueado.
- `fechaBloqueo`: marca temporal en la que se aplicó el bloqueo.

### Relaciones entre entidades
- Un **Usuario** posee exactamente una **Lista** compuesta por cero o muchas **Entradas de Lista (AnimeEntry)**.
- Un **Usuario** no puede tener más de una **Entrada de Lista** para el mismo **Anime**.
- Un **Usuario** posee exactamente un registro de **PreferenciasNotificacion**.
- Un **Usuario** puede recibir cero o muchas **Notificaciones**.
- Un **Usuario** puede tener múltiples **Amistades** con otros usuarios.
- Un **Usuario** puede registrar cero o múltiples **Bloqueos** hacia otros usuarios.
- Una **Recomendación** vincula a un **Usuario Emisor**, un **Usuario Destinatario** y un **Anime**.
- Un **Anime** puede poseer cero o múltiples referencias a **Secuelas**.

---

## 4. Reglas de negocio (RN-xxx)

- **RN-001 (Unicidad en lista):** Un usuario no puede registrar dos veces el mismo anime en su lista personal.
- **RN-002 (Límites de episodios):** El valor de `episodiosVistos` no puede ser menor a 0. Si el anime tiene un `totalEpisodios` determinado, `episodiosVistos` no puede exceder dicho total.
- **RN-003 (Transición a Completado):** Si `episodiosVistos` alcanza el `totalEpisodios` de una obra con cantidad de episodios confirmada, el sistema transiciona automáticamente el estado a `COMPLETED`, salvo en las obras alcanzadas por RN-040.
- **RN-004 (Rango de calificación):** Toda puntuación asignada debe ser un número entero comprendido entre 1 y 10, ambos inclusive.
- **RN-005 (Bilateralidad de amistad):** Dos usuarios solo pueden enviarse recomendaciones directas si existe un registro de `Friendship` con estado `ACCEPTED` entre ambos.
- **RN-006 (Prohibición de auto-recomendación):** Un usuario no puede enviarse una recomendación a sí mismo (`emisorId != destinatarioId`).
- **RN-007 (Recomendaciones según estado previo del destinatario):** No se permite recomendar un anime si el destinatario ya posee dicha obra en su lista en estado `WATCHING` o `COMPLETED`. Si el amigo tiene el anime en `PLANNING` u `ON_HOLD`, el envío se permite como incentivo de visualización. Si el amigo tiene la obra en estado `DROPPED`, el sistema solicitará una confirmación explícita al emisor informándole que su amigo abandonó la serie antes de despachar la recomendación.
- **RN-008 (Recomendación no duplicada activa):** No se puede emitir una recomendación para un amigo si ya existe una recomendación previa con estado `PENDING` para el mismo anime y destinatario.
- **RN-009 (Aceptación de recomendación y sincronización de lista):** Al aceptar una recomendación con estado `PENDING`: si la obra no estaba en la lista del destinatario, se incorpora automáticamente con estado `PLANNING`; si ya figuraba en `PLANNING`, se mantiene el estado asociando la recomendación; y si figuraba en `ON_HOLD` o `DROPPED`, se le ofrece al destinatario reactivar la obra a `PLANNING` o `WATCHING`. En todos los casos, la recomendación pasa a estado `ACCEPTED` respetando la regla de unicidad en lista (RN-001).
- **RN-010 (Exclusión en motor de afinidad):** El motor de recomendaciones automáticas nunca debe sugerir un anime que el usuario ya tenga registrado en su lista personal en cualquiera de sus estados.
- **RN-011 (Condición de Rewatch):** Para iniciar un rewatch o registrar un incremento en `vecesRevisto`, la entrada debe encontrarse previamente en estado `COMPLETED`. Al iniciar un rewatch, `rewatchActivo` se establece en `true`, el estado pasa a `WATCHING` y `episodiosVistos` se reinicia en 0. Al completarse nuevamente, `rewatchActivo` vuelve a `false` y `vecesRevisto` se incrementa en +1.
- **RN-012 (Cálculo mediante Ponderación Bayesiana del Índice de Revisitado):** El `indiceRewatch` es una métrica comunitaria regularizada calculada como $\text{indiceRewatch} = \frac{N_{\text{completados}} \cdot R_{\text{observado}} + C \cdot R_{\text{global}}}{N_{\text{completados}} + C}$, donde $R_{\text{observado}} = (N_{\text{rewatched}} / N_{\text{completados}}) \times 100$, $R_{\text{global}}$ es la tasa promedio global de rewatch en la plataforma, y $C = 15$ es la constante de regularización bayesiana que amortigua anomalías en títulos con pocas visualizaciones hacia el promedio de la comunidad. Si $N_{\text{completados}} = 0$, el índice se reporta como nulo (sin datos). Si $N_{\text{completados}} < C$, se presenta con la etiqueta *"En calibración"*; y si $N_{\text{completados}} \ge C$, con la etiqueta *"Muestra representativa"*. El cálculo se actualiza de forma asíncrona periódica en base de datos.
- **RN-013 (Identidad delegada externa y cero almacenamiento de contraseñas):** Amane delega el 100% de la autenticación de usuarios y gestión de credenciales en proveedores externos (OAuth 2.0 / OpenID Connect y servicios de identidad gestionados). Amane **nunca solicita, procesa ni almacena contraseñas**.
- **RN-014 (Soporte multi-proveedor y unicidad de cuenta):** El sistema admite autenticación tanto por Social Login directo (ej. Google, Discord) como por plataformas SaaS IdP. Cada cuenta de usuario se vincula de forma única a la tupla `(authProvider, externalId)`. El `email` registrado debe ser un correo verificado por el proveedor emisor.
- **RN-015 (Validación criptográfica de tokens):** Todo acceso a recursos protegidos de la API requiere un Bearer Token (JWT) criptográficamente válido emitido por el proveedor configurado, validando obligatoriamente firma digital, emisor (`iss`), audiencia (`aud`) y tiempo de caducidad (`exp`).
- **RN-016 (Onboarding y asignación obligatoria de username):** Al autenticarse por primera vez mediante cualquier proveedor externo admitido, el usuario ingresa en estado `PENDING_ONBOARDING`. El sistema exige definir un `username` único en Amane (3 a 20 caracteres alfanuméricos) antes de habilitar el uso del tracker o las funciones sociales.
- **RN-017 (Generación de notificación In-App por evento):** Todo evento social (solicitud de amistad recibida/aceptada, recomendación directa recibida) o detección de secuela confirmada genera automáticamente una notificación In-App para el usuario destinatario, independientemente de sus preferencias de correo.
- **RN-018 (Opt-in estricto de notificaciones por Email):** Toda preferencia de notificación por correo electrónico inicia desactivada por defecto (`false`). El sistema nunca enviará correos sin el consentimiento expreso del usuario. Durante el onboarding o desde los ajustes de cuenta, el usuario debe autorizar activamente qué categorías de alerta desea recibir por correo.
- **RN-019 (Desacoplamiento y resiliencia en entrega de emails):** El despacho de correos se procesa de forma asíncrona mediante un puerto de infraestructura (`EmailSender`), utilizando Amazon SES como proveedor de alta eficiencia y costo-beneficio. Un fallo en el proveedor de correo externo jamás debe bloquear ni revertir la transacción del dominio (la recomendación o solicitud permanece creada y registrada In-App).
- **RN-020 (Borrado lógico y período de gracia de 30 días):** La solicitud voluntaria de baja de cuenta aplica un borrado lógico inmediato: la cuenta transiciona a estado `PENDING_DELETION` y se establece una `fechaProgramadaEliminacion` a 30 días corridos. Durante este período, el usuario queda completamente oculto para la comunidad (no aparece en búsquedas, no puede recibir nuevas solicitudes ni recomendaciones y se cierran sus sesiones activas).
- **RN-021 (Reactivación durante el período de gracia):** Si el usuario vuelve a autenticarse mediante su proveedor antes de que expire el plazo de 30 días, el sistema detectará el estado `PENDING_DELETION` y le ofrecerá la cancelación del borrado. Si confirma la reactivación, la cuenta retorna a estado `ACTIVE`, se anula la `fechaProgramadaEliminacion` y se restaura el acceso normal con sus datos íntegros.
- **RN-022 (Purga física definitiva y tratamiento de interacciones huérfanas):** Una vez cumplidos los 30 días sin reactivación, un proceso automatizado ejecuta la purga física e irreversible de los datos personales del usuario. El registro de `Usuario` se conserva en estado `DELETED` con sus campos personales anonimizados (`email`, `externalId` y `biografia` en `null`, y `username` sustituido por un identificador anónimo no reutilizable); se eliminan sus notas privadas y entradas de lista, así como sus vínculos de `Amistad` y registros de `Bloqueo`. Las solicitudes y recomendaciones en estado `PENDING` se suprimen en todas las direcciones. Para las recomendaciones previamente aceptadas (`ACCEPTED`), el anime se preserva en la lista del destinatario anonimizando la autoría (`emisorId = null`).
- **RN-023 (Portabilidad de datos personales - GDPR):** Todo usuario activo tiene derecho a solicitar y descargar en cualquier momento una copia íntegra y legible por máquina de sus datos personales en formato estándar `JSON` (perfil, lista de animes con estados, episodios vistos, puntuaciones y notas personales).
- **RN-024 (Visibilidad configurable de lista personal):** Por defecto, la lista personal de animes es pública (`PUBLIC`), permitiendo a cualquier usuario autenticado consultar títulos, estados, progreso de episodios y calificaciones. El usuario puede cambiar su visibilidad en cualquier momento a `FRIENDS_ONLY` (restringida exclusivamente a usuarios con amistad mutua en estado `ACCEPTED`) o a `PRIVATE` (restringida únicamente al propio usuario).
- **RN-025 (Visibilidad configurable de notas personales):** Por defecto, las notas personales de una entrada de lista son estrictamente confidenciales (`PRIVATE`), permaneciendo ocultas ante cualquier otro usuario. El usuario puede configurar en sus ajustes la visibilidad de sus notas como `FRIENDS_ONLY` (visibles solo para amistades aceptadas) o `PUBLIC` (visibles para quienes tengan acceso a consultar su lista). Si la lista se encuentra configurada en `PRIVATE`, las notas son inaccesibles por transitividad.
- **RN-026 (Eliminación de entrada de lista):** El usuario puede eliminar cualquier `AnimeEntry` de su lista personal. La operación requiere confirmación explícita y suprime de forma inmediata el registro de progreso, calificación personal y notas privadas de dicha obra en su perfil.
- **RN-027 (Corrección de progreso y reversión de estado):** El usuario puede decrementar (`-1`) o editar numéricamente el valor de `episodiosVistos` ($0 \le \text{episodiosVistos} \le \text{totalEpisodios}$). Si una entrada en estado `COMPLETED` ve reducido su progreso por debajo de `totalEpisodios`, el sistema debe revertir automáticamente su estado a `WATCHING`.
- **RN-028 (Cancelación de rewatch en curso):** El usuario puede cancelar en cualquier momento un rewatch activo. Al cancelarlo, la entrada retorna inmediatamente a estado `COMPLETED`, el valor de `episodiosVistos` se restaura al total de episodios de la serie, `rewatchActivo` se establece en `false` y el contador `vecesRevisto` permanece intacto sin sufrir incrementos ni decrementos.
- **RN-029 (Gestión, advertencia y enfriamiento de 24 horas en solicitudes de amistad):** El receptor de una solicitud de amistad en estado `PENDING` puede aceptarla o rechazarla. Al seleccionar "Rechazar", el sistema debe presentar un diálogo de advertencia explícito informando que el solicitante quedará inhabilitado para reenviar una solicitud durante 24 horas, previniendo rechazos accidentales. Tanto al confirmarse el rechazo como al cancelar el emisor una solicitud enviada, el solicitante queda inhabilitado para emitir una nueva solicitud al mismo destinatario durante un período de 24 horas continuas contadas desde ese momento.
- **RN-030 (Disolución bilateral de amistad):** Cualquiera de los integrantes de una amistad en estado `ACCEPTED` puede disolver el vínculo unilateralmente. La disolución revoca de forma inmediata el acceso a listas configuradas como `FRIENDS_ONLY`, impide futuros envíos de recomendaciones y purga toda recomendación en estado `PENDING` emitida en cualquier dirección entre ambos usuarios. Las recomendaciones previamente aceptadas (`ACCEPTED`) se mantienen en las listas personales.
- **RN-031 (Efectos del bloqueo silencioso y prevención de interacción):** El bloqueo es unilateral y silencioso (el usuario bloqueado no recibe ninguna notificación). La acción disuelve de inmediato cualquier vínculo de amistad previo, purga las solicitudes de amistad y recomendaciones en estado `PENDING` en ambas direcciones entre ambos usuarios y bloquea toda interacción (el perfil del bloqueador queda completamente oculto e inaccesible para el bloqueado en búsquedas y enlaces directos). El bloqueador podrá consultar su lista de usuarios bloqueados y revertir la acción (desbloquear) desde su configuración de perfil.
- **RN-032 (Cuotas y límites en solicitudes de amistad):** Un usuario no puede superar un máximo de 25 solicitudes de amistad emitidas en una ventana móvil de 24 horas, ni mantener más de 20 solicitudes salientes en estado `PENDING` de forma simultánea. Al alcanzar cualquiera de estos umbrales, el sistema impedirá el envío de nuevas solicitudes hasta que se resuelvan las pendientes o expire la ventana de tiempo.
- **RN-033 (Cuotas y libertad de recomendación sin saturación):** Para fomentar el intercambio fluido sin desvirtuar la plataforma, un usuario podrá acumular hasta un máximo de 15 recomendaciones en estado `PENDING` dirigidas a un mismo amigo simultáneamente, y emitir hasta 60 recomendaciones directas en una ventana móvil de 24 horas. Al superarse el límite hacia un amigo en particular, el emisor deberá esperar a que el destinatario acepte o descarte alguna antes de enviarle recomendaciones adicionales.
- **RN-034 (Rate limiting técnico en endpoints de interacción):** Los endpoints de creación de solicitudes de amistad y recomendaciones directas aplican un límite técnico de hasta 15 peticiones por minuto por cuenta autenticada para neutralizar scripts o ráfagas automatizadas abusivas. Las peticiones que excedan este ratio serán denegadas con código HTTP `429 Too Many Requests` e indicación del tiempo de espera en la cabecera `Retry-After`.
- **RN-035 (Incompatibilidad de Rewatch en obras sin final definido):** La funcionalidad de iniciar rewatch (`F-TRK-004`) y el cómputo de `vecesRevisto` aplican exclusivamente a obras que cuenten con un `totalEpisodios` determinado y se encuentren previamente en estado `COMPLETED`. En animes con `totalEpisodios = null`, la opción de rewatch permanece inhabilitada tanto en la interfaz como en el backend.
- **RN-036 (Ponderación del Motor de Afinidad):** El motor de descubrimiento calcula la afinidad de animes no registrados mediante una función de scoring compuesta: 60% peso de afinidad por géneros, 25% por afinidad de estudio de animación y 15% por calificación media global en catálogo. Solo aquellas entradas con calificación personal $\ge 7$ nutren el vector de preferencias positivas del usuario (las notas 9 y 10 tienen doble ponderación).
- **RN-037 (Explicabilidad en lenguaje natural conciso):** Toda sugerencia automática generada por el motor debe acompañarse obligatoriamente de una justificación explicativa en lenguaje natural claro y conversacional basada en los factores determinantes, evitando tecnicismos, fórmulas o notas numéricas entre paréntesis (ej. *"Sugerido por tus altas valoraciones en Ciencia Ficción y Suspenso"* o *"Sugerido por tu preferencia hacia el estudio MAPPA"*).
- **RN-038 (Condición de activación y Cold Start del motor):** El motor personalizado requiere un historial mínimo de al menos 3 animes calificados con nota $\ge 7$. Si el usuario no alcanza dicho volumen inicial (arranque en frío), la sección de descubrimiento ofrecerá la cartelera de tendencias globales de temporada, motivando al usuario a registrar y calificar sus primeras obras para calibrar su afinidad.
- **RN-039 (Incremento sin tope cuando `totalEpisodios` no está confirmado):** En animes con `totalEpisodios = null`, el contador `episodiosVistos` puede incrementarse mediante el botón `+1` o edición numérica sin ninguna restricción de tope superior.
- **RN-040 (Restricción de completado en emisión continua):** La transición a `COMPLETED` (tanto la automática de RN-003 como la selección manual del usuario) solo está permitida para obras con `estadoEmision = FINISHED` y `totalEpisodios` confirmado. Mientras una obra se encuentre en emisión abierta (`estadoEmision = AIRING` o `NOT_YET_AIRED`) o tenga `totalEpisodios = null`, los únicos estados válidos para la entrada son `PLANNING`, `WATCHING`, `ON_HOLD` y `DROPPED`, y la opción de marcar `COMPLETED` permanece deshabilitada hasta que la obra concluya su transmisión oficial con cantidad de episodios confirmada en el catálogo canónico.
- **RN-041 (Cobertura multiformato en catálogo canónico):** El catálogo de Amane admite y clasifica obras bajo cinco formatos canónicos: series de televisión (`TV`), películas (`MOVIE`), animaciones originales para video (`OVA`), animaciones para la web (`ONA`) y especiales conmemorativos (`SPECIAL`). Cada obra registrará su tipo de formato y duración promedio por capítulo o metraje cinematográfico.
- **RN-042 (Comportamiento de obras unitarias y largometrajes):** Para cualquier obra con formato `MOVIE` o con `totalEpisodios = 1` que no se encuentre en emisión conforme a RN-040, el registro del primer episodio ($0 \to 1$) transiciona de inmediato la entrada a estado `COMPLETED`. Si el usuario selecciona directamente `COMPLETED` como estado inicial al agregarla a su lista, el sistema inicializa automáticamente `episodiosVistos` en 1.
- **RN-043 (Continuidad de franquicia multiformato en feed de secuelas):** El detector de novedades y secuelas (`F-NOT-001`) reconoce cualquier película cinematográfica, OVA o temporada televisiva vinculada en el catálogo como secuela válida de la franquicia seguida por el usuario.

---

## 5. Features (F-xxx-000)

### Módulo: Autenticación e Identidad (AUTH)

#### F-AUTH-001: Autenticación multi-proveedor (Social Login y SaaS IdP)
- **Historia de Usuario:**  
  *Como* usuario (nuevo o recurrente),  
  *quiero* iniciar sesión mediante proveedores externos confiables (Google, Discord o proveedor SaaS OIDC),  
  *para* acceder a Amane de forma rápida, segura y sin gestionar contraseñas locales.
- **Criterios de Aceptación:**
  - **Criterio 1 (Usuario existente - Login exitoso):**  
    *Dado que* el usuario posee una cuenta previamente creada y se autentica exitosamente con su proveedor externo,  
    *cuando* el backend de Amane valida criptográficamente el token (RN-015),  
    *entonces* se reconoce la identidad del usuario, se actualiza `fechaUltimoLogin` y se le permite acceso a sus datos.
  - **Criterio 2 (Nuevo usuario - Redirección a Onboarding):**  
    *Dado que* la identidad externa provista (`authProvider` + `externalId`) no existe en Amane,  
    *cuando* se valida el token por primera vez,  
    *entonces* el sistema crea el registro con estado `PENDING_ONBOARDING` y redirige al flujo de selección de `username` (RN-016).
  - **Criterio 3 (Rechazo de credencial inválida):**  
    *Dado que* una petición presenta un token manipulado, vencido o con emisor no autorizado,  
    *cuando* el backend realiza la verificación criptográfica,  
    *entonces* rechaza la solicitud de inmediato con código 401 Unauthorized sin emitir sesión.

#### F-AUTH-002: Asignación de nombre de usuario (Onboarding)
- **Historia de Usuario:**  
  *Como* usuario nuevo en estado `PENDING_ONBOARDING`,  
  *quiero* asignar mi nombre de usuario único en Amane,  
  *para* completar la activación de mi cuenta y poder interactuar con la plataforma.
- **Criterios de Aceptación:**
  - **Criterio 1 (Asignación exitosa):**  
    *Dado que* el usuario ingresa un `username` libre de entre 3 y 20 caracteres alfanuméricos,  
    *cuando* confirma la selección,  
    *entonces* el sistema asigna el `username`, transiciona la cuenta a estado `ACTIVE` y habilita el acceso total a las funciones de tracker y social.
  - **Criterio 2 (Colisión o formato inválido):**  
    *Dado que* el `username` ingresado ya está en uso por otro usuario o contiene caracteres especiales/espacios,  
    *cuando* intenta confirmarlo,  
    *entonces* el sistema muestra el mensaje de error correspondiente y mantiene al usuario en `PENDING_ONBOARDING`.
  - **Criterio 3 (Paso de consentimiento de notificaciones - RN-018):**  
    *Dado que* el usuario asigna exitosamente su `username`,  
    *cuando* culmina el proceso de alta,  
    *entonces* el sistema le presenta una pantalla consultándole si desea activar alertas por correo electrónico (todas desmarcadas por defecto en `false`); si el usuario no las marca o salta el paso, se persisten en `false` respetando el opt-in estricto.

#### F-AUTH-003: Cierre de sesión (Logout)
- **Historia de Usuario:**  
  *Como* usuario autenticado,  
  *quiero* cerrar mi sesión actual,  
  *para* proteger el acceso a mis datos en dispositivos compartidos.
- **Criterios de Aceptación:**
  - **Criterio 1 (Cierre de sesión local):**  
    *Dado que* el usuario solicita cerrar su sesión,  
    *cuando* se ejecuta la acción,  
    *entonces* el cliente descarta las credenciales/tokens locales y cualquier petición subsecuente a la API es rechazada hasta una nueva autenticación.

#### F-AUTH-004: Solicitud de eliminación de cuenta (Borrado lógico con gracia)
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* solicitar la eliminación de mi cuenta y mis datos personales,  
  *para* ejercer mi derecho al olvido con la seguridad de contar con un período de gracia de 30 días en caso de arrepentirme.
- **Criterios de Aceptación:**
  - **Criterio 1 (Confirmación explícita y pase a PENDING_DELETION - RN-020):**  
    *Dado que* el usuario se encuentra en la sección de seguridad de su perfil e ingresa su `username` exacto en el diálogo de confirmación,  
    *cuando* presiona "Confirmar eliminación de mi cuenta",  
    *entonces* la cuenta pasa a estado `PENDING_DELETION`, se calcula `fechaProgramadaEliminacion` a 30 días, se cierran todas sus sesiones y el perfil queda completamente oculto para otros usuarios.
  - **Criterio 2 (Bloqueo por confirmación incorrecta):**  
    *Dado que* el usuario no escribe con exactitud su `username`,  
    *cuando* visualiza el diálogo de confirmación,  
    *entonces* el botón de confirmación permanece inhabilitado impidiendo borrados involuntarios.

#### F-AUTH-005: Reactivación de cuenta durante período de gracia
- **Historia de Usuario:**  
  *Como* usuario con cuenta en estado `PENDING_DELETION`,  
  *quiero* tener la oportunidad de reactivar mi cuenta al iniciar sesión antes de cumplirse los 30 días,  
  *para* recuperar mi historial y lista personal sin pérdida de información.
- **Criterios de Aceptación:**
  - **Criterio 1 (Cancelación de borrado y reactivación exitosa - RN-021):**  
    *Dado que* el usuario posee su cuenta en `PENDING_DELETION` y se autentica mediante su proveedor externo dentro de los 30 días de gracia,  
    *cuando* el sistema le presenta el aviso de cuenta en proceso de eliminación y el usuario selecciona "Reactivar mi cuenta",  
    *entonces* la cuenta retorna a estado `ACTIVE`, se elimina la fecha programada de purga y se restablece el acceso habitual con todos sus datos intactos.
  - **Criterio 2 (Rechazo de reactivación / Mantener borrado):**  
    *Dado que* el usuario en `PENDING_DELETION` se autentica pero selecciona "Mantener eliminación",  
    *cuando* confirma su elección,  
    *entonces* el sistema cierra su sesión de inmediato y el cronómetro de los 30 días continúa sin interrupción.

#### F-AUTH-006: Exportación de datos personales (Portabilidad JSON - RN-023)
- **Historia de Usuario:**  
  *Como* usuario activo,  
  *quiero* descargar una copia completa de mis datos almacenados en formato JSON,  
  *para* respaldar mi información personal y ejercer mi derecho de portabilidad.
- **Criterios de Aceptación:**
  - **Criterio 1 (Descarga exitosa de archivo JSON):**  
    *Dado que* el usuario se encuentra en los ajustes de privacidad de su perfil,  
    *cuando* presiona "Exportar mis datos (JSON)",  
    *entonces* el sistema genera y descarga un archivo estructurado conteniendo sus datos de perfil, preferencias de notificación y la totalidad de sus entradas de lista con calificaciones y notas privadas.

---

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
  - **Criterio 3 (Filtrado por formato - RN-041):**  
    *Dado que* el usuario realiza una búsqueda y aplica un filtro de formato (`TV`, `MOVIE`, `OVA`, `ONA`, `SPECIAL`),  
    *cuando* se procesa la consulta,  
    *entonces* el sistema retorna exclusivamente las obras que coincidan con el formato seleccionado mostrando su duración y número de episodios.

#### F-TRK-002: Agregar anime a lista personal
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* añadir un anime a mi lista especificando su estado inicial,  
  *para* comenzar a llevar el registro de mi progreso adaptado a su formato.
- **Criterios de Aceptación:**
  - **Criterio 1 (Camino feliz):**  
    *Dado que* el usuario visualiza la ficha de un anime que no está en su lista,  
    *cuando* selecciona el estado `PLANNING` y presiona *"Guardar en mi lista"*,  
    *entonces* se crea una nueva `AnimeEntry` con 0 episodios vistos, fecha actual y queda visible de inmediato en su lista.
  - **Criterio 2 (Intento de duplicado - Regla RN-001):**  
    *Dado que* el anime ya existe en la lista del usuario,  
    *cuando* el usuario intenta agregarlo nuevamente,  
    *entonces* el sistema deshabilita la opción de agregar y ofrece únicamente la opción de editar la entrada existente.
  - **Criterio 3 (Película u obra de 1 episodio agregada como COMPLETADA - RN-042):**  
    *Dado que* el usuario visualiza una película (`MOVIE`) o un especial de 1 episodio que no está en su lista,  
    *cuando* selecciona directamente el estado `COMPLETED` y guarda,  
    *entonces* se crea la `AnimeEntry` con `episodiosVistos = 1` automáticamente sin requerir incrementos adicionales.

#### F-TRK-003: Actualización y corrección de progreso de episodios
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* incrementar (+1), decrementar (-1) o editar numéricamente los episodios vistos de un anime,  
  *para* mantener mi progreso al día y corregir rápidamente equivocaciones.
- **Criterios de Aceptación:**
  - **Criterio 1 (Incremento simple):**  
    *Dado que* un anime en estado `WATCHING` tiene 5 episodios vistos de un total de 12,  
    *cuando* el usuario presiona el botón `+1`,  
    *entonces* el contador de episodios vistos se actualiza inmediatamente a 6.
  - **Criterio 2 (Llegada al límite y completado automático - RN-002 y RN-003):**  
    *Dado que* un anime tiene 11 episodios vistos de un total de 12,  
    *cuando* el usuario presiona el botón `+1`,  
    *entonces* el contador pasa a 12 y el sistema transiciona automáticamente el estado a `COMPLETED`.
  - **Criterio 3 (Corrección y decremento con reversión de estado - RN-027):**  
    *Dado que* un anime se encuentra en `COMPLETED` con 12 de 12 episodios vistos,  
    *cuando* el usuario presiona `-1` o edita manualmente el valor a 11,  
    *entonces* el contador pasa a 11 y el estado revierte automáticamente a `WATCHING`.
  - **Criterio 4 (Límites inferiores y superiores):**  
    *Dado que* un anime tiene 0 episodios vistos,  
    *cuando* el usuario intenta decrementar con `-1`,  
    *entonces* el botón `-1` permanece inhabilitado y el valor no desciende de 0.
  - **Criterio 5 (Incremento en obra de emisión abierta sin total de episodios - RN-039 y RN-040):**  
    *Dado que* un anime se encuentra en emisión continua con `totalEpisodios = null` (ej. episodio 1090),  
    *cuando* el usuario presiona el botón `+1` o ingresa un número mayor,  
    *entonces* el contador actualiza normalmente su valor sin bloqueo superior y la entrada permanece en estado `WATCHING` sin aplicar completado automático.

#### F-TRK-004: Iniciar, completar y cancelar rewatch personal
- **Historia de Usuario:**  
  *Como* usuario registrado que completó una obra,  
  *quiero* iniciar, seguir o cancelar una re-visualización (rewatch) del anime,  
  *para* llevar el seguimiento de los capítulos que estoy volviendo a ver y corregir si inicié el rewatch por error.
- **Criterios de Aceptación:**
  - **Criterio 1 (Inicio de rewatch exitoso - RN-011):**  
    *Dado que* un anime se encuentra en la lista del usuario con estado `COMPLETED`,  
    *cuando* el usuario presiona la acción *"Iniciar Rewatch"*,  
    *entonces* `rewatchActivo` pasa a `true` (mostrando el indicador visual de rewatch), el estado pasa a `WATCHING`, `episodiosVistos` se reinicia a 0 y `vecesRevisto` conserva su valor actual.
  - **Criterio 2 (Completar un rewatch):**  
    *Dado que* el usuario se encuentra en rewatch y alcanza el total de episodios de la serie,  
    *cuando* se registra el último episodio,  
    *entonces* el estado retorna a `COMPLETED`, `rewatchActivo` vuelve a `false` y el contador `vecesRevisto` se incrementa automáticamente en +1.
  - **Criterio 3 (Cancelar rewatch en curso - RN-028):**  
    *Dado que* el usuario se encuentra en un rewatch activo (ej. en el episodio 3 de 12),  
    *cuando* selecciona la opción *"Cancelar Rewatch"*,  
    *entonces* el estado retorna a `COMPLETED`, `episodiosVistos` se restaura al total (12) y `vecesRevisto` permanece intacto sin sumar ni restar.
  - **Criterio 4 (Acción no permitida - RN-011):**  
    *Dado que* un anime se encuentra en cualquier estado distinto de `COMPLETED` (por ejemplo `PLANNING`, `WATCHING`, `ON_HOLD` o `DROPPED`),  
    *cuando* el usuario visualiza sus opciones,  
    *entonces* la opción de *"Iniciar Rewatch"* no se encuentra disponible.
  - **Criterio 5 (Obra en emisión abierta con `totalEpisodios = null` - RN-035):**  
    *Dado que* un anime se encuentra en emisión abierta o continua con `totalEpisodios = null` (ej. obras de cientos de capítulos en emisión),  
    *cuando* el usuario consulta las opciones de su entrada en lista,  
    *entonces* la opción de *"Iniciar Rewatch"* permanece deshabilitada indicando que el rewatch requiere que la obra cuente con episodios confirmados y haya alcanzado el estado completado.

#### F-TRK-005: Visualización de índice de rewatchability (Métrica de Revisitado)
- **Historia de Usuario:**  
  *Como* usuario que explora el catálogo o recomendaciones,  
  *quiero* consultar el índice de revisitado de un anime calculado de forma bayesiana,  
  *para* identificar series que la comunidad considera altamente rejugables con estabilidad estadística.
- **Criterios de Aceptación:**
  - **Criterio 1 (Muestra representativa - RN-012):**  
    *Dado que* el anime cuenta con $N_{\text{completados}} \ge 15$ usuarios que lo finalizaron,  
    *cuando* carga la ficha detallada,  
    *entonces* el sistema muestra el porcentaje regularizado por la fórmula bayesiana (ej. *"42.5% de revisitado"*), su categoría cualitativa y el distintivo *"Muestra representativa"*.
  - **Criterio 2 (Muestra en calibración - RN-012):**  
    *Dado que* el anime posee entre 1 y 14 usuarios completados ($N_{\text{completados}} < 15$),  
    *cuando* el usuario consulta su ficha,  
    *entonces* el sistema muestra el índice suavizado hacia el promedio global de la plataforma acompañado del distintivo informativo *"En calibración"*.
  - **Criterio 3 (Cero completados):**  
    *Dado que* la obra es un estreno o ningún usuario en la plataforma la ha completado aún ($N_{\text{completados}} = 0$),  
    *cuando* se consulta su ficha,  
    *entonces* el sistema indica *"Índice no disponible (sin datos de visualización)"* sin arrojar errores.

#### F-TRK-006: Consulta de lista ajena y control de visibilidad
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* consultar la lista personal de otro usuario y ver solo la información que este haya autorizado,  
  *para* conocer sus gustos respetando sus preferencias de privacidad de lista y notas.
- **Criterios de Aceptación:**
  - **Criterio 1 (Acceso a lista pública por cualquier usuario - RN-024):**  
    *Dado que* el usuario "Sakura" tiene su lista configurada en `PUBLIC`,  
    *cuando* cualquier usuario registrado accede al perfil de Sakura,  
    *entonces* visualiza los animes de su lista con sus estados, episodios vistos y calificaciones.
  - **Criterio 2 (Acceso restringido a solo amigos - RN-024):**  
    *Dado que* el usuario "Kuro" tiene su lista en `FRIENDS_ONLY` y un usuario sin amistad aceptada intenta consultarla,  
    *cuando* solicita la lista,  
    *entonces* el sistema deniega el acceso indicando que la lista solo es visible para amigos mutuos (403 Forbidden).
  - **Criterio 3 (Lista estrictamente privada):**  
    *Dado que* un usuario tiene su lista en `PRIVATE`,  
    *cuando* cualquier otra persona (incluyendo amigos) intenta consultarla,  
    *entonces* el sistema responde que la lista es privada (403 Forbidden).
  - **Criterio 4 (Filtrado de notas personales según RN-025):**  
    *Dado que* un usuario consulta la lista de un amigo con notas en `PRIVATE`,  
    *cuando* se reciben las entradas,  
    *entonces* los campos `notas` son omitidos de la respuesta; únicamente si el dueño configuró sus notas en `FRIENDS_ONLY` o `PUBLIC` se incluirá su contenido.

#### F-TRK-007: Eliminar anime de la lista personal
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* eliminar un anime de mi lista personal tras una confirmación de seguridad,  
  *para* quitar obras agregadas por error o que ya no deseo rastrear.
- **Criterios de Aceptación:**
  - **Criterio 1 (Eliminación exitosa con confirmación - RN-026):**  
    *Dado que* el usuario selecciona la opción "Eliminar anime de mi lista" en una entrada y confirma la acción en el modal de advertencia,  
    *cuando* se ejecuta la solicitud,  
    *entonces* la entrada se suprime de su lista personal, se eliminan sus calificaciones y notas privadas asociadas y el título vuelve a mostrarse en estado no registrado.
  - **Criterio 2 (Cancelación de modal de borrado):**  
    *Dado que* el usuario abre el modal de confirmación pero selecciona "Cancelar",  
    *cuando* se cierra el diálogo,  
    *entonces* la entrada se mantiene exactamente en el mismo estado sin alteración alguna.

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
  - **Criterio 3 (Límite anti-spam de solicitudes - RN-032):**  
    *Dado que* el usuario ya alcanzó el cupo de 20 solicitudes salientes en `PENDING` o emitió 25 solicitudes en las últimas 24 horas,  
    *cuando* intenta enviar una nueva solicitud de amistad,  
    *entonces* el sistema bloquea el envío informando que ha alcanzado el límite permitido temporalmente.

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
  - **Criterio 3 (Saturación de recomendaciones hacia el amigo - RN-033):**  
    *Dado que* "Kuro" ya acumula 15 recomendaciones en estado `PENDING` enviadas por el usuario,  
    *cuando* el usuario intenta enviar una recomendación adicional,  
    *entonces* el sistema le indica *"Tu amigo tiene 15 recomendaciones pendientes tuyas; espera a que las revise antes de enviarle más"*.
  - **Criterio 4 (Amigo tiene el anime en DROPPED - RN-007):**  
    *Dado que* "Kuro" tiene la obra en estado `DROPPED`,  
    *cuando* el usuario intenta seleccionarlo como destinatario,  
    *entonces* el sistema despliega un aviso informando *"Kuro marcó esta obra como abandonada. ¿Deseas enviarle tu recomendación igualmente?"*; si el usuario confirma, la recomendación se crea en estado `PENDING`.

#### F-SOC-003: Aceptar o descartar recomendación en buzón
- **Historia de Usuario:**  
  *Como* usuario receptor,  
  *quiero* gestionar las recomendaciones recibidas en mi buzón,  
  *para* incorporar a mis pendientes las que me interesen o archivar las que no.
- **Criterios de Aceptación:**
  - **Criterio 1 (Aceptación sin registro previo - Regla RN-009):**  
    *Dado que* el usuario tiene una recomendación pendiente en su buzón de un anime no registrado en su lista,  
    *cuando* presiona *"Aceptar recomendación"*,  
    *entonces* la recomendación cambia a `ACCEPTED`, y el anime se incorpora automáticamente a la lista personal en estado `PLANNING`.
  - **Criterio 2 (Descarte de recomendación):**  
    *Dado que* el usuario no desea ver el anime recomendado,  
    *cuando* presiona *"Descartar"*,  
    *entonces* la recomendación pasa a estado `DISMISSED` y no se altera su lista personal.
  - **Criterio 3 (Aceptación cuando ya figuraba en PLANNING, ON_HOLD o DROPPED - RN-009):**  
    *Dado que* el anime ya figuraba previamente en la lista del usuario en estado `PLANNING`, `ON_HOLD` o `DROPPED`,  
    *cuando* el usuario presiona *"Aceptar recomendación"*,  
    *entonces* la recomendación pasa a `ACCEPTED`, se actualiza la entrada vinculada sin duplicarla y el sistema le ofrece reactivar el anime al estado `PLANNING` o `WATCHING`.

#### F-SOC-004: Gestión de solicitudes de amistad (Aceptar, Rechazar con advertencia y Cancelar)
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* aceptar, rechazar o cancelar solicitudes de amistad con confirmación informada,  
  *para* gestionar mi círculo social evitando rechazos accidentales.
- **Criterios de Aceptación:**
  - **Criterio 1 (Aceptación de solicitud):**  
    *Dado que* el usuario recibe una solicitud de "Sakura" en estado `PENDING`,  
    *cuando* presiona "Aceptar",  
    *entonces* la solicitud pasa a `ACCEPTED`, se establece la amistad bilateral y ambos quedan habilitados para recomendarse animes y consultar listas si están en `FRIENDS_ONLY`.
  - **Criterio 2 (Advertencia y confirmación de rechazo con enfriamiento de 24 hs - RN-029):**  
    *Dado que* el usuario presiona "Rechazar" en una solicitud entrante de "Sakura",  
    *cuando* el sistema despliega el diálogo de confirmación indicando *"¿Rechazar solicitud? Sakura no podrá volver a enviarte una solicitud durante 24 horas"* y el usuario confirma,  
    *entonces* la solicitud se elimina/rechaza y se activa la ventana de enfriamiento de 24 horas para Sakura.
  - **Criterio 3 (Cancelación de diálogo de rechazo):**  
    *Dado que* el usuario visualiza la advertencia de rechazo pero presiona "Cancelar",  
    *cuando* se cierra el modal,  
    *entonces* la solicitud permanece en estado `PENDING` sin cambios.
  - **Criterio 4 (Cancelación de solicitud saliente):**  
    *Dado que* el usuario emitió una solicitud que aún figura `PENDING`,  
    *cuando* presiona "Cancelar solicitud",  
    *entonces* la solicitud se retira inmediatamente y se aplica el enfriamiento de 24 horas para volver a emitirla.

#### F-SOC-005: Disolución de amistad (Eliminar amigo)
- **Historia de Usuario:**  
  *Como* usuario con amistades activas,  
  *quiero* eliminar a un usuario de mi lista de amigos,  
  *para* finalizar el vínculo bilateral cuando ya no desee compartir recomendaciones.
- **Criterios de Aceptación:**
  - **Criterio 1 (Eliminación exitosa - RN-030):**  
    *Dado que* el usuario tiene amistad aceptada con "Kuro",  
    *cuando* selecciona "Eliminar de mis amigos" y confirma la acción,  
    *entonces* la amistad se disuelve de inmediato para ambos, se descartan las recomendaciones `PENDING` en ambas direcciones y se revoca el acceso a listas configuradas en `FRIENDS_ONLY`.
  - **Criterio 2 (Conservación de animes aceptados):**  
    *Dado que* "Kuro" había aceptado previamente animes recomendados por el usuario,  
    *cuando* se disuelve la amistad,  
    *entonces* las obras permanecen intactas en las listas personales de ambos.

#### F-SOC-006: Bloqueo y desbloqueo de usuarios
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* bloquear a usuarios no deseados o desbloquearlos cuando lo considere,  
  *para* salvaguardar mi privacidad y evitar interacciones molestas.
- **Criterios de Aceptación:**
  - **Criterio 1 (Bloqueo silencioso e inmediato - RN-031):**  
    *Dado que* el usuario selecciona "Bloquear usuario" sobre "Spammer" y confirma,  
    *cuando* se ejecuta el bloqueo,  
    *entonces* se crea el registro en `UserBlock`, se disuelve cualquier amistad previa, se suprimen las solicitudes y recomendaciones `PENDING` en ambas direcciones y el perfil del usuario queda completamente invisible para "Spammer" sin emitir notificación alguna a este último.
  - **Criterio 2 (Desbloqueo desde ajustes):**  
    *Dado que* el usuario consulta su sección de "Usuarios bloqueados" en ajustes,  
    *cuando* presiona "Desbloquear" junto al nombre de un usuario,  
    *entonces* se elimina el registro de `UserBlock` restableciendo la visibilidad normal (sin restaurar automáticamente amistades previas).

---

### Módulo: Descubrimiento y Afinidad (REC)

#### F-REC-001: Sugerencias automáticas por afinidad
- **Historia de Usuario:**  
  *Como* usuario con historial de visualización,  
  *quiero* recibir sugerencias generadas por el sistema basadas en mis preferencias de géneros y estudios,  
  *para* descubrir series afines con una justificación transparente y natural de por qué me fueron recomendadas.
- **Criterios de Aceptación:**
  - **Criterio 1 (Generación de recomendaciones con explicación limpia - RN-036 y RN-037):**  
    *Dado que* el usuario tiene al menos 3 animes calificados con nota $\ge 7$,  
    *cuando* accede a su sección de descubrimiento,  
    *entonces* el sistema le presenta títulos ordenados por el scoring ponderado (60% géneros, 25% estudio, 15% calidad global) acompañados de una justificación natural y limpia sin paréntesis ni tecnicismos numéricos (ej. *"Sugerido por tus altas valoraciones en Ciencia Ficción y Suspenso"* o *"Sugerido por tu preferencia hacia el estudio MAPPA"*).
  - **Criterio 2 (Arranque en frío / Cold Start - RN-038):**  
    *Dado que* el usuario no posee al menos 3 animes calificados con nota $\ge 7$,  
    *cuando* ingresa a la sección de descubrimiento,  
    *entonces* el sistema le presenta la cartelera de tendencias globales de la temporada actual invitándolo a calificar sus primeros animes para activar su motor de afinidad.
  - **Criterio 3 (Exclusión estricta de lista personal - RN-010):**  
    *Dado que* un anime candidato coincide en género y estudio pero ya está registrado en la lista del usuario en cualquiera de sus cinco estados (`PLANNING`, `WATCHING`, `COMPLETED`, `ON_HOLD` o `DROPPED`),  
    *cuando* el motor genera las sugerencias,  
    *entonces* dicho anime es omitido rigurosamente del conjunto de resultados.

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
  - **Criterio 2 (Secuelas multiformato en película u OVA - RN-043):**  
    *Dado que* el usuario completó una serie de televisión y se estrena una película secuela canónica (`MOVIE`) o un `OVA` de continuación oficial,  
    *cuando* se procesa la novedad en el catálogo,  
    *entonces* el feed de novedades presenta la tarjeta destacando el formato cinematográfico/OVA y ofreciendo guardarla directamente en pendientes.

---

### Módulo: Notificaciones (NTF)

#### F-NTF-001: Centro de notificaciones In-App
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* ver una lista de mis notificaciones recientes con indicador de no leídas,  
  *para* enterarme de recomendaciones y solicitudes de mis amigos directamente en la plataforma.
- **Criterios de Aceptación:**
  - **Criterio 1 (Distintivo visual y contador de no leídas):**  
    *Dado que* el usuario recibe una recomendación o solicitud de amistad,  
    *cuando* navega por la interfaz web,  
    *entonces* visualiza un distintivo numérico sobre el icono de campana indicando la cantidad de notificaciones no leídas.
  - **Criterio 2 (Interacción y marcado como leída):**  
    *Dado que* el usuario abre el centro de notificaciones y selecciona un elemento,  
    *cuando* el sistema lo redirige a la acción vinculada (ej. al buzón o lista de amigos),  
    *entonces* la notificación pasa a `leida = true` y el contador de notificaciones no leídas disminuye inmediatamente.

#### F-NTF-002: Gestión de preferencias de notificación por Email (Opt-in)
- **Historia de Usuario:**  
  *Como* usuario registrado,  
  *quiero* configurar explícitamente qué notificaciones deseo recibir en mi correo electrónico,  
  *para* mantener el control total sobre los mensajes enviados a mi casilla.
- **Criterios de Aceptación:**
  - **Criterio 1 (Gestión en ajustes):**  
    *Dado que* el usuario ingresa a la configuración de su cuenta,  
    *cuando* activa o desactiva los interruptores de notificación por email (Recomendaciones, Solicitudes de amistad o Secuelas) y guarda los cambios,  
    *entonces* el sistema actualiza su registro de `PreferenciasNotificacion`.
  - **Criterio 2 (Respeto al estado desactivado por defecto - RN-018):**  
    *Dado que* un usuario tiene en `false` la opción de correos por recomendaciones y recibe una recomendación de un amigo,  
    *cuando* se procesa el evento,  
    *entonces* se crea la notificación In-App pero **no** se despacha ningún email a través de Amazon SES.

---

## 6. Alcance del proyecto (Scope Management)

### DENTRO del alcance — Versión 1.0
- Autenticación delegada multi-proveedor (Social Login: Google, Discord y SaaS IdP OIDC) y gestión básica de perfil de usuario con onboarding de `username`.
- Búsqueda de anime contra catálogo externo oficial.
- Gestión completa de lista personal de animes (5 estados canónicos, progreso de episodios, nota 1-10, notas privadas).
- Sistema de amistad bilateral consentida.
- Buzón de recomendaciones entre amigos con notas personalizadas y aceptación directa hacia `PLANNING`.
- Algoritmo de afinidad por géneros/estudios con explicabilidad de sugerencia.
- Feed de tendencias de temporada y detección de secuelas de títulos seguidos.
- Centro de notificaciones In-App y servicio desacoplado de email transaccional (Amazon SES) con opt-in estricto.
- Control granular de privacidad para lista personal y notas personales (`PUBLIC`, `FRIENDS_ONLY`, `PRIVATE`).

### FUERA del alcance (Explícito para Versión 1.0)
- **Almacenamiento o gestión de contraseñas locales:** Amane nunca almacena contraseñas; la gestión de credenciales y recuperación de contraseñas queda delegada íntegramente a los proveedores de identidad externos.
- **Grupos de amigos y recomendaciones colectivas:** Planificado formalmente para la versión 1.1; en la V1.0 el intercambio social es estrictamente bilateral (1 a 1 entre dos amigos).
- **Streaming y reproducción de video:** Amane no es una plataforma de contenido audiovisual.
- **Red social abierta:** Sin foros de discusión, comentarios públicos en fichas de animes ni muros globales.
- **Redacción editorial de noticias:** Las novedades son procesadas automáticamente desde los metadatos de las APIs externas.
- **Catálogo de Manga y Novelas Ligeras:** Exclusivo para anime en la V1.0.
- **Integración con Discord Bot:** Planificada formalmente para la versión 2.0 (se diseñará el núcleo en el `architect.md` desacoplado mediante arquitectura hexagonal para habilitar el bot en V2 sin rehacer el dominio).
- **Importación/Exportación de listas (XML/JSON de MAL):** Postergado para versión posterior a la v1.0.

---

