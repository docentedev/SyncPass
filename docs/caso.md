# Formato de Caso para Docentes y Estudiantes — SyncPass
**Desarrollo de Aplicaciones Móviles · DSY1105**
**Duoc UC · Escuela de Informática y Telecomunicaciones · CITT**

## 1. Identificación del caso

| Antecedente | Información para el caso |
|---|---|
| Título breve del desafío | SyncPass |
| Organización | TI CONSULTING DIEGO GARCÉS MIRANDA E.I.R.L. |
| Rubro o ámbito | Consultoría, asesoría y desarrollo de software |
| Sede / coordinador(a) | CITT Plaza Vespucio / Nelson Melo |
| Asignatura | DSY1105 · Desarrollo de Aplicaciones Móviles |
| Fecha de entrega al docente | 04-09-2026 |

---

## 2. Caso que se presentará a los equipos

### 2.1 Contexto de la organización

**Ticdgm** es una consultora y agencia tecnológica enfocada en impulsar la transformación digital mediante soluciones integrales de TI adaptadas a las necesidades de cada cliente. Su oferta incluye:

- **Desarrollo TI:** Creación de software a medida, aplicaciones móviles, plataformas web y productos mínimos viables (MVP) escalables.
- **Consultoría y Asesorías:** Diagnóstico, optimización operativa, modelado de datos, diseño de arquitectura de software y gobernanza tecnológica estratégica.
- **Capacitación Continua:** Programas formativos en desarrollo, bases de datos, metodologías ágiles y herramientas emergentes.

La consultora atiende a diversos sectores que buscan optimizar sus procesos:

- **Instituciones Educativas y de Innovación:** Digitalización académica, gestión de proyectos y programas especializados.
- **Pymes y Startups:** Automatización de flujos de trabajo, modernización de bases de datos y validación de MVPs.
- **Fundaciones y Organizaciones Comunitarias:** Herramientas para gestión en terreno, vinculación y medición de impacto.
- **Equipos Técnicos TI:** Asesoría avanzada en arquitectura, gobernanza y entrenamiento especializado.

### 2.2 Situación actual

Actualmente, la gestión de los proyectos de vinculación comunitaria se sostiene por completo en herramientas manuales y desarticuladas, sin un sistema digital que unifique el registro y seguimiento de las actividades en terreno.

El flujo actual es:

1. **Planificación:** El Docente Líder o Coordinador(a) de Proyecto define objetivos, cupos, fechas, la unidad vecinal y los requerimientos de cada taller o actividad. La convocatoria se difunde por canales informales —redes sociales, correos y grupos de mensajería— y los beneficiarios se inscriben mediante planillas compartidas o formularios web aislados, sin ninguna integración entre sí.
2. **Día del evento:** Los Estudiantes Voluntarios y el/la Coordinador(a) se presentan en la unidad vecinal, colegio o espacio comunitario asignado, donde la asistencia se sigue tomando en una planilla impresa: cada participante debe escribir su nombre, identificador y firma de puño y letra al ingresar.
3. **Post-evento:** El Equipo de Coordinación intenta levantar la percepción de los asistentes enviando, días o incluso semanas después, un enlace a un formulario de satisfacción en línea a los correos o teléfonos recopilados en las listas de papel.
4. **Consolidación:** El Equipo Administrativo y Docente debe transcribir manualmente los registros en papel hacia planillas de cálculo institucionales, verificar que los datos coincidan, cruzar las horas de participación de los estudiantes y calcular los promedios de satisfacción para elaborar el informe de cierre semestral.

> Este es el estado actual del proceso, completamente dependiente de soporte físico y de la gestión manual de la información.

### 2.3 Necesidad o problema

**Frecuencia del problema:** Ocurre de manera sistemática y continua cada semana durante todo el semestre académico, intensificándose durante los meses de mayor actividad territorial y en los periodos de cierre de hitos institucionales.

#### Manifestación del problema en terreno

- **Pérdida y deterioro de evidencia:** Las hojas de asistencia en papel frecuentemente se extravían, se dañan por traslados en terreno o resultan ilegibles debido a mala caligrafía o registros incompletos.
- **Congestión en accesos:** El registro manual genera filas y demoras de hasta 15 o 20 minutos en la entrada de las actividades, restando tiempo efectivo al desarrollo del taller o servicio.
- **Deserción en la evaluación de impacto:** Al aplicar las encuestas de valoración días después por correo electrónico, la tasa de respuesta efectiva cae por debajo del 15%, impidiendo medir el impacto real y la percepción cualitativa de la comunidad.
- **Falta de conectividad estable:** Varias de las unidades vecinales periféricas presentan baja o nula cobertura de red móvil, lo que inhabilita el uso de aplicaciones web tradicionales dependientes de conexión permanente.

#### Consecuencias operativas y de gestión

- Pérdida de cientos de horas-hombre semestrales dedicadas exclusivamente a la digitación y corrección de datos.
- Descuadres en las métricas institucionales de asistencia auditadas para acreditación y reportabilidad de Vinculación con el Medio.
- Falta de trazabilidad en tiempo real sobre el quórum real de los proyectos y la satisfacción inmediata del beneficiario.

