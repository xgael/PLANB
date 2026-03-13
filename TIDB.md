# TiDB & TiDB X - Documentacion Completa

> **TiDB** es una base de datos SQL distribuida, open-source, cloud-native, disenada para cargas de trabajo HTAP (Hybrid Transactional and Analytical Processing). Creada por **PingCAP**, compatible con MySQL, y con escalabilidad horizontal masiva.
>
> **TiDB X** es el motor cloud-native de nueva generacion que impulsa **TiDB Cloud**, con arquitectura basada en object storage, Virtual Clusters y elasticidad serverless real.

---

## Tabla de Contenidos

1. [Que es TiDB](#1-que-es-tidb)
2. [Arquitectura de TiDB](#2-arquitectura-de-tidb)
3. [Componentes Principales](#3-componentes-principales)
4. [Transacciones Distribuidas](#4-transacciones-distribuidas)
5. [HTAP - Procesamiento Hibrido](#5-htap---procesamiento-hibrido)
6. [Compatibilidad con MySQL](#6-compatibilidad-con-mysql)
7. [Seguridad](#7-seguridad)
8. [Ecosistema de Herramientas](#8-ecosistema-de-herramientas)
9. [TiDB Cloud](#9-tidb-cloud)
10. [TiDB X - El Motor de Nueva Generacion](#10-tidb-x---el-motor-de-nueva-generacion)
11. [TiDB en Alibaba Cloud](#11-tidb-en-alibaba-cloud)
12. [Busqueda Vectorial e IA](#12-busqueda-vectorial-e-ia)
13. [Monitorizacion y Observabilidad](#13-monitorizacion-y-observabilidad)
14. [Historial de Versiones](#14-historial-de-versiones)
15. [Casos de Uso en Produccion](#15-casos-de-uso-en-produccion)
16. [Comparativa con Competidores](#16-comparativa-con-competidores)
17. [Limitaciones Conocidas](#17-limitaciones-conocidas)
18. [Comparativa con PolarDB-X de Alibaba](#18-comparativa-con-polardb-x-de-alibaba)
19. [Ecosistema de BDs Distribuidas en China](#19-ecosistema-de-bds-distribuidas-en-china)

---

## 1. Que es TiDB

**TiDB** (Titanium Database) es una base de datos SQL distribuida de codigo abierto creada por **PingCAP** en 2015. Esta licenciada bajo **Apache 2.0** y disenada inspirandose en la arquitectura de **Google Spanner** y **Google F1**.

### Caracteristicas Fundamentales

| Caracteristica | Descripcion |
|---|---|
| **Tipo** | NewSQL / Distributed SQL Database |
| **Modelo** | HTAP (Transaccional + Analitico) |
| **Compatibilidad** | MySQL 8.0 |
| **Consistencia** | ACID completo con transacciones distribuidas |
| **Escalabilidad** | Horizontal, hasta nivel de Petabytes |
| **Alta Disponibilidad** | Multi-replica con Raft consensus |
| **Licencia** | Apache 2.0 (Open Source) |
| **Lenguaje** | Go (TiDB Server), Rust (TiKV), C++ (TiFlash) |
| **Fundadores** | Max Liu, Ed Huang, Dylan Cui (Beijing, 2015) |
| **Financiacion** | +$500M USD (Serie D: $270M en 2020) |
| **GitHub Stars** | +37,000 |
| **CNCF** | TiKV graduado de CNCF (Sept 2020) |
| **Usuarios** | +3,000 empresas globales |

### Que Resuelve TiDB

- **Escalabilidad MySQL**: MySQL no escala horizontalmente de forma nativa; TiDB si
- **Sharding manual eliminado**: Auto-sharding transparente sin modificar la aplicacion
- **OLTP + OLAP unificados**: Elimina la necesidad de ETL y data warehouses separados
- **Alta disponibilidad nativa**: Sin necesidad de configurar replicacion master-slave
- **Consistencia fuerte**: Transacciones ACID distribuidas sin compromisos

---

## 2. Arquitectura de TiDB

TiDB usa una arquitectura de **separacion completa de computo y almacenamiento** con tres capas principales:

```
                    +---------------------------+
                    |      Aplicaciones          |
                    |   (MySQL Protocol 8.0)     |
                    +------------+--------------+
                                 |
                    +------------v--------------+
                    |       TiDB Server          |
                    |   (Capa SQL Stateless)      |
                    |  - Parser SQL               |
                    |  - Optimizador (CBO)        |
                    |  - Ejecutor distribuido      |
                    +---+-------------------+----+
                        |                   |
           +------------v----+    +---------v-----------+
           |   TiKV Server   |    |     TiFlash          |
           | (Row Storage)   |    |  (Columnar Storage)  |
           | - Key-Value     |    |  - Analitica OLAP    |
           | - Raft Consensus|    |  - MPP Engine         |
           | - MVCC          |    |  - Raft Learner       |
           | - RocksDB       |    |  - DeltaTree          |
           +--------+--------+    +----------+-----------+
                    |                        |
           +--------v------------------------v-----------+
           |          PD (Placement Driver)               |
           |  - Metadata del cluster                      |
           |  - Timestamp Oracle (TSO)                    |
           |  - Scheduling y balanceo                     |
           |  - TiDB Dashboard                            |
           +---------------------------------------------+
```

### Flujo de una Consulta SQL

1. El cliente se conecta via **protocolo MySQL** al TiDB Server
2. TiDB Server **parsea** el SQL y genera un plan de ejecucion optimizado (CBO)
3. El optimizador decide si usar **TiKV** (row-based, OLTP) o **TiFlash** (columnar, OLAP)
4. Las solicitudes se distribuyen a los nodos TiKV/TiFlash correspondientes
5. **PD** proporciona timestamps globales (TSO) y metadata de ubicacion de datos
6. Los resultados se agregan en TiDB Server y se devuelven al cliente

---

## 3. Componentes Principales

### 3.1 TiDB Server (Capa SQL)

- **Stateless**: No almacena datos, puede escalar anadiendo mas nodos
- **Compatible MySQL 8.0**: Protocolo wire-compatible
- **Optimizador CBO**: Cost-Based Optimizer con estadisticas
- **Max nodos**: Hasta 512 nodos de computo
- **Max concurrencia**: 1,000 conexiones concurrentes por nodo
- **Funciones**: Parser SQL, optimizacion de queries, generacion de planes distribuidos, ejecucion de DDL en linea

### 3.2 TiKV (Motor de Almacenamiento Distribuido)

TiKV es el motor de almacenamiento distribuido key-value que forma el corazon de TiDB.

**Arquitectura interna:**

```
+--------------------------------------------------+
|                   TiKV Node                       |
|  +--------------------------------------------+  |
|  |              Raft Store                     |  |
|  |  +--------+  +--------+  +--------+        |  |
|  |  |Region 1|  |Region 2|  |Region N|  ...   |  |
|  |  | Leader |  |Follower|  | Leader |         |  |
|  |  +--------+  +--------+  +--------+        |  |
|  +--------------------------------------------+  |
|  |            Transaction Layer                |  |
|  |  - MVCC (Multi-Version Concurrency Control) |  |
|  |  - Percolator 2PC                           |  |
|  +--------------------------------------------+  |
|  |              RocksDB (LSM-Tree)             |  |
|  |  - Write Column (commit info)               |  |
|  |  - Data Column (real data)                  |  |
|  |  - Lock Column (lock info)                  |  |
|  +--------------------------------------------+  |
+--------------------------------------------------+
```

**Conceptos clave:**

| Concepto | Descripcion |
|---|---|
| **Region** | Unidad minima de datos (~96 MB por defecto). Cada Region contiene un rango continuo de claves |
| **Replicas** | 3 replicas por defecto de cada Region, distribuidas en nodos diferentes |
| **Raft** | Protocolo de consenso para replicacion. Cada Region forma un Raft Group |
| **Multi-Raft** | Multiples Raft Groups operando en paralelo para maxima concurrencia |
| **Leader** | Cada Region tiene un lider que maneja lecturas/escrituras |
| **Follower** | Replicas que reciben logs Raft del lider |
| **Coprocessor** | Empuja computo hacia el nodo de almacenamiento (pushdown) |
| **Region Split/Merge** | Regions se dividen automaticamente cuando crecen o se fusionan cuando se reducen |

**Almacenamiento en RocksDB:**

TiKV almacena datos en RocksDB (motor LSM-Tree de Facebook). Los datos se organizan en tres Column Families:

- **Write CF**: Informacion de commits (timestamps de commit)
- **Default CF**: Datos reales (valores)
- **Lock CF**: Informacion de bloqueos para transacciones en curso

Las claves MVCC se codifican como: `key_userkey + version_timestamp`

**Encoding de Tablas a Key-Value:**
```
Datos de fila:    t{TableID}_r{RowID}                          -> {row_data}
Indice unico:     t{TableID}_i{IndexID}_{IndexValues}          -> {RowID}
Indice no-unico:  t{TableID}_i{IndexID}_{IndexValues}_{RowID}  -> null
```
Este encoding garantiza que filas de la misma tabla se almacenan contiguas y ordenadas por PK.

### 3.3 PD (Placement Driver)

El "cerebro" del cluster TiDB:

- **Metadata Store**: Almacena topologia del cluster y ubicacion de cada Region
- **Timestamp Oracle (TSO)**: Genera timestamps globales monotonicos crecientes para MVCC
- **Scheduler**: Balancea regions entre nodos TiKV, maneja hot spots
- **Leader Election**: Selecciona lideres de Region basandose en carga
- **TiDB Dashboard**: Interfaz web de gestion integrada
- **Diagnostico**: Informacion de cluster, slow queries, hot regions

**Estrategias de scheduling:**
- Balance de lideres entre nodos
- Balance de regiones entre nodos
- Deteccion y mitigacion de hot spots
- Reemplazo de replicas en nodos caidos

**Garantias RPO/RTO:**

| Escenario | RPO | RTO |
|---|---|---|
| Fallo de un nodo (3 replicas) | 0 (sin perdida) | ~30s (eleccion de lider) |
| Fallo de un DC (3 DCs, 3 replicas) | 0 | Segundos a minutos |
| 2-DC + 1 witness, fallo DC primario | 0 (modo sync) | Minutos |
| Replicacion TiCDC cross-cluster | Segundos (lag async) | Minutos a horas |
| BR full backup + PITR restore | Depende de intervalo backup | Horas (segun volumen) |

### 3.4 TiFlash (Motor Columnar Analitico)

TiFlash es la extension columnar que habilita HTAP real:

```
TiKV (Row Store) ---Raft Learner Replication---> TiFlash (Column Store)
     OLTP                                              OLAP
  (Transacciones)                                   (Analitica)
```

**Caracteristicas:**

| Feature | Detalle |
|---|---|
| **Replicacion** | Asincrona via Raft Learner (no afecta rendimiento OLTP) |
| **Consistencia** | Fuerte - misma garantia que TiKV usando snapshot reads |
| **Formato** | DeltaTree (optimizado para columnar con actualizaciones frecuentes) |
| **Motor MPP** | Massively Parallel Processing para queries analiticas |
| **Aislamiento** | Completo - cargas OLAP no impactan OLTP |
| **Automatico** | El optimizador decide automaticamente si usar TiKV o TiFlash |

**Caso de uso tipico:**
```sql
-- Crear replica columnar de una tabla
ALTER TABLE orders SET TIFLASH REPLICA 1;

-- El optimizador rutea automaticamente queries analiticas a TiFlash
SELECT region, SUM(amount), COUNT(*)
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY region;
-- ^ Esta query se ejecuta automaticamente en TiFlash via MPP
```

### 3.5 TiSpark

Conector que permite ejecutar queries Apache Spark directamente sobre TiKV/TiFlash, habilitando analytics complejos con el ecosistema Hadoop/Spark sin mover datos.

---

## 4. Transacciones Distribuidas

TiDB implementa transacciones ACID completas sobre un sistema distribuido usando el modelo **Percolator** (inspirado en Google Percolator).

### 4.1 Modelo Percolator - Two-Phase Commit (2PC)

```
Fase 1: PREWRITE
+----------+     +----------+     +----------+
| TiDB     |---->| TiKV-1   |     | TiKV-2   |
| (Coord.) |     | Primary  |     | Secondary|
|          |---->| Lock     |---->| Lock     |
+----------+     +----------+     +----------+

Fase 2: COMMIT
+----------+     +----------+     +----------+
| TiDB     |---->| TiKV-1   |     | TiKV-2   |
| (Coord.) |     | Write    |     | Write    |
|          |     | +Unlock  |     | +Unlock  |
+----------+     +----------+     +----------+
```

**Proceso detallado:**

1. **Begin**: TiDB solicita un `start_ts` (timestamp de inicio) al TSO de PD
2. **Prewrite**:
   - Se elige una clave como **primary lock**
   - Se escriben locks en todas las claves involucradas
   - Datos se escriben en la Data Column con el `start_ts`
   - Lock Column registra informacion de bloqueo (apuntando al primary)
3. **Commit**:
   - Se solicita un `commit_ts` al TSO
   - Se escribe el commit del primary lock en la Write Column
   - Se eliminan los locks secundarios asincrónicamente
4. **Cleanup**: Locks huerfanos se resuelven automaticamente

### 4.2 Modos de Transaccion

| Modo | Descripcion | Caso de Uso |
|---|---|---|
| **Optimista** | Detecta conflictos al commit (Percolator clasico) | Baja contención, alto throughput |
| **Pesimista** | Adquiere locks en cada DML (compatible MySQL) | Alta contención, compatibilidad MySQL |

**Modo pesimista** (default desde TiDB 3.0.8):
- Comportamiento identico a MySQL InnoDB
- Lock adquirido en `SELECT ... FOR UPDATE`
- Deadlock detection distribuido
- Mas compatible con aplicaciones MySQL existentes

### 4.3 Niveles de Aislamiento

| Nivel | Soportado | Notas |
|---|---|---|
| **Snapshot Isolation (SI)** | Si (default) | Equivalente a Repeatable Read de MySQL |
| **Read Committed** | Si | Desde TiDB 4.0 |
| **Serializable** | No | Usar SI como alternativa |

### 4.4 MVCC (Multi-Version Concurrency Control)

- Cada escritura crea una nueva version con timestamp unico
- Lectores nunca bloquean escritores y viceversa
- Snapshots consistentes definidos por `start_ts` del TSO
- **Garbage Collection (GC)**: Versiones antiguas se eliminan periodicamente basandose en un `safe_point` global

---

## 5. HTAP - Procesamiento Hibrido

TiDB es una de las pocas bases de datos que ofrece HTAP real en una sola plataforma:

```
                        TiDB Server
                       /           \
                      /             \
               TiKV (Row)      TiFlash (Column)
               OLTP              OLAP

  - INSERT/UPDATE/DELETE     - Aggregations
  - Point queries            - Full table scans
  - Index lookups            - GROUP BY, JOIN
  - Transaction heavy        - Analytics heavy
```

### Ventajas del HTAP

1. **Sin ETL**: Los datos se replican automaticamente de TiKV a TiFlash via Raft Learner
2. **Consistencia en tiempo real**: TiFlash siempre tiene datos consistentes (no eventual)
3. **Aislamiento de recursos**: OLAP en TiFlash no degrada OLTP en TiKV
4. **Decision automatica**: El optimizador CBO elige el motor optimo por query
5. **MPP Engine**: Procesamiento paralelo masivo para queries analiticas complejas

### Motor MPP (Massively Parallel Processing)

TiFlash incluye un motor MPP que:
- Distribuye ejecucion de queries entre multiples nodos TiFlash
- Soporta Hash Join distribuido, Broadcast Join
- Exchange operators para shuffle de datos entre nodos
- Escalabilidad lineal para queries analiticas

---

## 6. Compatibilidad con MySQL

### Lo que ES compatible

- **Protocolo**: Wire-compatible con MySQL 8.0
- **Sintaxis SQL**: Gran mayoria del SQL de MySQL soportado
- **Tipos de datos**: Todos los tipos principales (INT, VARCHAR, DATETIME, JSON, BLOB, etc.)
- **Indices**: Primary Key, Unique, Secondary, Composite, Expression indexes
- **Vistas**: CREATE VIEW soportado
- **CTEs**: Common Table Expressions (WITH)
- **Window Functions**: ROW_NUMBER, RANK, DENSE_RANK, etc.
- **JSON Functions**: Soporte completo de funciones JSON
- **Charset**: utf8mb4 por defecto
- **Prepared Statements**: Soportados con cache
- **Subqueries**: Soportadas incluyendo correlacionadas
- **Auto Increment**: Soportado (con comportamiento ligeramente diferente en distribuido)

### Lo que NO es compatible

| Feature No Soportada | Alternativa / Notas |
|---|---|
| **Stored Procedures** | Migrar logica a la capa de aplicacion |
| **Triggers** | Usar TiCDC o logica de aplicacion |
| **User-Defined Functions (UDF)** | No soportado |
| **Events (Scheduler)** | Usar cron externo o schedulers |
| **FULLTEXT Index** | Usar sistemas externos (Elasticsearch) |
| **Spatial/GIS Functions** | Soporte limitado |
| **SERIALIZABLE Isolation** | Usar Snapshot Isolation |
| **Foreign Keys** | Soportados desde v6.6 pero con limitaciones |
| **XA Transactions** | No soportado |
| **CREATE TABLE ... SELECT** | Soportado parcialmente |
| **Temporary Tables** | Soportado (Local Temporary Tables) |

### Diferencias de Comportamiento

- **Auto Increment IDs**: No son secuenciales continuos (se asignan en lotes por nodo)
- **DDL Online**: ALTER TABLE es no-bloqueante pero puede tomar mas tiempo
- **LIMIT sin ORDER BY**: Resultados pueden variar entre ejecuciones (distribuido)
- **Collation**: Soporta `utf8mb4_bin` y `utf8mb4_general_ci`, otros limitados

---

## 7. Seguridad

### 7.1 Cifrado

| Capa | Implementacion |
|---|---|
| **En transito** | TLS 1.2/1.3 entre todos los componentes (cliente-TiDB, TiDB-TiKV, TiKV-TiKV) |
| **En reposo** | Transparent Data Encryption (TDE) sobre archivos de datos y logs |
| **Claves gestionadas** | Customer-Managed Encryption Keys (CMEK) via AWS KMS (en TiDB Cloud) |

### 7.2 Autenticacion y Autorizacion

- **Autenticacion**: Compatible con metodos de autenticacion MySQL (mysql_native_password, caching_sha2_password)
- **RBAC**: Role-Based Access Control completo (heredado de MySQL 8.0)
- **Dynamic Privileges**: Privilegios granulares dinamicos
- **Password Policies**: Politicas de complejidad y expiracion de contrasenas

### 7.3 Auditoria

- **Audit Logging**: Registro detallado de operaciones SQL y accesos
- **Integracion SIEM**: Export a Splunk, Datadog y otros
- **Slow Query Log**: Registro de queries lentas con planes de ejecucion

### 7.4 Compliance

Certificaciones mantenidas:
- SOC 2 Type 2
- ISO/IEC 27001
- ISO/IEC 27701
- GDPR
- HIPAA (Business Associate)
- PCI-DSS
- CCPA

---

## 8. Ecosistema de Herramientas

### 8.1 Despliegue y Gestion

| Herramienta | Funcion |
|---|---|
| **TiUP** | Package manager y herramienta de despliegue. Instala, inicia, detiene, destruye, escala y actualiza clusters completos |
| **TiDB Operator** | Operador Kubernetes para despliegue y gestion automatizada en entornos cloud-native |
| **TiDB Dashboard** | Interfaz web integrada para monitoreo, diagnostico y gestion del cluster |

### 8.2 Migracion de Datos

| Herramienta | Funcion | Caso de Uso |
|---|---|---|
| **TiDB Data Migration (DM)** | Migracion completa + replicacion incremental desde MySQL/MariaDB | Migracion en vivo de MySQL a TiDB |
| **TiDB Lightning** | Importacion masiva ultra-rapida | Carga inicial de TB de datos |
| **Dumpling** | Exportacion de datos como SQL o CSV | Backup logico, exportacion |

**TiDB Lightning - Modos de importacion:**

| Modo | Descripcion | Velocidad | Disponibilidad durante import |
|---|---|---|---|
| **Physical Import** | Genera SST files e inyecta directamente en TiKV | ~500 GB/h | Cluster no disponible |
| **Logical Import** | INSERT via protocolo SQL | Mas lento | Cluster disponible |

### 8.3 Replicacion y Backup

| Herramienta | Funcion |
|---|---|
| **TiCDC** | Change Data Capture en tiempo real. Captura cambios de TiKV y los envia a downstream (MySQL, Kafka, S3, etc.) |
| **BR (Backup & Restore)** | Backup y restauracion fisica de cluster completo. Soporta backup incremental |
| **PITR** | Point-in-Time Recovery usando BR + logs |

### 8.4 Comandos CLI de Referencia Rapida

**TiUP - Gestion de Cluster:**
```bash
tiup cluster deploy mydb v8.5.0 topology.yaml   # Desplegar cluster
tiup cluster start mydb                          # Iniciar cluster
tiup cluster stop mydb                           # Detener cluster
tiup cluster upgrade mydb v8.5.0                 # Upgrade rolling
tiup cluster scale-out mydb scale.yaml           # Anadir nodos
tiup cluster scale-in mydb --node 10.0.0.5:20160 # Eliminar nodo
tiup cluster display mydb                        # Estado del cluster
tiup playground                                  # Cluster local de prueba
```

**TiCDC - Change Data Capture:**
```bash
# Replicar a MySQL downstream
tiup cdc cli changefeed create \
  --server=http://cdc:8300 \
  --sink-uri="mysql://user:pass@downstream:3306/" \
  --changefeed-id="mysql-repl"

# Replicar a Kafka
tiup cdc cli changefeed create \
  --server=http://cdc:8300 \
  --sink-uri="kafka://broker:9092/topic?protocol=canal-json" \
  --changefeed-id="kafka-stream"
```

**BR - Backup & Restore:**
```bash
# Backup completo a S3
tiup br backup full --pd "pd:2379" --storage "s3://bucket/backup"

# Restore desde backup
tiup br restore full --pd "pd:2379" --storage "s3://bucket/backup"

# Iniciar log backup para PITR
tiup br log start --pd "pd:2379" --storage "s3://bucket/logs"

# Restaurar a punto en el tiempo
tiup br restore point --pd "pd:2379" \
  --storage "s3://bucket/logs" \
  --full-backup-storage "s3://bucket/backup" \
  --restored-ts "2025-01-15 10:30:00"
```

**Dumpling - Exportacion:**
```bash
dumpling -u root -P 4000 -h 127.0.0.1 \
  --filetype sql -o /export \
  -t 8 -F 256MiB \
  --filter "mydb.*" \
  --consistency snapshot
```

### 8.5 Flujo Tipico de Migracion MySQL -> TiDB

```
MySQL Source
    |
    v
[Dumpling] --> SQL/CSV files --> [TiDB Lightning] --> TiDB Cluster
    |                                                       ^
    v                                                       |
[TiDB DM] -------- Full + Incremental Replication -------->|
                   (para migracion en vivo)
```

---

## 9. TiDB Cloud

**TiDB Cloud** es el servicio DBaaS (Database-as-a-Service) completamente gestionado por PingCAP.

### Planes Disponibles

| Plan | Descripcion | Precio Base | SLA |
|---|---|---|---|
| **Starter** (antes Serverless) | Auto-scaling, escala a cero, free tier generoso | $0/mes (free tier) | 99.9% |
| **Essential** | Balance de escalabilidad, rendimiento y seguridad | ~$500/mes | 99.99% |
| **Dedicated** | Infra dedicada para cargas empresariales criticas | Basado en nodos | 99.99% |

### Free Tier (Starter)

- 25 GiB de almacenamiento row-based (gratuito por org)
- 25 GiB de almacenamiento columnar
- 250M Request Units (RUs) por mes
- Auto-scaling automatico
- Escala a cero cuando esta inactivo

### Proveedores Cloud Soportados

| Proveedor | Regiones |
|---|---|
| **AWS** | Multiples regiones globales |
| **Google Cloud** | Multiples regiones globales |
| **Alibaba Cloud** | Tokyo, Singapore, Jakarta, Mexico |

### Funcionalidades Cloud Especificas

- Deploy con un clic via consola web
- Auto-scaling de computo y almacenamiento
- Monitoring y alertas integradas
- Backup automatico y PITR
- Private Link / VPC Peering
- Import/Export desde/hacia S3, GCS, Alibaba OSS
- Integracion con Datadog, Prometheus, Grafana

### Features Avanzados de TiDB Cloud

| Feature | Descripcion |
|---|---|
| **Chat2Query** | Interfaz de lenguaje natural a SQL usando IA (genera SQL desde preguntas en texto) |
| **Data Service** | Endpoints RESTful API generados automaticamente desde SQL, sin backend propio |
| **Branching** | Ramas de base de datos tipo Git para workflows de desarrollo/CI-CD (Serverless) |
| **Changefeeds gestionados** | TiCDC gestionado para streaming a Kafka, MySQL, S3 |
| **Import Service** | Importacion gestionada desde S3, GCS o archivos locales |
| **Terraform Provider** | Infrastructure-as-code para gestionar recursos TiDB Cloud |
| **ticloud CLI** | Herramienta de linea de comandos para gestionar TiDB Cloud |

---

## 10. TiDB X - El Motor de Nueva Generacion

**TiDB X** es la reinvencion arquitectonica del motor que impulsa TiDB Cloud. No es un producto independiente ni un fork, sino el **nuevo core engine** disenado desde cero para cloud-native serverless.

### 10.1 Origen

| Periodo | Hito |
|---|---|
| **2017** | Max Liu inicia proyecto interno de optimizacion de transacciones |
| **2017-2021** | Inspiracion en paper WiscKey y BadgerDB (separacion keys/values en LSM-Trees) |
| **Abril 2021** | Ed Huang presenta conceptos iniciales de TiDB X en conferencia CCF |
| **2022** | Reinicio del proyecto con objetivos ambiciosos no-incrementales |
| **2023-2024** | TiDB X se convierte en el motor de TiDB Cloud |

### 10.2 Los 5 Principios Arquitectonicos

#### 1. Filosofia Service-Oriented
TiDB X no es una base de datos empaquetada para desplegar; es un **servicio online cloud-native** cuyo producto es una base de datos SQL. Esto cambia fundamentalmente la estructura del equipo, patrones de despliegue y operaciones.

#### 2. Arquitectura SOA (Service-Oriented Architecture)
- Descomposicion en microservicios granulares
- Escalabilidad independiente de componentes
- Multi-tenancy de alta densidad
- Aislamiento de fallos entre componentes
- Evolucion independiente de cada servicio

#### 3. Virtual Clusters (VCs)
Concepto revolucionario que reemplaza instancias fisicas:

```
+--------------------------------------------------+
|              Shared Infrastructure                |
|  +--------+  +--------+  +--------+  +--------+  |
|  |  VC-1  |  |  VC-2  |  |  VC-3  |  | VC-N   |  |
|  | Tenant |  | Tenant |  | Tenant |  | Tenant |  |
|  |   A    |  |   B    |  |   C    |  |   N    |  |
|  +--------+  +--------+  +--------+  +--------+  |
|                                                    |
|  Shared Compute Pool | Shared Object Storage       |
+--------------------------------------------------+
```

- Clusters logicos que comparten computo y almacenamiento
- Aislamiento via metadata y routing (prefix-based key isolation)
- Permite servir **millones de tenants ligeros**
- Creacion y destruccion de clusters en **segundos**

#### 4. Log-as-Database
Inspirado en Amazon Aurora:
- Los **logs son la fuente de verdad**
- Todo lo demas es reconstruible (NVMe y memoria son cache de lectura)
- SSDs manejan escrituras append-only
- Perder estado local es economico: se reconstruye desde logs

#### 5. Object Storage como Base
El cambio mas transformador de la arquitectura:

```
Arquitectura Legacy TiDB          Arquitectura TiDB X
+--------+--------+--------+      +--------+--------+--------+
| TiKV-1 | TiKV-2 | TiKV-3 |      | Comp-1 | Comp-2 | Comp-N |
| Data   | Data   | Data   |      | Cache  | Cache  | Cache  |
| Local  | Local  | Local  |      | Only   | Only   | Only   |
+--------+--------+--------+      +---+----+---+----+---+----+
                                       |        |        |
                                  +----v--------v--------v----+
                                  |     Object Storage (S3)    |
                                  |   SST Files + Raft Logs    |
                                  +----------------------------+
```

### 10.3 Comparativa: TiDB Legacy vs TiDB X

| Aspecto | TiDB Legacy | TiDB X |
|---|---|---|
| **Almacenamiento** | Local, shared-nothing | Object Storage (S3) + cache local |
| **Modelo de computo** | Nodos stateful | Servicios stateless |
| **Escalado** | Requiere rebalanceo de datos (horas) | Instantaneo via snapshots en object storage |
| **Multi-tenancy** | Instancia fisica por usuario | Millones via Virtual Clusters |
| **Costo minimo** | Cientos $/mes | Free tier disponible |
| **Creacion de cluster** | Minutos | Segundos |
| **Escalado automatico** | Limitado | 0 -> 100M QPS/TPS y viceversa |

### 10.4 Capacidades Tecnicas

- **Escalado ascendente**: Miles de nodos de computo en segundos, cada uno cargando shards desde object storage en paralelo
- **Escalado descendente**: Actualizacion de tablas de routing y terminacion inmediata de nodos (sin rebalanceo)
- **Raft Log Partitioning**: Organizado por shard, elimina cuellos de botella de single-writer
- **Logical Sharding**: Los shards residen en object storage, permitiendo carga instantanea en nuevos nodos

### 10.5 Donde se Usa TiDB X

TiDB X **no es un producto independiente**. Impulsa internamente:
- **TiDB Cloud Starter**: Cargas auto-scaling
- **TiDB Cloud Essential**: Cargas con balance de escalabilidad y seguridad
- **TiDB Cloud Dedicated**: Cargas empresariales predecibles

---

## 11. TiDB en Alibaba Cloud

### Partnership Estrategico PingCAP + Alibaba Cloud

PingCAP tiene una alianza estrategica con Alibaba Cloud para ofrecer TiDB como servicio gestionado en el Alibaba Cloud Marketplace.

**Relacion de inversion**: Alibaba (a traves de sus brazos de inversion) ha sido **inversor en PingCAP**. PingCAP levanto una ronda Serie D de $270 millones USD en 2020, con participacion de inversores del ecosistema tecnologico chino e internacional.

### Servicios Disponibles

| Tier | Precio | Caracteristicas |
|---|---|---|
| **TiDB Cloud Starter** | Desde $0/mes (25GB gratis) | Auto-scaling, escala a cero, 99.9% SLA |
| **TiDB Cloud Essential** | Desde ~$500/mes | Redundancia 3-AZ, Private Link, 99.99% SLA |

### Regiones Disponibles en Alibaba Cloud

- Singapore
- Tokyo
- Jakarta
- Mexico

### Integraciones con Ecosistema Alibaba

- **Alibaba Cloud ECS**: Computo
- **DataWorks**: Pipelines de datos
- **AI/ML Tools**: Herramientas de IA de Alibaba
- **OSS (Object Storage Service)**: Import/Export de datos
- **Kubernetes (ACK/ASK)**: Despliegue via TiDB Operator
- **DMS (Data Management Service)**: TiDB soportado como tipo de base de datos

### Despliegue en Alibaba Cloud Kubernetes

TiDB puede desplegarse en Alibaba Cloud usando:
- **ACK (Alibaba Container Service for Kubernetes)**: Kubernetes gestionado
- **ASK (Alibaba Serverless Kubernetes)**: Kubernetes serverless
- **TiDB Operator**: Automatiza lifecycle del cluster

### Billing

- Billing unificado via Alibaba Cloud Marketplace
- Facturacion integrada con la cuenta de Alibaba Cloud

---

## 12. Busqueda Vectorial e IA

Desde **TiDB v8.4.0** (recomendado v8.5.0+), TiDB incluye capacidades de busqueda vectorial nativas.

### Tipo de Dato Vector

```sql
-- Crear tabla con columna vector
CREATE TABLE documents (
    id INT PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536)  -- Vector de 1536 dimensiones (OpenAI ada-002)
);

-- Insertar vector
INSERT INTO documents VALUES (1, 'Hello world', '[0.1, 0.2, ..., 0.9]');

-- Busqueda de similitud (cosine distance)
SELECT id, content,
       VEC_COSINE_DISTANCE(embedding, '[0.15, 0.25, ..., 0.85]') as distance
FROM documents
ORDER BY distance
LIMIT 10;
```

### Indice Vectorial

- **HNSW** (Hierarchical Navigable Small World): Indice de busqueda vectorial eficiente
- Operaciones de similitud: cosine distance, L2 distance, inner product

### Integraciones IA

| Framework | Soporte |
|---|---|
| **LangChain** | TiDB Vector Store integrado |
| **LlamaIndex** | Conector disponible |
| **Dify** | Integracion nativa |
| **OpenAI** | Compatible (embeddings API) |

### Caso de Uso: RAG (Retrieval-Augmented Generation)

TiDB permite combinar datos transaccionales, analiticos Y vectoriales en una sola base de datos, eliminando la necesidad de un vector database separado para aplicaciones RAG/AI.

---

## 13. Monitorizacion y Observabilidad

### Stack de Monitoring

```
TiDB/TiKV/TiFlash/PD
        |
        | (metricas via endpoints)
        v
   Prometheus
        |
        v
     Grafana
   (Dashboards)
```

### Dashboards Grafana Incluidos

| Dashboard | Contenido |
|---|---|
| **Overview** | Vista general del cluster, QPS, latencia, storage |
| **TiDB** | Metricas del SQL layer (query duration, statement count, connection count) |
| **TiKV** | Metricas de almacenamiento (Raft, storage, coprocessor, gRPC) |
| **TiFlash** | Metricas del motor columnar |
| **PD** | Metricas de scheduling, TSO, hot regions |
| **Performance Overview** | Dashboard integrado para diagnostico rapido |
| **Node Exporter** | Metricas de sistema (CPU, memoria, disco, red) |
| **Disk Performance** | I/O detallado de discos |

### TiDB Dashboard (Built-in)

Interfaz web integrada en PD con:
- **Cluster Info**: Estado de todos los nodos
- **Key Visualizer**: Mapa de calor de acceso a datos (hot spots)
- **SQL Statements**: Top SQL por tiempo/frecuencia
- **Slow Query**: Analisis de queries lentas con planes de ejecucion
- **Diagnostics**: Diagnosticos automatizados
- **Log Search**: Busqueda distribuida de logs

### Metricas Clave a Monitorear

| Metrica | Umbral de Alerta |
|---|---|
| **Query Duration P99** | > 200ms requiere investigacion |
| **TiKV CPU** | > 80% indica necesidad de escalar |
| **Region Count** | Balanceado entre nodos |
| **Raft Propose Wait Duration** | > 100ms indica contención |
| **TSO Wait Duration** | > 50ms indica problema con PD |
| **Storage Usage** | > 80% planificar expansion |

### Performance Tuning

Tecnicas comprobadas:
- **Prepared Statement Cache**: Reduce latencia de queries repetitivos
- **RC Read Feature**: Reduce duracion promedio de queries en >40%
- **Coprocessor Cache**: Cachea resultados de pushdown en TiKV
- **Index Optimization**: Indices multi-valor (desde v7.1)
- **Statistics Update**: `ANALYZE TABLE` regular para estadisticas actualizadas del CBO

---

## 14. Historial de Versiones

### Versiones LTS (Long-Term Support)

| Version | Fecha GA | Novedades Clave |
|---|---|---|
| **TiDB 7.1.0 LTS** | Mayo 31, 2023 | Checkpoint Restore, Multi-valued indexes GA, Resource Control |
| **TiDB 7.5.0 LTS** | Dic 1, 2023 | Distributed eXecution Framework (DXF) GA, DDL paralelo |
| **TiDB 8.1.0 LTS** | Mayo 24, 2024 | `IMPORT INTO ... FROM SELECT` GA, mejoras CBO |
| **TiDB 8.5.0 LTS** | Dic 19, 2024 | Creacion acelerada de tablas (millones), MVCC in-memory engine en TiKV, Vector Search |

### Versiones DMR (Development Milestone Release)

| Version | Novedades |
|---|---|
| **TiDB 8.2.0 DMR** | Mejoras de rendimiento y estabilidad |
| **TiDB 8.3.0 DMR** | Features experimentales |
| **TiDB 8.4.0 DMR** | Vector Search (beta), mejoras MPP |

### Evolucion Historica

| Ano | Hito |
|---|---|
| **2015** | PingCAP fundada, inicio del desarrollo de TiDB |
| **2016** | Primera release publica de TiDB |
| **2017** | TiDB 1.0 GA |
| **2018** | TiDB 2.0 - Mejoras de estabilidad y rendimiento |
| **2019** | TiDB 3.0 - Transacciones pesimistas, TiFlash beta |
| **2020** | TiDB 4.0 - TiFlash GA, TiDB Dashboard, TiKV gradua CNCF |
| **2021** | TiDB 5.0 - MPP engine, Clustered Index |
| **2022** | TiDB 6.0 - Placement Rules in SQL, Hot Spot scheduling mejorado |
| **2023** | TiDB 7.x - Resource Control, DXF, TTL Tables |
| **2024** | TiDB 8.x - Vector Search, MVCC in-memory engine |

---

## 15. Casos de Uso en Produccion

### Empresas Notables

| Empresa | Industria | Uso |
|---|---|---|
| **ByteDance (TikTok)** | Internet/Social | Cientos de clusters TiDB, uno de los mayores usuarios globales |
| **Square (Block)** | Fintech | Escalar workloads MySQL crecientes |
| **Shopee** | E-commerce | Transacciones de alta concurrencia |
| **China UnionPay** | Pagos | Sistema de pagos distribuido |
| **Ele.me** | Delivery | 80% trafico en vivo en TiKV (4 DCs, 100+ nodos, decenas de TB) |
| **Dailymotion** | Video | Transacciones en tiempo real, reduccion de costos migrando a TiDB Cloud |
| **ENGIE** | Energia | Multiples sistemas de produccion |
| **Zhihu** | Social/Q&A | Analisis en tiempo real |
| **Meituan** | Delivery | HTAP para operaciones y analitica |
| **iQIYI** | Streaming | Gestion de datos de usuarios |
| **Xiaomi** | Electronica | Plataforma de datos unificada |
| **JD.com** | E-commerce | Procesamiento de pedidos y analitica |
| **Bilibili** | Streaming/Video | Plataforma de video |
| **BookMyShow** | Entretenimiento | Ticketing (India, mayor plataforma) |
| **PayPay** | Fintech (Japon) | Servicio de pagos moviles |
| **WeBank** | Banca | Servicios financieros |

### Escala de Adopcion

- **+3,000 empresas** globales usan TiDB
- Industrias principales: **Finanzas, E-commerce, Gaming, Logistica, Energia, Telecomunicaciones**
- Despliegues de **centenas de nodos** con **decenas de Terabytes**
- Picos de concurrencia en eventos tipo **Double 11** (Singles' Day)

### Patrones de Uso Comunes

1. **Reemplazo de MySQL sharding**: Eliminar middleware de sharding (Vitess, ProxySQL)
2. **HTAP unificado**: Eliminar pipeline ETL hacia data warehouse
3. **Escalabilidad bajo demanda**: Picos estacionales (e-commerce, gaming)
4. **Multi-datacenter**: Alta disponibilidad cross-region
5. **Migracion gradual**: DM replica de MySQL mientras se migra tabla por tabla

---

## 16. Comparativa con Competidores

### TiDB vs CockroachDB vs YugabyteDB vs Google Spanner vs Amazon Aurora

| Caracteristica | TiDB | CockroachDB | YugabyteDB | Google Spanner | Amazon Aurora |
|---|---|---|---|---|---|
| **Compatibilidad SQL** | MySQL 8.0 | PostgreSQL | PostgreSQL | GoogleSQL/PostgreSQL | MySQL/PostgreSQL |
| **Arquitectura** | Separacion computo/storage | Multi-layered | Tablet-based | Propietaria global | Shared storage |
| **HTAP Nativo** | Si (TiFlash) | No | No | Limitado | No |
| **Licencia** | Apache 2.0 | BSL / Apache | Apache 2.0 | Propietaria | Propietaria |
| **Aislamiento** | Snapshot (SI) | Serializable | Snapshot/Serializable | External Consistency | Repeatable Read |
| **Auto-sharding** | Si | Si | Si | Si | No (single-writer) |
| **Escala global** | Multi-region | Multi-region | Multi-region | Global nativo | Multi-region |
| **Open Source** | Si | Core BSL | Si | No | No |
| **Busqueda vectorial** | Si (v8.4+) | No | No | No | Si (Aurora ML) |
| **Free Tier Cloud** | Si (generoso) | Si | Trial | No | No |
| **Motor analitico** | TiFlash (MPP) | No | No | No | No |
| **Consensus** | Multi-Raft | Multi-Raft | Multi-Raft | Paxos/TrueTime | Custom |

### Fortalezas Unicas de TiDB

1. **HTAP real**: Unica BD distribuida con motor OLAP columnar integrado (TiFlash MPP)
2. **Compatibilidad MySQL**: Drop-in replacement para aplicaciones MySQL
3. **Open Source completo**: Apache 2.0 sin restricciones BSL
4. **Vector Search nativo**: Busqueda vectorial integrada para IA/RAG
5. **Ecosistema de herramientas**: Lightning, DM, TiCDC, BR - herramientas maduras de migracion

### Debilidades Relativas de TiDB

1. **No Serializable**: Solo Snapshot Isolation (vs CockroachDB que ofrece Serializable)
2. **Indice sin afinidad local**: Indices y PKs no tienen locality affinity (vs PolarDB-X, OceanBase)
3. **Sin stored procedures**: Limitacion para migraciones MySQL complejas
4. **TSO centralizado**: PD es punto de contención potencial (mitigado con TSO proxy)

---

## 17. Limitaciones Conocidas

### Funcionalidades No Soportadas

| Feature | Estado | Alternativa |
|---|---|---|
| Stored Procedures | No soportado | Logica en aplicacion |
| Triggers | No soportado | TiCDC o logica en app |
| User-Defined Functions | No soportado | Expression Index o logica en app |
| Events / Scheduler | No soportado | Cron externo |
| XA Transactions | No soportado | Transacciones nativas TiDB |
| FULLTEXT Index | No soportado | Elasticsearch externo |
| Spatial Index (completo) | Parcial | PostGIS externo |
| SERIALIZABLE Isolation | No soportado | Snapshot Isolation (SI) |
| Multiple ALTER en un statement | Limitado | Multiples ALTER separados |
| Algunos type casts (DECIMAL->DATE) | No soportado | Cast intermedio |
| Index hints (HASH/BTREE/RTREE) | Parseados pero ignorados | - |

### Limites de Escala

| Dimension | Limite |
|---|---|
| **Nodos de computo** | Max 512 nodos TiDB |
| **Concurrencia por nodo** | Max 1,000 |
| **Capacidad de cluster** | Nivel de Petabytes |
| **Tamano de transaccion** | Default 100 MB (configurable) |
| **Tamano de fila** | Max 6 MB |
| **Columnas por tabla** | Max 1,017 |
| **Tamano de una sola tabla** | Sin limite duro (auto-sharding) |
| **Regions** | Millones por cluster |
| **Memory limit por query** | Configurable (default 1 GB) |

### Rendimiento y Benchmarks

| Metrica | Resultado Tipico |
|---|---|
| **Latencia point read** | 1-5 ms (por PK) |
| **Latencia point write** | 5-15 ms (por Raft consensus) |
| **TPC-C** | ~324,457 tpmC (benchmark publicado por PingCAP, 2020) |
| **Sysbench point select** | ~30,000-50,000+ QPS por TiDB Server individual |
| **Escalado de lectura** | Casi lineal anadiendo TiDB Servers |
| **Escalado de escritura** | Lineal anadiendo nodos TiKV |
| **TPC-H 100GB (TiFlash MPP)** | Mayoria de 22 queries en segundos a decenas de segundos |
| **Replicacion TiKV->TiFlash** | Sub-segundo (tiempo real) |

> **Nota**: El rendimiento varia segun hardware, configuracion, y patrones de carga. Las cifras son orientativas basadas en benchmarks publicados.

### Anti-patrones a Evitar

1. **Hotspot de escritura en auto-increment PK**: Usar `AUTO_RANDOM` en lugar de `AUTO_INCREMENT` para tablas de alto throughput
2. **Transacciones muy grandes**: Dividir en lotes mas pequenos
3. **Queries sin indices**: TiKV full scan es costoso en distribuido
4. **Depender de LIMIT sin ORDER BY**: Resultados no deterministicos en distribuido
5. **Asumir auto-increment continuo**: IDs se asignan en lotes, pueden tener gaps
6. **`SELECT COUNT(*) FROM tabla`**: Requiere full scan (no hay row count cacheado como InnoDB). Usar `SHOW TABLE STATUS` para conteo aproximado
7. **Usar TiDB como cache de baja latencia**: La latencia minima es ~0.5-1ms (vs ~0.1ms en MySQL standalone). Usar Redis para caching
8. **Transacciones OLTP de larga duracion**: Bloquean el GC safe point, degradando rendimiento de scans en todo el cluster
9. **Over-partitioning**: Cada particion genera sus propias Regions. Limite: 8,192 particiones por tabla
10. **Ignorar `backoff` y `cop_wait` en slow queries**: Indican problemas de infraestructura, no de SQL

---

## 18. Comparativa con PolarDB-X de Alibaba

PolarDB-X es la base de datos distribuida propia de Alibaba Cloud, que compite directamente con TiDB.

### PolarDB-X vs TiDB

| Aspecto | TiDB | PolarDB-X (Alibaba) |
|---|---|---|
| **Origen** | PingCAP (open-source) | Alibaba Cloud (propietario + open-source) |
| **Compatibilidad** | MySQL 8.0 | MySQL (con extensiones) |
| **Arquitectura** | Shared-nothing distribuido | Shared-nothing con locality affinity |
| **HTAP** | Si (TiFlash MPP) | Si (MPP integrado) |
| **Indice local** | Sin locality affinity (costoso) | Con locality affinity (eficiente) |
| **Cloud Lock-in** | Multi-cloud (AWS, GCP, Alibaba) | Solo Alibaba Cloud |
| **Open Source** | Si (Apache 2.0) | Parcialmente open-source |
| **Proven at Scale** | Tmall Double 11 (Alibaba interno) | Tmall Double 11 (infra Alibaba) |
| **Transacciones locales** | Siempre distribuidas | Locales cuando datos colocados |
| **Rendimiento indice** | Menor (distribuido siempre) | Mayor (locality affinity) |
| **Ecosistema** | Global, multi-cloud | Centrado en Alibaba Cloud |
| **Comunidad** | Grande, global, open-source activo | Mas limitada |

### Cuando Elegir TiDB sobre PolarDB-X

- Necesitas **multi-cloud** o portabilidad
- Priorizas **open-source** sin vendor lock-in
- Necesitas **HTAP** con motor columnar dedicado (TiFlash)
- Tu equipo esta familiarizado con el ecosistema TiDB
- Necesitas **busqueda vectorial** integrada

### Cuando Elegir PolarDB-X sobre TiDB

- Tu infraestructura esta en **Alibaba Cloud exclusivamente**
- Necesitas **rendimiento maximo** en indices locales con locality affinity
- Prefieres soporte integrado con el ecosistema Alibaba
- Necesitas **transacciones locales** de alto rendimiento

---

## 19. Ecosistema de BDs Distribuidas en China

TiDB opera en un contexto competitivo unico dentro del mercado chino de bases de datos distribuidas, impulsado por politicas nacionales de autosuficiencia tecnologica (**Xin Chuang** / innovacion domestica).

### Principales Competidores Chinos

| Base de Datos | Empresa | Compatibilidad | Tipo | Cloud Nativo |
|---|---|---|---|---|
| **TiDB** | PingCAP | MySQL 8.0 | HTAP Distribuido | TiDB Cloud (AWS, GCP, Alibaba) |
| **OceanBase** | Ant Group (ex-Alibaba) | MySQL / Oracle | HTAP Distribuido | OceanBase Cloud, Alibaba Cloud |
| **PolarDB-X** | Alibaba Cloud | MySQL | Distribuido OLTP | Alibaba Cloud nativo |
| **PolarDB** | Alibaba Cloud | MySQL / PostgreSQL / Oracle | Shared-storage | Alibaba Cloud nativo |
| **GaussDB** | Huawei | PostgreSQL / OpenGauss | Distribuido | Huawei Cloud nativo |
| **TDSQL** | Tencent | MySQL / PostgreSQL | Distribuido | Tencent Cloud nativo |
| **openGauss** | Huawei (open-source) | PostgreSQL | Standalone/Distribuido | Multi-cloud |

### OceanBase (Ant Group / Alibaba)

- Desarrollado originalmente dentro de Alibaba/Ant Group para **Alipay**
- Ahora es una empresa independiente (OceanBase Technology)
- Usado extensamente en sistemas financieros de Alipay
- Compatible con MySQL y Oracle
- Record mundial en **TPC-C** benchmark
- Disponible en Alibaba Cloud como servicio gestionado
- Competidor directo de TiDB en el mercado chino financiero

### Dinamica de Mercado

Cada gran proveedor cloud chino ha desarrollado su propia BD distribuida en lugar de ofrecer TiDB como servicio gestionado:

```
Alibaba Cloud --> PolarDB-X, PolarDB, OceanBase
Tencent Cloud --> TDSQL
Huawei Cloud  --> GaussDB, openGauss
PingCAP       --> TiDB Cloud (multi-cloud independiente)
```

Esto posiciona a TiDB como la opcion **multi-cloud, vendor-neutral y open-source** frente a soluciones propietarias atadas a un proveedor cloud especifico.

### Nota sobre "TiDBX" como Termino

> **Aclaracion importante**: "TiDBX" como producto de Alibaba Cloud **no existe**. El termino puede confundirse con:
> - **TiDB X**: Motor cloud-native de PingCAP que impulsa TiDB Cloud (ver Seccion 10)
> - **PolarDB-X**: BD distribuida MySQL-compatible propia de Alibaba Cloud (ver Seccion 18)
> - **TiDB on Alibaba Cloud**: TiDB desplegado en Alibaba Cloud via Marketplace (ver Seccion 11)

---

## Referencias y Fuentes

### Documentacion Oficial
- [TiDB Docs](https://docs.pingcap.com/tidb/stable/)
- [TiDB Architecture](https://docs.pingcap.com/tidb/stable/tidb-architecture/)
- [TiFlash Overview](https://docs.pingcap.com/tidb/stable/tiflash-overview/)
- [MySQL Compatibility](https://docs.pingcap.com/tidb/stable/mysql-compatibility/)
- [TiDB Tools Overview](https://docs.pingcap.com/tidb/stable/ecosystem-tool-user-guide/)
- [Security Concepts](https://docs.pingcap.com/tidbcloud/security-concepts/)
- [Release Notes](https://docs.pingcap.com/tidb/stable/release-notes/)
- [Vector Search Overview](https://docs.pingcap.com/tidbcloud/vector-search-overview/)

### Articulos Tecnicos
- [TiDB X: Origins, Architecture, and What's to Come](https://www.pingcap.com/blog/tidbx-origins-architecture/)
- [Understanding TiDB's Cluster Architecture](https://www.pingcap.com/article/understanding-tidbs-cluster-architecture-and-core-components/)
- [ACID Transactions in Distributed Databases](https://www.pingcap.com/blog/distributed-transactions-tidb/)
- [Understanding MVCC in TiDB](https://www.pingcap.com/article/understanding-mvcc-in-tidb-for-high-concurrency-applications/)
- [Deep Dive into Distributed Transactions](https://dataturbo.medium.com/deep-dive-into-distributed-transactions-in-tikv-and-tidb-80337b4104cb)

### Alibaba Cloud
- [TiDB on Alibaba Cloud Marketplace](https://www.alibabacloud.com/en/marketplace/tidb)
- [TiDB Cloud on Alibaba Cloud](https://www.pingcap.com/partners/alibaba-cloud/)
- [TiDB Cloud Essential on AWS and Alibaba Cloud](https://www.pingcap.com/blog/tidb-cloud-essential-now-available-public-preview-aws-alibaba-cloud/)
- [Deploy TiDB on Alibaba Cloud ASK](https://www.alibabacloud.com/blog/deploy-tidb-on-alibaba-cloud-serverless-kubernetes-ask_600343)
- [Evaluation: TiDB, OceanBase, PolarDB-X, CockroachDB](https://www.alibabacloud.com/en/developer/a/open-source/tidb-oceanbase-polardb-x-cockroachdb)

### Codigo Fuente
- [TiDB GitHub](https://github.com/pingcap/tidb) - Motor SQL (Go)
- [TiKV GitHub](https://github.com/tikv/tikv) - Storage Engine (Rust)
- [PD GitHub](https://github.com/tikv/pd) - Placement Driver (Go)

### Comparativas
- [Distributed SQL 2025: CockroachDB vs TiDB vs YugabyteDB](https://sanj.dev/post/distributed-sql-databases-comparison)
- [TiDB vs MySQL 2025](https://www.bytebase.com/blog/tidb-vs-mysql/)
- [PingCAP Customers](https://www.pingcap.com/customers/)

---

> **Ultima actualizacion**: Marzo 2026
> **Versiones cubiertas**: TiDB hasta v8.5.0 LTS, TiDB X, TiDB Cloud