# 🎓 Tutor Académico Inteligente (RAG) — Nivelación de Estudios

> **Proyecto desarrollado para el Challenge de Alura y Oracle.**
> Un asistente de Inteligencia Artificial móvil, flexible y disponible 24/7, diseñado con el objetivo que el estudiante pueda culminar con éxito la Enseñanza Media (Secundaria / Bachillerato) en Chile y Latinoamérica (próximamente).

### 🤖 [Prueba el bot aquí: @TutorConIAbot](https://t.me/TutorConIAbot)

---

## 📑 Tabla de contenidos

- [El impacto social y la problemática](#-el-impacto-social-y-la-problemática)
- [Mi solución](#-mi-solución)
- [Canal de interacción: Telegram](#-canal-de-interacción-telegram)
- [El corazón del proyecto: personalización y contexto real](#-el-corazón-del-proyecto-personalización-y-contexto-real)
- [Arquitectura técnica (RAG)](#️-arquitectura-técnica-rag)
- [Stack tecnológico](#️-stack-tecnológico)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Instalación y despliegue](#-instalación-y-despliegue)
- [Demo y capturas](#-demo-y-capturas)
- [Futuras versiones y comunidad](#-futuras-versiones-y-comunidad)
- [Autor](#-autor)

---

## 🌎 El impacto social y la problemática

En Chile —y en gran parte de Latinoamérica— miles de personas adultas se enfrentan a la necesidad urgente de culminar su educación obligatoria, la **Enseñanza Media** (equivalente a la secundaria, el bachillerato o el _high school_ en otros países). Esta necesidad responde a dos realidades críticas:

1. **Reincorporación laboral:** personas desvinculadas que, al intentar postular a un nuevo empleo, descubren que el mercado actual exige la Enseñanza Media completa — cuando en su época el mínimo requerido para trabajar era la educación básica (8° básico / primaria).
2. **Ascenso profesional:** trabajadores activos cuyos contratos o la propia legislación les exigen el título escolar completo para optar a capacitaciones o ascender de puesto.

### ❌ Las barreras tradicionales

- **Frustración y vergüenza:** volver a un aula física después de décadas es complejo y genera desmotivación.
- **Falta de tiempo:** las clases presenciales rígidas chocan con las jornadas laborales y familiares.
- **Rigidez de horarios:** la vida adulta no siempre calza con un horario de clases fijo.

### 💡 Mi solución

Transformar cualquier dispositivo móvil en un espacio de estudio dinámico, interactivo y disponible las 24 horas, adaptando el aprendizaje al ritmo de cada estudiante — y haciéndolo mucho más entretenido incorporando una sección donde se le pregunta al alumno por sus hobbies, sus actividades diarias o en qué trabaja, para que cada ejercicio y ejemplo sea completamente personalizado para él.

---

## 🤖 Canal de interacción: Telegram

El tutor está desplegado y disponible directamente en **Telegram**, para que el estudiante lo use desde su celular sin instalar ninguna aplicación nueva.

- **Recepción de mensajes:** nodo `Telegram Trigger` de n8n, que escucha cada mensaje que el alumno envía al bot.
- **Envío de respuestas:** nodo `Telegram Send Message`, configurado con `parse_mode: HTML`.
- **Formato de las respuestas:** la primera instrucción del `systemMessage` del agente — antes incluso de explicar las modalidades — es la prohibición absoluta de usar sintaxis Markdown (`**negrita**`, `*cursiva*`, `_cursiva_`, `#` para títulos), porque Telegram no la interpreta de forma nativa y el texto llegaría roto al estudiante. En su lugar, todas las respuestas usan únicamente etiquetas HTML (`<b>`, `<i>`) para negritas y cursivas, garantizando que el formato se vea correctamente tanto en la app móvil como en la de escritorio.
- **Memoria por usuario:** cada conversación se identifica con el `id` de Telegram del alumno (`sessionKey` personalizado en el nodo de memoria), así el bot recuerda su modalidad y sus gustos entre sesiones distintas sin mezclarlos con los de otro estudiante.

---

## ⚡ El corazón del proyecto: personalización y contexto real

Para que la enseñanza sea efectiva y no frustrante, el tutor se rige bajo una regla fundamental:

> 💡 **"Aprender para trabajar... no es lo mismo que aprender para seguir estudiando."**

El agente identifica y separa estas dos realidades evaluativas con precisión:

### 💼 1. Modalidad: Fines Laborales

- **Qué es:** certificación de competencias orientada a obtener la licencia de escolaridad obligatoria completa exclusivamente para efectos de trabajo.
- **Contenido:** basado directamente en las normativas y exámenes oficiales en PDF del Ministerio de Educación (MINEDUC).
- **Dinámica:** la IA realiza un perfilamiento empático, preguntándole al alumno por sus pasatiempos, su trabajo o sus actividades diarias. Cada ejercicio se redacta de forma personalizada usando esos intereses — por ejemplo, si le gusta el fútbol, los problemas de matemáticas se plantean con estadísticas de su equipo favorito.

### 🎓 2. Modalidad: Continuidad de Estudios

- **Qué es:** obtener la licencia de Enseñanza Media con el objetivo de continuar estudios en Educación Superior (CFT, Institutos Profesionales, Universidades) o acceder a capacitaciones y cursos como SENSE.
- **Contenido:** basado en mallas curriculares extensas para la preparación de exámenes de validación de estudios.
- **Asignaturas:** Matemáticas, Lenguaje y Comunicación, Inglés, Ciencias Sociales y Ciencias Naturales.

### 🧠 Cómo funciona el flujo de conversación

1. **Selección de modalidad:** al iniciar, el bot pregunta si el usuario viene por Fines Laborales o por Continuidad de Estudios, y guarda esa elección en memoria para no volver a preguntarla.
2. **Perfilamiento de hobbies:** si es la primera vez que el usuario pide practicar, el bot le pregunta brevemente por sus intereses (deportes, videojuegos, trabajo, vida cotidiana) y también los guarda en memoria.
3. **Clasificación de la materia (Regla 0):** antes de generar cualquier pregunta, el agente clasifica la materia solicitada en uno de dos grupos:
   - **Grupo A — Adaptable:** Matemáticas y Lenguaje y Comunicación (en ambas modalidades) e Inglés (en Continuidad de Estudios). Aquí el enunciado sí se puede ambientar con los gustos del alumno, manteniendo exactamente los mismos números, dimensiones y clave de respuesta del documento original.
   - **Grupo B — No adaptable:** Ciencias Sociales, Historia y Ciencias Naturales. Aquí los hobbies del usuario nunca se mezclan con el contenido de la pregunta — se respeta el hecho histórico o científico real tal como aparece en el temario oficial indexado en Qdrant.

### 🏢 3. Consultas institucionales e información general

El alumno no necesita salir de su entorno de estudio para resolver dudas administrativas. Puede consultar directamente en el chat:

- Horarios de atención y oficinas de evaluación.
- Escalas de notas, calificaciones mínimas y máximas de aprobación.
- Requisitos de inscripción y detalles administrativos del proceso.

---

## ⚙️ Arquitectura técnica (RAG)

El sistema utiliza una arquitectura **RAG (Retrieval-Augmented Generation)** construida en **n8n** como orquestador principal, dividida en dos flujos independientes.

```text
┌────────────────────────────┐
│  Documentos PDF oficiales   │
│  (MINEDUC)                  │
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│  Flujo de Ingesta y          │
│  Dosificación (n8n)          │
│  Loops + Wait anti rate-limit│
└──────────────┬──────────────┘
               │
┌──────────────▼──────────────┐
│  Qdrant Vector Database      │
│  ┌─────────────────────┐    │
│  │ fines_laborales      │    │
│  ├─────────────────────┤    │
│  │ continuidad_estudios  │    │
│  ├─────────────────────┤    │
│  │ general               │    │
│  └─────────────────────┘    │
└──────────────▲──────────────┘
               │ (consulta semántica)
┌──────────────┴──────────────┐
│  Agente de Consulta IA       │
│  (Gemini 3.1 Flash Lite       │
│   + fallback 2.0, Tools Agent│
│   + memoria)                 │
└──────────────▲──────────────┘
               │
        [ Telegram Bot ]
               │
          [ Estudiante ]
```

### 1. Flujo de Ingesta y Dosificación (backend de datos)

- **Problema:** enviar documentos masivos a la API de embeddings en paralelo satura el nivel gratuito de Gemini y provoca caídas por límite de uso (`Error 429 — Too Many Requests`).
- **Solución:** el flujo lee los PDF de cada carpeta (`fines_laborales`, `continuidad_de_estudios`, `general`), extrae su texto de forma nativa, lo fragmenta con un Text Splitter y lo procesa mediante **dosificadores secuenciales (`Loop Over Items`) con tiempos de espera (`Wait`)** entre cada documento, evitando saturar la API.
- **Persistencia:** los fragmentos se vectorizan con **Gemini Embeddings** y se almacenan de forma permanente en **Qdrant**, separados en tres colecciones independientes: `fines_laborales`, `continuidad_estudios` y `general`.

### 2. Flujo del Agente de Consulta (experiencia de usuario)

- **Procesamiento:** utiliza el modelo **Gemini 3.1 Flash Lite** en modo _Tools Agent_, con un segundo modelo (**Gemini 2.0 Flash Lite**) configurado como respaldo automático ante fallos del modelo principal.
- **Memoria:** conserva los últimos 30 mensajes de la conversación (`Simple Memory`, ventana de contexto de 30), identificada por el `id` de Telegram del alumno.
- **Ruteo semántico de colecciones:** en lugar de buscar en un único bloque masivo de datos, el agente evalúa la intención de la consulta y activa específicamente la herramienta de Qdrant asociada a la colección correcta (`fines_laborales`, `continuidad_estudios` o `general`), maximizando la precisión de la respuesta y evitando mezclar temarios.
- **Personalización dirigida por prompt:** el agente sigue la Regla 0 descrita más arriba para decidir cuándo puede y cuándo no puede mezclar los gustos del alumno con el contenido de la pregunta, y luego califica sus respuestas usando la pauta de corrección incluida en el propio documento recuperado.

---

## 🛠️ Stack tecnológico

La arquitectura fue pensada bajo principios de robustez y rentabilidad, sosteniendo el servicio de forma gratuita y lista para producción:

| Tecnología | Rol |
| --- | --- |
| ⚙️ **n8n** | Orquestador visual de los flujos de trabajo y la lógica del agente |
| 🤖 **Telegram Bot API** | Canal de interacción con el estudiante (mensajes entrantes y salientes en formato HTML) |
| 🧠 **Google Gemini 3.1 Flash Lite** | Modelo de lenguaje (LLM) principal, de alta velocidad y bajo costo |
| 🔁 **Google Gemini 2.0 Flash Lite** | Modelo de respaldo (*fallback*): si el modelo principal falla, el agente reintenta automáticamente con este modelo, sin interrumpir la conversación del estudiante |
| 🏷️ **Gemini Embeddings** | Modelo que codifica consultas y fragmentos de texto en vectores |
| 🗄️ **Qdrant** | Base de datos vectorial persistente para el almacenamiento y la segmentación semántica |
| 🐳 **Docker / Docker Compose** | Orquestación de los contenedores de n8n, Qdrant y Caddy |
| 🔒 **Caddy Server** | Reverse proxy con emisión y renovación automática de certificados SSL/TLS |
| ☁️ **Google Cloud Platform (Compute Engine)** | Máquina virtual con IP pública donde corre toda la infraestructura, 24/7 |

---

## 📂 Estructura del repositorio

```text
CHALLENGE/
├── backend-n8n/
│   ├── documentos/
│   │   ├── continuidad_de_estudios/   # PDFs de las 5 asignaturas + temario oficial
│   │   ├── fines_laborales/           # PDFs y temario laboral
│   │   └── general/                   # Información institucional
│   └── workflows/
│       ├── ingesta-qdrant.json        # Flujo 1: ingesta y dosificación
│       └── challenge-tutor-con-ia.json # Flujo 2: agente conversacional (Telegram)
├── infra/
│   ├── docker-compose.yml             # Servicios n8n + Qdrant + Caddy
│   ├── Caddyfile                      # Configuración del reverse proxy y HTTPS
│   └── .env
├── .gitignore
├── arrancador.bat                     # Levanta Docker + n8n localmente (entorno de desarrollo)
└── README.md
```

---

## 🚀 Instalación y despliegue

### Requisitos previos

- Docker y Docker Compose instalados (localmente o en tu VM de nube).
- Una API Key de [Google AI Studio](https://aistudio.google.com/) para Gemini.
- Un bot de Telegram creado con [@BotFather](https://t.me/BotFather) y su token de API.
- (Para producción) un dominio o subdominio propio apuntando a la IP de tu servidor, para que Caddy pueda emitir el certificado SSL automáticamente.

### Pasos

1. **Levanta los contenedores de n8n, Qdrant y Caddy:**
   ```bash
   cd infra
   docker compose up -d
   ```
2. **Crea las tres colecciones en Qdrant** (`fines_laborales`, `continuidad_estudios`, `general`) — se crean automáticamente la primera vez que corres el flujo de ingesta en modo _insert_.
3. **Importa los flujos JSON** en n8n desde `backend-n8n/workflows/`:
   - `ingesta-qdrant.json` → flujo de carga de documentos.
   - `challenge-tutor-con-ia.json` → flujo del agente conversacional con Telegram.
4. **Configura tus credenciales** en n8n:
   - Google Gemini (PaLM API) en los nodos de LLM y Embeddings.
   - Qdrant API, apuntando a `http://qdrant:6333` (nombre del servicio dentro de la red de Docker).
   - Telegram API, con el token entregado por @BotFather.
5. **Coloca tus PDFs oficiales** en `backend-n8n/documentos/`, respetando las tres carpetas por categoría.
6. **Ejecuta el flujo de ingesta** manualmente para poblar las colecciones vectoriales.
7. **Activa el flujo del agente** — el webhook de Telegram queda expuesto automáticamente a través de Caddy sobre HTTPS, sin necesidad de túneles como ngrok.

En producción, esta infraestructura corre de forma persistente en una instancia de **Google Cloud Platform (Compute Engine)**, con Caddy gestionando el certificado SSL y el subdominio público del servidor.

---

## 📸 Demo y capturas

| Captura | Descripción |
| --- | --- |
| `docs/telegram-saludo.png` | Saludo inicial del bot y selección de modalidad (Fines Laborales / Continuidad de Estudios). |
| `docs/telegram-pregunta.png` | El bot presentando una pregunta de práctica personalizada con formato HTML. |
| `docs/telegram-respuesta.png` | El alumno respondiendo y el bot calificando con la pauta oficial. |
| `docs/n8n-workflow-agent.png` | Flujo completo en n8n: el agente "El Señor IA", el memory buffer, el modelo Gemini y las 3 herramientas de Qdrant conectadas. |
| `docs/qdrant-collections.png` | Panel de Qdrant mostrando las colecciones `fines_laborales`, `continuidad_estudios` y `general`. |
| `docs/gcp-caddy-deploy.png` | Consola de Google Cloud / terminal SSH ejecutando `docker ps`, con los contenedores de Caddy, n8n y Qdrant activos bajo HTTPS. |

---

## 📈 Futuras versiones y comunidad

Este proyecto es una herramienta en constante evolución. Próximas actualizaciones incluyen:

- Integración de nuevos canales (WhatsApp, WebChat) para ampliar el acceso.
- Módulos de analítica para docentes o tutores de acompañamiento.
- Adaptación de temarios oficiales para otros países de Latinoamérica.

Si tienes ideas para mejorar la nivelación escolar de más personas adultas, tus _Pull Requests_ son más que bienvenidos.

---

## 👤 Autor

Proyecto desarrollado por **Andrés Palma** para el Challenge de **Alura y Oracle**.