### 2.4 Mejora esperada

La implementación del proyecto busca transformar la gestión operativa en terreno de las actividades comunitarias, logrando un impacto significativo en los siguientes ámbitos:

- **Digitalización y eliminación del papel al 100%:** Sustituir completamente las planillas impresas de asistencia por un sistema de acreditación digital directa mediante lectura de códigos QR, eliminando el riesgo de extravío, deterioro o ilegibilidad de documentos físicos.
- **Agilización del flujo de acceso y acreditación:** Disminuir el tiempo de registro por participante de 45 segundos (manual) a menos de 5 segundos, descongestionando la entrada de los recintos y evitando retrasos en el inicio de las actividades.
- **Garantía de operatividad continua (Modo Offline-First):** Asegurar el 100% de disponibilidad operacional en zonas periféricas con cobertura nula o intermitente de red móvil, mediante el almacenamiento local de datos transaccionales en el dispositivo y su sincronización automática posterior al recuperar la señal.
- **Aumento sustancial en la tasa de respuesta de encuestas de impacto:** Incrementar la completitud de las encuestas de satisfacción y valoración de un 15% histórico a más del 65%, al capturar la percepción de los asistentes de forma inmediata e interactiva al finalizar el evento.
- **Reducción de la carga y costos administrativos:** Disminuir en al menos un 80% las horas-hombre dedicadas a la digitación, corrección y consolidación manual de planillas, automatizando la generación de reportes e informes de impacto institucionales.
- **Trazabilidad e integridad de los datos en tiempo real:** Disponer de información confiable con marcas de tiempo (timestamps) y coordenadas geográficas referenciales, permitiendo el monitoreo en vivo del quórum de los proyectos y la auditoría transparente de métricas.

---

## 3. Información necesaria para desarrollar la propuesta

### 3.1 Tareas o funciones de los perfiles usuarios

#### Docente / Coordinador(a) de Proyecto

- **Tareas actuales:** Crea y revisa las fichas de los proyectos, consulta listas previas de inscritos, supervisa el cumplimiento del quórum mínimo en el lugar y redacta los informes de impacto.
- **Dificultades:** Carece de indicadores en tiempo real durante la ejecución del evento, depende de la entrega física de listas y no cuenta con retroalimentación inmediata sobre la calidad de la sesión.

#### Estudiante Voluntario / Monitor

- **Tareas actuales:** Recibe a los asistentes en el acceso, busca los nombres en listas impresas, anota a quienes llegan sin inscripción previa y coordina el flujo de personas.
- **Dificultades:** Enfrenta aglomeraciones en la entrada, lidia con errores de lectura de datos personales y no dispone de herramientas para registrar asistencia si no hay señal de internet en el recinto.

#### Beneficiario Comunitario / Asistente

- **Tareas actuales:** Hace filas para identificarse y firmar en la entrada; si recibe un correo posterior, debe recordar la experiencia para calificar la actividad.
- **Dificultades:** Proceso de entrada lento; el llenado de formularios largos post-evento resulta invasivo y suele ser ignorado.

### 3.2 Información necesaria para el proceso

- **Proyectos Comunitarios:** Identificador, título, descripción, área temática (ej. tecnología, salud, emprendimiento), unidad vecinal responsable, fecha de inicio y término, estado.
- **Eventos / Talleres:** Identificador del evento, proyecto asociado, nombre de la actividad, fecha/hora programada, cupos máximos, recinto/dirección, coordenadas geográficas aproximadas.
- **Asistentes / Participantes:** Identificador sintético/ficticio, tipo de participante (Estudiante / Vecino / Docente), estado de inscripción.
- **Registro de Asistencia:** Código del evento, identificador del asistente, fecha y hora exacta de marcación (timestamp), estado de validación y estado de sincronización.
- **Encuestas de Valoración:** Escala de satisfacción del evento (1 a 5 estrellas / Likert), nivel de utilidad del taller, comentarios u observaciones cualitativas, fecha de envío.

### 3.3 Condiciones de uso de la aplicación móvil

- **Dispositivo:** La aplicación funcionará exclusivamente en teléfonos inteligentes con sistema operativo Android.
- **Conectividad:** Debe poder utilizarse tanto con conexión a internet (Wi-Fi o datos móviles) como sin conexión, en unidades vecinales sin cobertura.
- **Cámara:** Se utilizará para la lectura de códigos QR al momento de registrar la asistencia.
- **Ubicación:** Se utilizará de forma referencial para verificar que la marcación de asistencia ocurra dentro del radio de la unidad vecinal donde se realiza el evento.
- **Notificaciones:** Avisos visuales inmediatos ante cada acción, por ejemplo confirmaciones de registro o pendientes de sincronización.
- **Usabilidad y accesibilidad:**
  - Interfaz simple y optimizada para uso en terreno, con buena legibilidad bajo condiciones de luz variables y compatibilidad con modo claro y oscuro.
  - Elementos y botones de tamaño amplio y cómodo, pensados para poder operarse con una sola mano mientras se atiende a los asistentes.
  - Retroalimentación visual inmediata ante cada acción, mediante mensajes claros de confirmación, error o estado.
  - Lenguaje simple y cercano en la interfaz, adecuado para personas con distintos niveles de familiaridad con la tecnología.

