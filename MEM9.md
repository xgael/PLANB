# MEM9 - Memoria Persistente para Agentes IA

> **Repositorio:** [github.com/mem9-ai/mem9](https://github.com/mem9-ai/mem9)
> **Licencia:** Apache 2.0
> **Lenguajes:** Go (48.8%), TypeScript (21.1%), Python (14.1%), Shell (7.8%), Astro (6.1%)
> **Estado:** 229 estrellas, 31 forks, 209 commits

---

## 1. Que es MEM9

MEM9 es un sistema de memoria persistente y compartida en la nube para agentes de IA. Su lema es:

> *"Your agents forget everything between sessions. mem9 fixes that."*

Los agentes de IA (Claude Code, OpenCode, OpenClaw) pierden todo su contexto al terminar una sesion. MEM9 resuelve esto proporcionando una capa de memoria centralizada respaldada por **TiDB/MySQL** con busqueda hibrida (vectorial + keyword).

---

## 2. Problema que Resuelve

| Problema | Sin MEM9 | Con MEM9 |
|----------|----------|----------|
| **Amnesia** | El agente olvida todo al cerrar sesion | Memoria persiste entre sesiones |
| **Aislamiento** | Cada agente tiene su propia memoria local | Todos los agentes comparten un pool de memoria |
| **Dependencia de dispositivo** | Memoria atada a una sola maquina | Accesible desde cualquier maquina |
| **Sin colaboracion de equipo** | Los agentes de diferentes usuarios no comparten | Multiples usuarios/agentes comparten memorias |

---

## 3. Arquitectura

### Principio Fundamental

Los plugins de los agentes mantienen **cero estado**. Toda la memoria vive en el servidor (`mnemo-server`), respaldado por TiDB/MySQL.

```
+------------------+     +------------------+     +------------------+
|  Claude Code     |     |  OpenCode        |     |  OpenClaw        |
|  (Plugin)        |     |  (Plugin)        |     |  (Plugin)        |
+--------+---------+     +--------+---------+     +--------+---------+
         |                        |                        |
         +------------------------+------------------------+
                                  |
                          +-------v--------+
                          |  mnemo-server  |
                          |  (Go, HTTP)    |
                          +-------+--------+
                                  |
                          +-------v--------+
                          |  TiDB Starter  |
                          |  (DB + Vector) |
                          +----------------+
```

### Flujo de Datos

1. **Session Start** --> Carga memorias recientes en el contexto del agente
2. **User Prompt** --> Inyecta pista sobre herramientas de memoria disponibles
3. **Session Stop** --> Captura ultima respuesta y la guarda como memoria

---

## 4. Por que TiDB Starter

MEM9 utiliza **TiDB Starter** (antes TiDB Serverless) como base de datos por:

| Caracteristica | Detalle |
|----------------|---------|
| **Tier gratuito** | 25 GiB storage, 250M Request Units/mes |
| **Zero provisioning** | Creacion instantanea via API |
| **Tipo VECTOR nativo** | Elimina necesidad de BD vectorial separada |
| **Embeddings server-side** | Funcion `EMBED_TEXT` (no necesita clave OpenAI) |
| **Zero operaciones** | Escalado automatico y backups |
| **Compatible MySQL** | Migracion posible a TiDB self-hosted o MySQL |

---

## 5. Componentes del Proyecto

### 5.1 mnemo-server (Go)

El servidor central HTTP escrito en Go:

```
server/
├── cmd/mnemo-server/    # Punto de entrada
├── internal/
│   ├── config/          # Carga de variables de entorno
│   ├── domain/          # Tipos core, errores, generacion de tokens
│   ├── embedding/       # Proveedores: OpenAI, Ollama, custom
│   ├── handler/         # Endpoints HTTP (chi router)
│   ├── middleware/      # Autenticacion y rate limiting
│   ├── repository/      # Implementacion SQL para TiDB
│   └── service/         # Logica de negocio: upsert, LWW, busqueda hibrida
├── schema.sql           # Esquema principal
├── schema_db9.sql       # Esquema alternativo
├── schema_pg.sql        # Esquema PostgreSQL
├── Dockerfile           # Configuracion de contenedor
├── go.mod
└── go.sum
```

**Caracteristicas del servidor:**
- **Busqueda hibrida:** Combina vectores (semantica) + keywords (texto)
- **Upsert:** Creacion o actualizacion inteligente de memorias
- **LWW (Last-Write-Wins):** Resolucion de conflictos
- **Rate limiting** configurable por IP
- **Multi-tenancy** con aislamiento por tenant

### 5.2 Plugins de Agentes (TypeScript)

Todos los plugins exponen las mismas **5 herramientas unificadas**:

| Herramienta | Funcion |
|-------------|---------|
| `memory_store` | Crear/escribir memorias |
| `memory_search` | Busqueda hibrida (vector + keyword) |
| `memory_get` | Obtener memoria por ID |
| `memory_update` | Modificar memorias existentes |
| `memory_delete` | Eliminar memorias |

#### Plugin para Claude Code

- Usa arquitectura de **Hooks + Skills**
- Auto-carga memorias al inicio de sesion
- Auto-guarda al cerrar sesion

```
claude-plugin/
├── .claude-plugin/plugin.json   # Manifiesto
├── hooks/
│   ├── session-start.sh         # Carga memorias recientes
│   ├── stop.sh                  # Guarda ultima respuesta
│   ├── user-prompt-submit.sh    # Inyecta hint de memoria
│   └── common.sh                # Utilidades compartidas
└── skills/
    ├── mem9-recall/              # Buscar memorias bajo demanda
    └── mem9-store/               # Guardar memorias bajo demanda
```

**3 Hooks del ciclo de vida:**

| Hook | Trigger | Accion |
|------|---------|--------|
| `session-start.sh` | Inicio de sesion | Carga memorias recientes en `additionalContext` |
| `user-prompt-submit.sh` | Cada prompt | Inyecta hint sobre skills de memoria disponibles |
| `stop.sh` | Fin de sesion | Captura ultima respuesta y la guarda en BD |

**2 Skills bajo demanda:**

- **memory-store**: El usuario dice "recuerda esto" para guardar informacion explicitamente
- **memory-recall**: El usuario pregunta "que sabemos sobre X?" para buscar memorias

#### Plugin para OpenCode

- Integracion via Plugin SDK
- `system.transform` inyecta memorias
- `session.idle` auto-captura estado

#### Plugin para OpenClaw

- Tipo Memory Plugin (reemplaza el built-in `kind: "memory"`)
- El framework gestiona el ciclo de vida

---

## 6. API REST

### Autenticacion

- **v1alpha1 (Legacy):** Tenant ID en la ruta URL
- **v1alpha2 (Preferida):** Header `X-API-Key`
- Identificacion de agente via header `X-Mnemo-Agent-Id`

### Endpoints v1alpha1 (Legacy)

| Metodo | Ruta | Descripcion |
|--------|------|-------------|
| `POST` | `/v1alpha1/mem9s` | Provisionar tenant |
| `POST` | `/v1alpha1/mem9s/{tenantID}/memories` | Crear memoria |
| `GET` | `/v1alpha1/mem9s/{tenantID}/memories` | Buscar memorias |
| `GET` | `/v1alpha1/mem9s/{tenantID}/memories/:id` | Obtener por ID |
| `PUT` | `/v1alpha1/mem9s/{tenantID}/memories/:id` | Actualizar (con `If-Match` opcional) |
| `DELETE` | `/v1alpha1/mem9s/{tenantID}/memories/:id` | Eliminar |

### Endpoints v1alpha2 (Preferida)

| Metodo | Ruta | Descripcion |
|--------|------|-------------|
| `POST` | `/v1alpha2/mem9s/memories` | Crear memoria |
| `GET` | `/v1alpha2/mem9s/memories` | Buscar memorias |
| `GET` | `/v1alpha2/mem9s/memories/:id` | Obtener por ID |
| `PUT` | `/v1alpha2/mem9s/memories/:id` | Actualizar |
| `DELETE` | `/v1alpha2/mem9s/memories/:id` | Eliminar |

---

## 7. Configuracion

### Variables de Entorno del Servidor

| Variable | Requerida | Default | Proposito |
|----------|-----------|---------|-----------|
| `MNEMO_DSN` | Si | -- | Cadena de conexion a la BD |
| `MNEMO_PORT` | No | 8080 | Puerto HTTP |
| `MNEMO_RATE_LIMIT` | No | 100 | Requests/seg por IP |
| `MNEMO_RATE_BURST` | No | 200 | Allowance de burst |
| `MNEMO_EMBED_API_KEY` | No | -- | Credenciales del proveedor de embeddings |
| `MNEMO_EMBED_BASE_URL` | No | OpenAI | Endpoint de embeddings custom |
| `MNEMO_EMBED_MODEL` | No | text-embedding-3-small | Modelo de embeddings |
| `MNEMO_EMBED_DIMS` | No | 1536 | Dimensiones del vector |

### Configuracion del Plugin (Claude Code)

Agregar a `~/.claude/settings.json`:

```json
{
  "env": {
    "MEM9_TENANT_ID": "your-tenant-uuid"
  }
}
```

Obtener tenant ID en `https://api.mem9.ai`.

**Principio clave:** Agentes que comparten el mismo API key acceden al mismo pool de memoria.

---

## 8. Instalacion

### Servidor

```bash
# Desde codigo fuente
cd server
MNEMO_DSN="user:pass@tcp(host:4000)/mnemos?parseTime=true" go run ./cmd/mnemo-server

# Build local
make build
cd server
MNEMO_DSN="user:pass@tcp(host:4000)/mnemos?parseTime=true" ./bin/mnemo-server

# Docker
docker build -t mnemo-server ./server
docker run -e MNEMO_DSN="..." -p 8080:8080 mnemo-server
```

### Provisionamiento de Tenant

```bash
curl -s -X POST localhost:8080/v1alpha1/mem9s
# Respuesta: {"id":"uuid-del-tenant"}
```

### Plugins por Plataforma

| Plataforma | Metodo de Instalacion |
|------------|----------------------|
| **Claude Code (CoWork)** | `cowork install mem9-ai/mem9-claude-plugin --plugin` |
| **Claude Code (Marketplace)** | `/plugin marketplace add mem9-ai/mem9` luego `/plugin install mem9@mem9` |
| **Claude Code (Manual)** | Clonar repo, hacer hooks ejecutables, copiar skills, configurar `settings.json` |
| **OpenCode** | Agregar `"plugin": ["@mem9/opencode"]` a `opencode.json` |
| **OpenClaw** | Agregar `mnemo` a plugins en `openclaw.json` |

---

## 9. Busqueda Hibrida

MEM9 combina dos estrategias de busqueda:

### Busqueda Vectorial (Semantica)
- Usa embeddings para encontrar memorias semanticamente similares
- Proveedores soportados: **OpenAI**, **Ollama**, proveedores custom
- Modelo por defecto: `text-embedding-3-small` (1536 dimensiones)
- TiDB provee embeddings server-side via `EMBED_TEXT` (sin necesidad de clave OpenAI)

### Busqueda por Keyword (Texto)
- Busqueda clasica por coincidencia de texto
- Complementa la busqueda vectorial para mayor precision

### Combinacion
La busqueda hibrida fusiona ambos resultados para maximizar relevancia, combinando comprension semantica con coincidencia exacta de terminos.

---

## 10. Estructura del Proyecto

```
mem9/
├── server/               # Aplicacion Go core (mnemo-server)
├── opencode-plugin/      # Integracion con OpenCode
├── openclaw-plugin/      # Integracion con OpenClaw
├── claude-plugin/        # Hooks + Skills para Claude Code
├── skills/               # Definiciones de skills compartidas
├── docs/                 # Documentos de diseno y benchmarks
└── e2e/                  # Tests end-to-end
```

---

## 11. Roadmap

| Fase | Estado | Contenido |
|------|--------|-----------|
| **Fase 1** | Completada | Servidor core, CRUD, auth, busqueda hibrida, upsert, framework de plugins |
| **Fase 3** | Planificada | Resolucion de conflictos asistida por LLM, auto-tagging |
| **Fase 4** | Planificada | Dashboard web, import/export masivo, CLI wizard |

> Nota: Vector Clock CRDT fue diferido y removido del roadmap.

---

## 12. MEM9 vs Mem0

| Aspecto | MEM9 | Mem0 |
|---------|------|------|
| **Enfoque** | Agentes de codigo (Claude Code, OpenCode, OpenClaw) | Capa de memoria universal para apps IA |
| **Arquitectura** | Plugins stateless + servidor centralizado | SDK/API con multiples backends |
| **Base de datos** | TiDB/MySQL con vector nativo | Multiples (Qdrant, Pinecone, etc.) |
| **Busqueda** | Hibrida (vector + keyword) | Vector + Graph Memory |
| **Instalacion** | Self-hosted o API | SaaS o self-hosted |
| **Licencia** | Apache 2.0 | Apache 2.0 |
| **Comunidad** | ~229 estrellas | ~41,000 estrellas |
| **Uso principal** | Memoria entre sesiones de coding agents | Personalizacion de apps IA generales |

---

## 13. Casos de Uso

1. **Desarrollo continuo:** Un agente recuerda decisiones arquitecturales, convenciones y patrones del proyecto entre sesiones
2. **Colaboracion multi-agente:** Claude Code y OpenCode trabajando en el mismo proyecto comparten contexto
3. **Equipos distribuidos:** Multiples desarrolladores comparten memorias de proyecto a traves de un servidor comun
4. **Depuracion recurrente:** El agente recuerda soluciones a problemas encontrados previamente
5. **Onboarding:** Nuevos agentes/sesiones arrancan con todo el conocimiento acumulado del proyecto

---

## 14. Seguridad

- **Multi-tenancy** con aislamiento por tenant ID
- **Rate limiting** configurable por IP (default: 100 req/s, burst 200)
- **Autenticacion** via API key en headers
- **Optimistic locking** con `If-Match` para actualizaciones concurrentes

---

## 15. Fuentes

- [Repositorio oficial mem9-ai/mem9](https://github.com/mem9-ai/mem9)
- [Mem0 - The Memory Layer for AI Apps](https://mem0.ai/)
- [Mem0 raises $24M (TechCrunch)](https://techcrunch.com/2025/10/28/mem0-raises-24m-from-yc-peak-xv-and-basis-set-to-build-the-memory-layer-for-ai-apps/)
- [Top 10 AI Memory Products 2026 (Medium)](https://medium.com/@bumurzaqov2/top-10-ai-memory-products-2026-09d7900b5ab1)
- [AI Memory Systems 2026 Explained (AITechBoss)](https://www.aitechboss.com/ai-memory-systems-2026/)
- [mem9 + TiDB para OpenClaw (atbug.com)](https://atbug.com/openclaw-private-memory-mem9-tidb-self-hosted/)