# 🎓 Tutor Académico Inteligente (RAG) — Nivelación de Estudios

> **Proyecto desarrollado para el Challenge de Alura y Oracle.**
> Un asistente de Inteligencia Artificial móvil, flexible y disponible 24/7, diseñado con el objetivo que el estudiante pueda culminar con éxito la Enseñanza Media (Secundaria / Bachillerato) en Chile y Latinoamérica (proximamente).

---

## 📑 Tabla de contenidos

- [El impacto social y la problemática](#-el-impacto-social-y-la-problemática)
- [Nuestra solución](#-nuestra-solución)
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

## ⚡ El corazón del proyecto: personalización y contexto real

Para que la enseñanza sea efectiva y no frustrante, el tutor se rige bajo una regla fundamental:

> 💡 **"Aprender para trabajar... no es lo mismo que aprender para seguir estudiando."**

El agente identifica y separa estas dos realidades evaluativas con precisión:

### 💼 1. Modalidad: Fines Laborales

- **Enfoque:** preparación concentrada y práctica.
- **Contenido:** basado directamente en las normativas y exámenes oficiales en PDF del Ministerio de Educación (MINEDUC).
- **Dinámica:** la IA realiza un perfilamiento empático, preguntándole al alumno por sus pasatiempos, su trabajo o sus actividades diarias. Cada ejercicio se redacta de forma personalizada usando esos intereses — por ejemplo, si le gusta el fútbol, los problemas de matemáticas se plantean con estadísticas de su equipo favorito.

### 🎓 2. Modalidad: Continuidad de Estudios

- **Enfoque:** dominio académico profundo.
- **Contenido:** basado en mallas curriculares extensas para la preparación de exámenes de validación de estudios y educación superior.
- **Asignaturas:** Matemáticas, Lenguaje y Comunicación, Inglés, Ciencias Sociales y Ciencias Naturales.

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
│  (Gemini 2.5 Flash,           │
│   Tools Agent + memoria)     │
└──────────────▲──────────────┘
               │
          [ Estudiante ]
```

### 1. Flujo de Ingesta y Dosificación (backend de datos)

- **Problema:** enviar documentos masivos a la API de embeddings en paralelo satura el nivel gratuito de Gemini y provoca caídas por límite de uso (`Error 429 — Too Many Requests`).
- **Solución:** el flujo lee los PDF de cada carpeta (`fines_laborales`, `continuidad_de_estudios`, `general`), extrae su texto de forma nativa, lo fragmenta con un Text Splitter y lo procesa mediante **dosificadores secuenciales (`Loop Over Items`) con tiempos de espera (`Wait`)** entre cada documento, evitando saturar la API.
- **Persistencia:** los fragmentos se vectorizan con **Gemini Embeddings** y se almacenan de forma permanente en **Qdrant**, separados en tres colecciones independientes: `fines_laborales`, `continuidad_estudios` y `general`.

### 2. Flujo del Agente de Consulta (experiencia de usuario)

- **Procesamiento:** utiliza el modelo **Gemini 2.5 Flash** en modo _Tools Agent_, con memoria de conversación (`Simple Memory`) para mantener el contexto e hilo de la conversación con el alumno.
- **Ruteo semántico de colecciones:** en lugar de buscar en un único bloque masivo de datos, el agente evalúa la intención de la consulta y activa específicamente la herramienta de Qdrant asociada a la colección correcta (`fines_laborales`, `continuidad_estudios` o `general`), maximizando la precisión de la respuesta y evitando mezclar temarios.
- **Personalización dirigida por prompt:** el agente tiene instrucciones explícitas para preguntar por el contexto personal del alumno antes de generar un ejercicio, y para calificar sus respuestas usando la pauta de corrección incluida en el propio documento recuperado.

---

## 🛠️ Stack tecnológico

La arquitectura fue pensada bajo principios de robustez y rentabilidad, sosteniendo el servicio de forma completamente gratuita y lista para producción:

| Tecnología                                                  | Rol                                                                                               |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| ⚙️ **n8n**                                                  | Orquestador visual de los flujos de trabajo y la lógica del agente                                |
| 🧠 **Google Gemini 2.5 Flash**                              | Modelo de lenguaje (LLM) principal, de alta velocidad y amplia ventana de contexto                |
| 🏷️ **Gemini Embeddings**                                    | Modelo que codifica consultas y fragmentos de texto en vectores                                   |
| 🗄️ **Qdrant**                                               | Base de datos vectorial persistente para el almacenamiento y la segmentación semántica            |
| 🐳 **Docker / Docker Compose**                              | Contenerización de n8n y Qdrant para un despliegue reproducible                                   |
| 🌐 **ngrok**                                                | Túnel público para pruebas locales antes del despliegue en la nube                                |
| ☁️ **Oracle Cloud Infrastructure (OCI) — Always Free Tier** | Máquina virtual ARM (Ampere) donde n8n y Qdrant corren de forma persistente y 100% gratuita, 24/7 |

---

## 📂 Estructura del repositorio

```text
CHALLENGE/
├── backend-n8n/
│   ├── documentos/
│   │   ├── continuidad_de_estudios/   # PDFs de las 5 asignaturas
│   │   ├── fines_laborales/           # PDFs y temario laboral
│   │   └── general/                   # Información institucional
│   └── workflows/
│       ├── ingesta-qdrant.json        # Flujo 1: ingesta y dosificación
│       └── challenge-tutor-con-ia.json # Flujo 2: agente conversacional
├── infra/
│   ├── docker-compose.yml             # Servicios n8n + Qdrant
│   └── .env
├── .gitignore
├── arrancador.bat                     # Levanta Docker + n8n + túnel ngrok
└── README.md
```

---

## 🚀 Instalación y despliegue

### Requisitos previos

- Docker y Docker Compose instalados.
- Una API Key de [Google AI Studio](https://aistudio.google.com/) para Gemini.
- (Opcional, para pruebas locales) una cuenta de [ngrok](https://ngrok.com/) con un dominio estático.

### Pasos

1. **Levanta los contenedores de n8n y Qdrant:**
   ```bash
   cd infra
   docker compose up -d
   ```
2. **Crea las tres colecciones en Qdrant** (`fines_laborales`, `continuidad_estudios`, `general`) — se crean automáticamente la primera vez que corres el flujo de ingesta en modo _insert_.
3. **Importa los flujos JSON** en n8n desde `backend-n8n/workflows/`:
   - `ingesta-qdrant.json` → flujo de carga de documentos.
   - `challenge-tutor-con-ia.json` → flujo del agente conversacional.
4. **Configura tus credenciales** en n8n:
   - Google Gemini (PaLM API) en los nodos de LLM y Embeddings.
   - Qdrant API, apuntando a `http://qdrant:6333` (nombre del servicio dentro de la red de Docker).
5. **Coloca tus PDFs oficiales** en `backend-n8n/documentos/`, respetando las tres carpetas por categoría.
6. **Ejecuta el flujo de ingesta** manualmente para poblar las colecciones vectoriales.
7. **Activa el flujo del agente** y pruébalo desde el panel de Chat de n8n, o expón el webhook con ngrok / OCI para acceso externo.

Para producción, el mismo `docker-compose.yml` puede desplegarse sin cambios en una instancia **OCI Always Free Tier (ARM Ampere)**, garantizando disponibilidad 24/7 sin costo.

---

## 📸 Demo y capturas

| Captura                    | Descripción                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------- |
| `docs/demo-chat.png`       | El agente realizando el perfilamiento empático antes de formular un ejercicio personalizado.    |
| `docs/flujo-ingesta.png`   | Flujo 1: lectura de PDFs, extracción, loops de dosificación e indexación vectorial en Qdrant.   |
| `docs/n8n-flow.png`        | Flujo 2: el canvas conversacional con memoria, modelo Gemini y las tres herramientas de Qdrant. |
| `docs/oci-instance.png`    | Consola de Oracle Cloud mostrando la instancia en estado _Running_.                             |
| `docs/services-status.png` | Terminal SSH ejecutando `docker ps`, confirmando que n8n y Qdrant operan de forma persistente.  |

---

## 📈 Futuras versiones y comunidad

Este proyecto es una herramienta en constante evolución. Próximas actualizaciones incluyen:

- Integración de nuevos canales (WhatsApp, Telegram) para facilitar el acceso móvil.
- Módulos de analítica para docentes o tutores de acompañamiento.
- Adaptación de temarios oficiales para otros países de Latinoamérica.

Si tienes ideas para mejorar la nivelación escolar de más personas adultas, tus _Pull Requests_ son más que bienvenidos.

---

## 👤 Autor

Proyecto desarrollado por **Andrés Palma** para el Challenge de **Alura y Oracle**.
