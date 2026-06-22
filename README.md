# Aurora — Plataforma de coaching con IA

Aurora es el asistente conversacional del programa de coaching de Paula Andrea
Yépez. Acompaña a las alumnas resolviendo dudas sobre el contenido del curso a
partir de la base de conocimiento del programa, tanto desde un portal privado
con historial de conversaciones como desde un widget embebible.

## Características

- **Chat con recuperación de contexto (RAG)** sobre el material del curso.
- **Portal privado** con autenticación, conversaciones e historial por alumna.
- **Widget embebible** para integrarse en plataformas externas.
- **Respuestas en streaming** (SSE) para reducir el tiempo hasta el primer token.
- **Seguridad de producción**: aislamiento de datos por usuario (RLS),
  cabeceras de seguridad (Helmet/CSP/HSTS), CORS con lista blanca, rate limiting,
  validación de entrada (Zod) y sanitización de salida (DOMPurify).

## Stack

- **Backend**: Node.js 20 + TypeScript + Express 5
- **Base de datos y autenticación**: Supabase (PostgreSQL + Auth)
- **Modelo y recuperación**: OpenAI Responses API con `file_search`
- **Frontend**: portal y widget estáticos (HTML/CSS/JS)

## Estructura

```
src/            Código del servidor (rutas, servicios, configuración)
public/portal/  Portal privado de las alumnas
public/widget/  Widget embebible
scripts/        Utilidades de indexación de la base de conocimiento
```

## Requisitos

- Node.js 20+
- Una cuenta de OpenAI con acceso a la Responses API
- Un proyecto de Supabase

## Configuración

1. Instalar dependencias:

   ```bash
   npm ci
   ```

2. Crear el archivo de entorno a partir de la plantilla y completar los valores:

   ```bash
   cp .env.example .env
   ```

   Variables principales:

   | Variable | Descripción |
   |---|---|
   | `OPENAI_API_KEY` | Clave de la API de OpenAI |
   | `OPENAI_VECTOR_STORE_ID` | ID del vector store con la base de conocimiento |
   | `OPENAI_MODEL` | Modelo a usar (por defecto `gpt-4.1-mini`) |
   | `SUPABASE_URL` | URL del proyecto de Supabase |
   | `SUPABASE_ANON_KEY` | Clave pública (anon) de Supabase |
   | `SUPABASE_SERVICE_ROLE_KEY` | Clave de servicio de Supabase (solo backend) |
   | `ALLOWED_ORIGINS` | Orígenes permitidos para CORS y embebido del widget |

## Desarrollo

```bash
npm run dev
```

El servidor queda disponible en `http://localhost:8787`:

- Portal: `http://localhost:8787/portal/`
- Widget: `http://localhost:8787/widget/`

## Producción

```bash
npm run build
npm start
```

El comando `build` compila TypeScript a `dist/` y `start` ejecuta el servidor
compilado. El servicio no arranca si falta alguna variable de entorno
obligatoria.

## Seguridad

### Controles implementados

| Área | Mecanismo |
|---|---|
| Aislamiento de datos | Row-Level Security (RLS) en Supabase; cada consulta filtra por `user_id` |
| Headers HTTP | Helmet con CSP estricta, HSTS y `X-Frame-Options` |
| CORS | Lista blanca explícita vía `ALLOWED_ORIGINS`; el widget valida el origen del iframe |
| Rate limiting | `express-rate-limit` en rutas de chat y autenticación |
| Validación de entrada | Zod en todos los endpoints; límites de longitud en mensajes |
| Sanitización de salida | DOMPurify antes de cualquier `innerHTML` en el cliente |
| Errores | Los errores internos solo se registran en servidor; el cliente recibe mensajes genéricos |
| Caché | `Cache-Control: no-store` en rutas del portal privado |
| Variables de entorno | `required()` en arranque: el servidor falla rápido si falta alguna clave |

### Requisitos antes de desplegar

1. **`ALLOWED_ORIGINS`** — listar únicamente los dominios reales (Kajabi, dominio propio). No usar `*`.
2. **`NODE_ENV=production`** — activa el modo estricto de errores y desactiva logs de depuración.
3. **Claves de Supabase** — `SUPABASE_SERVICE_ROLE_KEY` solo se usa en el backend; nunca exponerla al cliente.
4. **RLS activo** — verificar que las políticas RLS estén habilitadas en las tablas `conversations` y `messages` antes de ir a producción.
5. **HTTPS** — el proveedor de hosting debe forzar HTTPS; la cabecera HSTS solo tiene efecto sobre conexiones seguras.