### 3.4 Conexión con otros sistemas o herramientas

- **API RESTful institucional (backend):** La aplicación debe interactuar con servicios backend para autenticación de usuarios por roles, descarga periódica de proyectos y eventos, y envío por lotes (batch sync) de los registros de asistencia y encuestas acumuladas en modo offline.
- **Capa de persistencia local embebida:** Integración con una base de datos relacional local (Room / SQLite) en el dispositivo móvil para almacenar en caché el catálogo de actividades y gestionar las colas de transacciones pendientes por enviar.
- **Mecanismos de sincronización:** Protocolo de sincronización bidireccional que verifique marcas temporales (timestamps) y banderas de sincronización (`is_synced`), resolviendo de forma segura la subida de datos una vez que se restablezca el canal HTTP/HTTPS.

---

## 4. Criterios para considerar útil la propuesta

### 4.1 Resultados o señales de éxito

En base al requerimiento y contexto operativo del proyecto, los resultados o señales de éxito previstos para medir el impacto de la solución son:

- **Tiempos de acreditación drásticamente reducidos:** Disminución del tiempo promedio de registro por persona de 45 segundos (método manual) a menos de 5 segundos mediante la lectura de códigos QR dinámicos.
- **Aumento sustancial en la tasa de respuesta de encuestas:** Incremento en la completitud de las encuestas de valoración del evento, pasando de un 15% histórico a más del 65% al aplicarse inmediatamente en terreno.
- **Cero pérdida de información:** Garantía de integridad total sobre los registros capturados, eliminando las pérdidas de datos causadas por extravío o deterioro físico de planillas en papel.
- **Disponibilidad operativa del 100% sin conexión:** Funcionamiento continuo en terreno sin interrupciones por falta o caída de señal móvil, gracias al uso de persistencia local y sincronización en segundo plano.
- **Eficiencia administrativa y ahorro de horas-hombre:** Reducción de al menos un 80% en las horas dedicadas a la transcripción, digitación y consolidación manual de reportes de asistencia al cierre de cada periodo.
- **Trazabilidad y visibilidad en tiempo real:** Disponibilidad inmediata de indicadores de asistencia, marcas temporales (timestamps), coordenadas de ubicación referenciales y métricas de satisfacción para la supervisión docente y la reportabilidad institucional.

---

## 5. Restricciones que deben considerar los equipos

### 5.1 Condiciones o límites del caso

Las restricciones que deben considerar los equipos para el desarrollo del proyecto son:

- **Alcance estrictamente académico (MVP):** El entregable se limita a un Producto Mínimo Viable (MVP) funcional para evaluación académica y no contempla el desarrollo definitivo ni la publicación productiva de la aplicación.
- **Prohibición del uso de datos reales:** Está estrictamente prohibido utilizar datos personales reales (RUT, nombres, correos, teléfonos, direcciones o datos sensibles/biométricos); todas las pruebas, demostraciones y bases de datos deben emplear información 100% ficticia o anonimizada.
- **Restricción de credenciales y sistemas internos:** No se entregarán ni utilizarán llaves de acceso, tokens, credenciales productivas, respaldos de información ni accesos a los sistemas o servidores internos del socio formador.
- **Uso regulado de sensores del dispositivo:** La captura de recursos como la cámara (para escaneo de códigos QR) y el GPS (para validación de ubicación) debe probarse o simularse sin almacenar datos personales ni identificar a personas reales.
- **Stack tecnológico obligatorio:** La solución debe construirse obligatoriamente con el stack del programa académico: Kotlin, Android Studio, Jetpack Compose, Material Design 3, arquitectura MVVM, persistencia en Room/SQLite y backend con API REST en Spring Boot mediante Retrofit.
- **Compatibilidad de hardware y usabilidad:** La aplicación Android debe ser compatible con la versión 8.0 (API 26) o superior (e iOS 15.0+ si aplica), asegurando adaptabilidad a distintas pantallas y áreas táctiles mínimas de 48x48 dp.
- **Privacidad en repositorios y entregas:** Todas las capturas de pantalla, presentaciones, videos de demostración y repositorios en Git/GitHub deben contener únicamente información simulada, debiendo eliminarse las copias de trabajo al concluir el semestre.

---

## 6. Material de apoyo que se entregará

Registrar únicamente material necesario, depurado y autorizado para uso académico.

| Material | Para qué se utilizará | Revisión |
|---|---|---|
|  |  | ☐ Depurado |
|  |  | ☐ Depurado |
|  |  | ☐ Depurado |

---

*Fuente: `docs/caso.pdf` (8 páginas) — Formato de caso DSY1105. Convertido a Markdown para mejor comprensión por IA y uso como contexto del proyecto SyncPass.*
