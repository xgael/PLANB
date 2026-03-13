# QWEN - La Familia de Modelos de IA de Alibaba Cloud

## Tabla de Contenidos

1. [Visión General](#1-visión-general)
2. [Historia y Cronología Completa](#2-historia-y-cronología-completa)
3. [Familias y Generaciones de Modelos](#3-familias-y-generaciones-de-modelos)
4. [Arquitectura Técnica](#4-arquitectura-técnica)
5. [Modelos Especializados](#5-modelos-especializados)
6. [QwQ - Modelo de Razonamiento](#6-qwq---modelo-de-razonamiento)
7. [Benchmarks y Rendimiento](#7-benchmarks-y-rendimiento)
8. [Licenciamiento](#8-licenciamiento)
9. [Cómo Acceder a Qwen](#9-cómo-acceder-a-qwen)
10. [Integración con Alibaba Cloud](#10-integración-con-alibaba-cloud)
11. [Qwen-Agent Framework](#11-qwen-agent-framework)
12. [Metodología de Entrenamiento](#12-metodología-de-entrenamiento)
13. [Casos de Uso y Aplicaciones](#13-casos-de-uso-y-aplicaciones)
14. [Ecosistema y Comunidad Open-Source](#14-ecosistema-y-comunidad-open-source)
15. [Recursos y Enlaces](#15-recursos-y-enlaces)

---

## 1. Visión General

**Qwen** (en chino: 通义千问 / Tongyi Qianwen, "Comprendiendo Mil Preguntas") es una familia a gran escala de modelos fundacionales de IA desarrollada por **Alibaba Cloud**, la subsidiaria de computación en la nube de Alibaba Group. Incluye:

- **LLMs** (Large Language Models) - Modelos de lenguaje generativos
- **VL Models** (Vision-Language) - Modelos multimodales de visión y lenguaje
- **Audio Models** - Modelos de comprensión de audio
- **Code Models** - Modelos especializados en programación
- **Math Models** - Modelos especializados en matemáticas
- **MoE Models** (Mixture of Experts) - Modelos de mezcla de expertos
- **Reasoning Models** (QwQ) - Modelos de razonamiento profundo

### Organización y Equipo

- **Desarrollador**: Qwen Team dentro de Alibaba Cloud Intelligence (absorbió gran parte de la antigua **DAMO Academy**)
- **DAMO Academy**: Laboratorio de investigación global de Alibaba, establecido en 2017 con $15B de inversión en investigación
- **Liderazgo**: Jingren Zhou (VP de Alibaba Group, jefe de la división AI de Alibaba Cloud)
- **Investigador principal**: Junyang Lin (dirección técnica, representante público del proyecto)
- **Sede**: Hangzhou, China

### Contexto Estratégico

Qwen es central para la estrategia de IA de Alibaba:
- Alimenta el chatbot de consumo **Tongyi Qianwen** (comparable a ChatGPT), con **100M+ usuarios** reportados a inicios de 2025
- Integrado en los negocios de e-commerce, logística y cloud de Alibaba
- Ofrecido comercialmente a través de la plataforma **Model Studio** de Alibaba Cloud
- Compite directamente con: Baidu ERNIE, Tencent Hunyuan, ByteDance Doubao, Meta Llama, Mistral, Google Gemma, OpenAI GPT y Anthropic Claude

---

## 2. Historia y Cronología Completa

| Fecha | Lanzamiento | Detalles |
|-------|-------------|---------|
| Ago 3, 2023 | **Qwen-7B, Qwen-7B-Chat** | Primer lanzamiento público; LLM open-weight inicial |
| Sep 25, 2023 | **Qwen-14B, Qwen-14B-Chat** | Variante más grande |
| Sep 2023 | **Qwen-VL, Qwen-VL-Chat** | Primer modelo visión-lenguaje |
| Nov 2023 | **Qwen-Audio, Qwen-Audio-Chat** | Primer modelo de comprensión de audio |
| Nov 30, 2023 | **Qwen-72B, Qwen-72B-Chat** | Modelo Qwen1 más grande |
| Dic 2023 | **Qwen-1.8B, Qwen-1.8B-Chat** | Modelo pequeño para edge/móvil |
| Feb 5, 2024 | **Qwen1.5 (todas las tallas)** | Upgrade mayor: 0.5B a 110B; mejora multilingüe y alineamiento |
| Mar 28, 2024 | **Qwen1.5-MoE-A2.7B** | Primer modelo Mixture-of-Experts |
| Abr 2024 | **CodeQwen1.5-7B** | Primer modelo de código dedicado |
| Jun 7, 2024 | **Qwen2 (todas las tallas)** | Nueva generación arquitectónica; GQA, tokenizador mejorado |
| Jul 2024 | **Qwen2-Audio** | Modelo de audio actualizado |
| Ago 2024 | **Qwen2-VL** | Gran upgrade de visión-lenguaje con resolución dinámica |
| Ago 2024 | **Qwen2-Math** | Modelo dedicado de razonamiento matemático |
| Sep 19, 2024 | **Qwen2.5 (familia completa)** | Mayor lanzamiento open-weight: LLMs base, Coder, Math |
| Nov 12, 2024 | **Qwen2.5-Coder-32B** | Modelo de código flagship open-source |
| Nov 28, 2024 | **QwQ-32B-Preview** | Primer modelo de razonamiento (chain-of-thought) |
| Ene 2025 | **Qwen2.5-VL** | Modelos visión-lenguaje actualizados (3B, 7B, 72B) |
| Ene 2025 | **Qwen2.5-Max** | Flagship MoE propietario (solo API) |
| Feb 2025 | **Qwen2.5-VL-32B** | Modelo visión-lenguaje de tamaño medio |
| Mar 6, 2025 | **QwQ-32B** | Lanzamiento completo del modelo de razonamiento (Apache 2.0) |
| Abr 29, 2025 | **Qwen3 (familia completa)** | Próxima generación con pensamiento híbrido |

---

## 3. Familias y Generaciones de Modelos

### 3.1 Generación 1: Qwen (Agosto - Diciembre 2023)

Arquitectura decoder-only transformer con Multi-Head Attention, RoPE, SwiGLU y RMSNorm.

| Modelo | Parámetros | Hidden Size | Capas | Attention Heads | Contexto |
|--------|-----------|-------------|-------|-----------------|----------|
| Qwen-1.8B | 1.8B | 2048 | 24 | 16 | 8K (extendible a 32K) |
| Qwen-7B | 7.7B | 4096 | 32 | 32 | 8K (extendible a 32K) |
| Qwen-14B | 14.2B | 5120 | 40 | 40 | 8K (extendible a 32K) |
| Qwen-72B | 72.7B | 8192 | 80 | 64 | 32K nativo |

- **Datos de entrenamiento**: ~3 billones de tokens (chino, inglés, código, multilingüe)
- **Licencia**: Tongyi Qianwen License (abierta para investigación; uso comercial < 100M MAU)

### 3.2 Generación 1.5: Qwen1.5 (Febrero 2024)

Misma arquitectura base con datos mejorados y mejor alineamiento.

**Modelos disponibles:**
- Qwen1.5-0.5B / 1.8B / 4B / 7B / 14B / 32B / 72B / **110B** (Chat para todos)
- **Qwen1.5-MoE-A2.7B** (14.3B total, 2.7B activos; 64 expertos, top-4 routing)

**Mejoras clave:**
- Todos los modelos soportan **32,768 tokens** de contexto
- Soporte multilingüe expandido más allá de chino/inglés
- Mejor instruction following y alineamiento de seguridad
- Modelos Chat alineados con SFT + DPO

### 3.3 Generación 2: Qwen2 (Junio 2024)

Nueva generación arquitectónica con cambios significativos.

| Modelo | Parámetros | Hidden Size | Capas | Query Heads | KV Heads (GQA) | Contexto |
|--------|-----------|-------------|-------|-------------|----------------|----------|
| Qwen2-0.5B | 0.49B | 896 | 24 | 14 | 2 | 32K |
| Qwen2-1.5B | 1.54B | 1536 | 28 | 12 | 2 | 32K |
| Qwen2-7B | 7.07B | 3584 | 28 | 28 | 4 | 128K |
| Qwen2-57B-A14B | 57B total / 14B activos | MoE | - | - | - | 64K |
| Qwen2-72B | 72.7B | 8192 | 80 | 64 | 8 | 128K |

**Cambios arquitectónicos desde Qwen1:**
- **Grouped Query Attention (GQA)** reemplazando Multi-Head Attention (reduce memoria KV-cache 2-8x)
- **Dual-chunk attention con YaRN** para extensión de contexto largo
- Tokenizador mejorado: **151,643 tokens** de vocabulario
- **Sliding Window Attention** en capas alternas
- Embedding tying removido para modelos grandes

**Datos de entrenamiento**: ~7 billones de tokens, expandido a **29 idiomas** (inglés, chino, francés, español, portugués, alemán, italiano, ruso, japonés, coreano, vietnamita, tailandés, árabe, turco, indonesio, polaco, holandés, checo, sueco, noruego, danés, finlandés, húngaro, rumano, griego, ucraniano, hebreo, malayo, bengalí)

**Licencia**: Apache 2.0 para 0.5B, 1.5B, 7B y 57B-A14B. Qwen2-72B inicialmente con licencia custom, luego relicenciado a Apache 2.0.

### 3.4 Generación 2.5: Qwen2.5 (Septiembre 2024)

El lanzamiento open-weight más comprehensivo de Qwen.

#### Modelos Open-Weight

| Modelo | Parámetros | Contexto | Licencia |
|--------|-----------|----------|----------|
| Qwen2.5-0.5B | 0.49B | 32K | Apache 2.0 |
| Qwen2.5-1.5B | 1.54B | 32K | Apache 2.0 |
| Qwen2.5-3B | 3.09B | 32K | Apache 2.0 |
| Qwen2.5-7B | 7.61B | 128K | Apache 2.0 |
| Qwen2.5-14B | 14.7B | 128K | Apache 2.0 |
| Qwen2.5-32B | 32.5B | 128K | Apache 2.0 |
| Qwen2.5-72B | 72.7B | 128K | Apache 2.0 |

Cada tamaño disponible en variantes **Base** (preentrenado) e **Instruct** (chat/instrucciones).

#### Modelos Propietarios API

| Modelo API | Descripción | Contexto |
|------------|-------------|----------|
| **Qwen2.5-Turbo** | Optimizado para costo, inferencia rápida | Hasta **1M tokens** |
| **Qwen2.5-Plus** | Modelo mid-tier balanceado | 128K |
| **Qwen2.5-Max** | Flagship MoE, máxima capacidad | 32K-128K |
| **Qwen-Long** | Especialista en contexto largo | Hasta **1M tokens** |

**Datos de entrenamiento**: ~**18 billones de tokens** (2.5x más que Qwen2)

**Mejoras clave sobre Qwen2:**
- Instruction following sustancialmente mejorado
- Mejor generación de salida estructurada (JSON, XML, tablas markdown)
- Comprensión de contexto largo mejorada hasta 128K tokens
- Matemáticas y código mejorados incluso en modelos base
- Capacidades multilingües expandidas
- Mejor alineamiento de seguridad y reducción de alucinaciones

### 3.5 Generación 3: Qwen3 (Abril 2025)

La generación más reciente.

| Modelo | Tipo | Parámetros Totales | Parámetros Activos |
|--------|------|-------------------|-------------------|
| Qwen3-0.6B | Dense | 0.6B | 0.6B |
| Qwen3-1.7B | Dense | 1.7B | 1.7B |
| Qwen3-4B | Dense | 4B | 4B |
| Qwen3-8B | Dense | 8B | 8B |
| Qwen3-14B | Dense | 14B | 14B |
| Qwen3-30B-A3B | **MoE** | ~30B | ~3B |
| Qwen3-32B | Dense | 32B | 32B |
| Qwen3-235B-A22B | **MoE** | ~235B | ~22B (flagship) |

#### Innovación Definitoria: Modo de Pensamiento Híbrido

Cada modelo Qwen3 puede operar en dos modos:

1. **Thinking Mode** (`/think`): Razonamiento extendido chain-of-thought, similar a OpenAI o1 o QwQ. El modelo razona paso a paso antes de producir una respuesta final. Ideal para: matemáticas, código, lógica y análisis complejo.

2. **Non-Thinking Mode** (`/no_think`): Respuestas rápidas y directas sin razonamiento extendido. Ideal para: preguntas simples, chat y aplicaciones sensibles a latencia.

**Un solo modelo maneja ambos modos** - no se necesitan despliegues separados.

**Detalles de entrenamiento:**
- Preentrenado en ~**36 billones de tokens** (doble que Qwen2.5)
- **119 idiomas y dialectos** soportados
- Pipeline de entrenamiento en 4 etapas:
  1. Preentrenamiento a gran escala en 30T+ tokens de datos generales
  2. Preentrenamiento continuado mejorado para razonamiento (STEM, código, lógica)
  3. Integración del modo thinking (entrenar al modelo para producir chain-of-thought)
  4. Alineamiento con Reinforcement Learning (GRPO) para ambos modos

**Rendimiento:**
- Qwen3-235B-A22B: competitivo con GPT-4o, Claude 3.5 Sonnet, DeepSeek-V3
- Qwen3-30B-A3B: iguala el rendimiento de Qwen2.5-72B usando ~4% de los parámetros activos
- Qwen3-4B: iguala a Qwen2.5-72B en algunos benchmarks de razonamiento en thinking mode

**Licencia**: Apache 2.0 para todos los modelos open-weight.

---

## 4. Arquitectura Técnica

### 4.1 Arquitectura Core (Qwen2/2.5/3)

Todos los modelos Qwen son **transformers autoregresivos decoder-only**.

**Estructura de cada bloque transformer:**

```
Input → RMSNorm → Self-Attention (GQA + RoPE) → Residual →
      → RMSNorm → FFN (SwiGLU) → Residual → Output
```

### 4.2 Decisiones Arquitectónicas Específicas

| Componente | Descripción |
|------------|-------------|
| **Pre-normalization (RMSNorm)** | Aplicado antes de cada sub-capa. Root Mean Square Layer Normalization, más rápido que LayerNorm estándar |
| **SwiGLU Activation** | Swish-Gated Linear Unit en la FFN. Tamaño intermedio FFN = 8/3 del hidden size (redondeado a múltiplo de 128) |
| **RoPE** | Rotary Position Embedding. Codifica información posicional rotando vectores query y key. Frecuencia base: 10,000 |
| **GQA** | Grouped Query Attention. Múltiples query heads comparten un KV head. Ej: Qwen2-72B tiene 64 query heads pero solo 8 KV heads (ratio 8:1). Reduce memoria KV-cache por el factor del ratio |
| **Sliding Window Attention** | En capas alternas, la atención se restringe a una ventana local (ej. 32K tokens). Otras capas usan atención global completa |
| **Sin bias** | Sin términos bias en capas lineales (siguiendo el enfoque LLaMA) |
| **Embedding tying** | En modelos pequeños (0.5B, 1.5B) los embeddings de entrada/salida están atados. Modelos más grandes tienen embeddings desatados |

### 4.3 Tokenizador

| Propiedad | Valor |
|-----------|-------|
| Algoritmo | Byte-level Byte-Pair Encoding (BBPE) vía `tiktoken` |
| Vocabulario | 151,643 tokens (Qwen2/2.5) |
| Optimización | Diseñado para alta eficiencia tanto en chino como en inglés |
| Eficiencia chino | 1-2 tokens por carácter chino (vs 2-3+ en tokenizador de LLaMA) |
| Formato chat | ChatML con `<\|im_start\|>` y `<\|im_end\|>` |
| Tokens especiales | `<\|endoftext\|>`, tokens de tool/function calling, placeholders de imagen/audio |

### 4.4 Mecanismos de Contexto Largo

| Mecanismo | Generación | Descripción |
|-----------|-----------|-------------|
| **NTK-aware Scaled RoPE** | Qwen1 | Modifica la frecuencia base de RoPE dinámicamente. Permite que modelos entrenados con 8K manejen 32K sin fine-tuning |
| **YaRN** | Qwen2+ | Extensión más sofisticada que escala tanto frecuencia como patrones de atención. Permite 32K→128K |
| **Dual-chunk attention** | Qwen2 | Divide secuencias largas en chunks y aplica atención dentro y entre chunks |
| **Sparse attention** | Qwen2.5-Turbo | Mecanismos avanzados para procesar hasta 1M tokens |

### 4.5 Variantes de Cuantización Disponibles

Para cada modelo, se producen múltiples versiones cuantizadas:

| Formato | Bits | Reducción Memoria | Notas |
|---------|------|-------------------|-------|
| BF16/FP16 | 16 | Baseline | Calidad completa |
| FP8 | 8 | ~50% | Pérdida mínima, GPUs modernas |
| GPTQ-Int8 | 8 | ~50% | Mejor calidad cuantizada |
| GPTQ-Int4 | 4 | ~75% | Pérdida mínima en modelos grandes |
| AWQ | 4 | ~75% | Alternativa a GPTQ, a menudo mejor calidad |
| GGUF | Variable | Variable | Para llama.cpp; Q2_K a F16 |

### 4.6 Requisitos de Memoria (Aproximados)

| Modelo | FP16/BF16 | Int8 | Int4/GPTQ |
|--------|-----------|------|-----------|
| 0.5B | ~1 GB | ~0.5 GB | ~0.3 GB |
| 1.5B | ~3 GB | ~1.5 GB | ~1 GB |
| 7B | ~14 GB | ~7 GB | ~4 GB |
| 14B | ~28 GB | ~14 GB | ~8 GB |
| 32B | ~64 GB | ~32 GB | ~18 GB |
| 72B | ~144 GB | ~72 GB | ~40 GB |

---

## 5. Modelos Especializados

### 5.1 Qwen-VL (Vision-Language Models)

#### Qwen-VL (Septiembre 2023) - Original
- Basado en Qwen-7B como backbone de lenguaje
- Encoder visual: **ViT-bigG** (~1.9B parámetros)
- Adaptador visión-lenguaje: módulo cross-attention de una capa
- Resolución de entrada: 448x448 fija
- Entrenado en ~1.4B pares imagen-texto

#### Qwen2-VL (Agosto 2024) - Upgrade Mayor
- Tamaños: **2B, 7B, 72B**
- **Naive Dynamic Resolution**: Procesa imágenes en su resolución nativa dividiendo en parches variables
- **M-RoPE (Multimodal RoPE)**: Embeddings posicionales para dimensiones temporales, alto y ancho
- **Video understanding**: Procesa videos como secuencias de frames
- OCR mejorado: documentos complejos, tablas, gráficos, texto manuscrito

#### Qwen2.5-VL (Enero 2025) - Última Versión
- Tamaños: **3B, 7B, 32B, 72B**
- Comprensión de video de horas de duración
- **Capacidades de agente visual**: Interactúa con GUIs, pantallas de teléfono, navegadores web
- Comprensión de documentos mejorada (facturas, recibos, formularios)
- Salida estructurada desde imágenes (JSON, tablas)

**Benchmarks Qwen2.5-VL-72B-Instruct:**

| Benchmark | Score | Categoría |
|-----------|-------|-----------|
| MMMU (val) | ~70% | Comprensión multimodal |
| MathVista | ~74% | Matemáticas visuales |
| DocVQA | ~96% | QA de documentos |
| ChartQA | ~88% | Comprensión de gráficos |
| TextVQA | ~84% | Texto en imágenes |
| OCRBench | ~88% | Calidad OCR |
| Video-MME | ~73% | Comprensión de video |

**Tipos de entrada soportados:** imágenes estáticas (cualquier resolución), múltiples imágenes, video, conversaciones interleaved imagen-texto, PDFs como imágenes, screenshots y GUIs.

### 5.2 Qwen-Audio (Modelos de Audio)

#### Qwen-Audio (Noviembre 2023)
- Basado en Qwen-7B
- Encoder de audio: **Whisper-large-v2** (OpenAI)
- ASR en 30+ idiomas, traducción speech-to-text, clasificación de eventos de audio, comprensión musical, identificación de hablante, reconocimiento de emociones

#### Qwen2-Audio (Julio 2024)
- Encoder de audio: **Whisper-large-v3** (~632M parámetros)
- Dos modos de interacción:
  1. **Voice Chat**: Entrada de voz directa, salida de texto
  2. **Audio Analysis**: Analizar clips de audio con consultas de texto

**Benchmarks:**

| Benchmark | Score | Categoría |
|-----------|-------|-----------|
| LibriSpeech clean (WER) | ~1.5% | ASR en inglés |
| LibriSpeech other (WER) | ~3.5% | ASR ruidoso |
| CoVoST2 (en→X) | Competitivo | Traducción de habla |

### 5.3 Qwen-Coder (Modelos de Código)

#### CodeQwen1.5-7B (Abril 2024)
- 7B parámetros basado en Qwen1.5
- 92 lenguajes de programación

#### Qwen2.5-Coder (Septiembre - Noviembre 2024)
- Tamaños: **0.5B, 1.5B, 3B, 7B, 14B, 32B**
- Entrenado en **5.5 billones de tokens** con énfasis en código fuente
- **92+ lenguajes de programación**: Python, JavaScript/TypeScript, Java, C/C++, Go, Rust, Ruby, PHP, Swift, Kotlin, SQL, HTML/CSS, Shell/Bash, R, MATLAB, Julia, Haskell, Scala, Perl, Lua, Dart, C# y muchos más

**Capacidades:**
- Generación de código desde lenguaje natural
- Completado de código (fill-in-the-middle)
- Explicación y documentación de código
- Detección y corrección de bugs
- Traducción entre lenguajes
- Comprensión a nivel de repositorio (128K contexto)
- Generación de tests
- Sugerencias de code review

**Benchmarks Qwen2.5-Coder-32B-Instruct:**

| Benchmark | Qwen2.5-Coder-32B | GPT-4o | Claude 3.5 Sonnet |
|-----------|-------------------|--------|-------------------|
| HumanEval | **92.7%** | 90.2% | 92.0% |
| HumanEval+ | **87.2%** | 86.8% | -- |
| MBPP | **90.2%** | -- | -- |
| Aider (polyglot) | 73.7% | 72.9% | 73.9% |
| LiveCodeBench | 55.2% | ~50% | ~55% |

### 5.4 Qwen-Math (Modelos de Matemáticas)

#### Qwen2-Math (Agosto 2024)
- Tamaños: 1.5B, 7B, 72B
- Chain-of-thought (CoT) y Tool-Integrated Reasoning (TIR con Python)

#### Qwen2.5-Math (Septiembre 2024)
- Tamaños: **1.5B, 7B, 72B**
- **Bilingüe** chino-inglés
- TIR mejorado: genera código Python, lo ejecuta e incorpora resultados

**Benchmarks Qwen2.5-Math-72B-Instruct:**

| Benchmark | CoT (razonamiento puro) | TIR (con código) |
|-----------|------------------------|-------------------|
| GSM8K | ~96.0% | ~97.0% |
| MATH (full) | ~85.0% | ~91.6% |
| MATH-500 | ~90% | ~95% |
| GaoKao 2024 | ~75% | ~80% |
| AIME 2024 | ~30% | ~47% |

### 5.5 Qwen-MoE (Mixture of Experts)

| Modelo MoE | Params Totales | Params Activos | Expertos Totales | Expertos Activos | Shared |
|------------|---------------|---------------|-----------------|-----------------|--------|
| Qwen1.5-MoE-A2.7B | 14.3B | 2.7B | 64 | 4 (top-4) | Sí |
| Qwen2-57B-A14B | 57B | 14B | 64 | 8 (top-8) | Sí |
| Qwen3-30B-A3B | ~30B | ~3B | No divulgado | No divulgado | Sí |
| Qwen3-235B-A22B | ~235B | ~22B | No divulgado | No divulgado | Sí |

**Innovaciones de diseño MoE de Qwen:**
1. **Segmentación de expertos fina**: En vez de 8 expertos grandes con top-2, usa 64 expertos pequeños con top-4. Mismo presupuesto de cómputo pero routing más expresivo
2. **Expertos shared + routed**: Capacidad siempre activa (shared) más expertos especializados (routed)
3. **Inicialización desde modelos densos**: "Upcycling" - convertir un modelo denso entrenado en MoE dividiendo la FFN en expertos
4. **Load balancing loss**: Loss auxiliar para prevenir colapso de expertos

---

## 6. QwQ - Modelo de Razonamiento

### 6.1 Qué es QwQ

**QwQ** ("Qwen with Questions" - referencia juguetona al emoticono QwQ y la naturaleza del modelo de cuestionar y razonar) fue desarrollado en respuesta a la serie o1 de OpenAI, demostrando que el razonamiento extendido chain-of-thought en tiempo de inferencia puede mejorar dramáticamente el rendimiento en problemas difíciles.

### 6.2 QwQ-32B-Preview (28 Noviembre 2024)

- **Parámetros**: 32.5B (denso)
- **Arquitectura**: Basado en Qwen2.5-32B
- **Contexto**: 32,768 tokens
- **Entrenamiento**: SFT + RL en tareas de razonamiento

**Limitaciones conocidas (reconocidas por el equipo):**
- Mezcla de idiomas (cambio entre chino e inglés mid-razonamiento)
- Bucles de razonamiento recursivo
- Gaps de alineamiento de seguridad
- Sin soporte de tool use
- Cadenas de razonamiento excesivamente largas

### 6.3 QwQ-32B (6 Marzo 2025) - Lanzamiento Completo

**Mejoras mayores sobre Preview:**
- Todas las limitaciones sustancialmente resueltas
- **Tool use soportado**: Puede llamar funciones y usar code interpreter durante razonamiento
- **Capacidades agentic**: Puede usarse en workflows multi-paso
- Mezcla de idiomas eliminada
- Bucles recursivos drásticamente reducidos
- Mejor alineamiento de seguridad
- **Contexto**: 131,072 tokens (128K)
- **Licencia**: Apache 2.0

### 6.4 Pipeline de Entrenamiento

1. **Modelo base**: Checkpoint preentrenado de Qwen2.5-32B
2. **Cold Start SFT**: Fine-tune con demostraciones de razonamiento chain-of-thought curadas (pruebas matemáticas, soluciones de código, razonamiento científico, puzzles lógicos)
3. **Scaled RL Training** con GRPO (Group Relative Policy Optimization):
   - **Math RL**: Problemas con respuestas numéricas verificables; reward = corrección
   - **Code RL**: Problemas con test suites; reward = tasa de tests pasados
   - **General RL**: Problemas open-ended con reward models
   - Algoritmo: Muestrear K respuestas por prompt → puntuar → actualizar policy para favorecer las mejores
4. **Refinamiento de patrones de pensamiento**: Penalizar razonamiento circular, recompensar cadenas concisas pero completas
5. **Preservación de capacidades generales**: Mezclar datos de instruction-following general durante RL

### 6.5 Cómo funciona QwQ en la práctica

```
<think>
Déjame analizar este problema paso a paso.

Primero, necesito entender qué se está preguntando...
[varios párrafos de razonamiento explícito]

Espera, déjame reconsiderar ese enfoque. Creo que hay una forma más simple...
[auto-corrección]

Entonces la respuesta debería ser...
Déjame verificar: [paso de verificación]

Sí, eso está correcto.
</think>

La respuesta es 42.
```

Las etiquetas `<think>` contienen la cadena de razonamiento (visible para desarrolladores, puede ocultarse de usuarios finales).

### 6.6 Benchmarks QwQ-32B

| Benchmark | QwQ-32B | DeepSeek-R1 (671B) | o1-mini |
|-----------|---------|---------------------|---------|
| AIME 2024 (pass@1) | **79.5%** | 79.8% | 63.6% |
| MATH-500 | 95.8% | **97.3%** | 96.0% |
| LiveCodeBench | 57.5% | **65.9%** | 53.8% |
| GPQA Diamond | 65.2% | **71.5%** | 60.0% |
| BFCL (Tool/Function Call) | **63.3%** | 47.4% | 60.7% |
| IFEval (strict) | **83.9%** | -- | -- |
| Arena-Hard | **85.7%** | -- | -- |

**Observaciones clave:**
- Con solo 32B parámetros, QwQ es remarkablemente cercano a DeepSeek-R1 (671B, ~20x más parámetros)
- Supera significativamente a o1-mini en la mayoría de benchmarks
- **Mejor en tool use** (BFCL: 63.3% vs 47.4% de DeepSeek-R1), más práctico como agente
- La tecnología de QwQ fue integrada en Qwen3 como el "thinking mode"

---

## 7. Benchmarks y Rendimiento

### 7.1 Qwen2.5-72B-Instruct (Flagship Open-Weight)

#### Razonamiento y conocimiento general

| Benchmark | Qwen2.5-72B | Llama-3.1-70B | Llama-3.1-405B | GPT-4o |
|-----------|-------------|---------------|----------------|--------|
| MMLU (5-shot) | **86.1%** | 83.6% | 87.3% | 87.2% |
| MMLU-Pro | 71.1% | 66.4% | 73.3% | 72.6% |
| GPQA | 49.0% | 46.7% | 51.1% | 53.6% |
| BBH (3-shot) | **85.3%** | -- | -- | -- |
| IFEval (strict) | **86.8%** | 83.6% | -- | 84.3% |
| Arena-Hard | **81.2%** | 55.7% | -- | -- |
| MT-Bench | ~9.35 | -- | -- | -- |

#### Matemáticas

| Benchmark | Qwen2.5-72B | Llama-3.1-70B | GPT-4o |
|-----------|-------------|---------------|--------|
| GSM8K | 95.8% | 95.1% | 96.1% |
| MATH | **83.1%** | 68.0% | 76.6% |

#### Código

| Benchmark | Qwen2.5-72B | Llama-3.1-70B | GPT-4o |
|-----------|-------------|---------------|--------|
| HumanEval | 86.6% | 80.5% | 90.2% |
| MBPP (3-shot) | **88.2%** | -- | -- |

#### Chino

| Benchmark | Score |
|-----------|-------|
| C-Eval | ~89% |
| CMMLU | ~90% |

### 7.2 Qwen2.5-Max (Flagship Propietario)

| Benchmark | Qwen2.5-Max | GPT-4o | Claude 3.5 Sonnet | DeepSeek-V3 |
|-----------|-------------|--------|-------------------|-------------|
| Arena-Hard | **89.4%** | 87.1% | 85.2% | 85.5% |
| LiveBench | **73.4%** | -- | -- | 71.4% |
| MMLU-Pro | ~73% | ~73% | ~73% | ~72% |

### 7.3 Comparativa de Escalado por Tamaño (MMLU)

| Modelo | MMLU (5-shot) |
|--------|--------------|
| Qwen2.5-72B-Instruct | 86.1% |
| Qwen2.5-32B-Instruct | ~83% |
| Qwen2.5-14B-Instruct | ~79% |
| Qwen2.5-7B-Instruct | ~75% |
| Qwen2.5-3B-Instruct | ~65% |
| Qwen2.5-1.5B-Instruct | ~60% |
| Qwen2.5-0.5B-Instruct | ~47% |

### 7.4 Comparativa de Código (HumanEval)

| Modelo | HumanEval |
|--------|-----------|
| Qwen2.5-Coder-32B-Instruct | 92.7% |
| Qwen2.5-72B-Instruct | 86.6% |
| Qwen2.5-Coder-7B-Instruct | ~83% |
| Qwen2.5-32B-Instruct | ~82% |
| Qwen2.5-7B-Instruct | ~76% |

---

## 8. Licenciamiento

### Evolución de Licencias

| Generación | Licencia | Uso Comercial |
|------------|----------|---------------|
| Qwen1 (2023) | Tongyi Qianwen License | Gratis < 100M MAU; licencia separada arriba |
| Qwen1.5 (2024) | Mixta (CC-BY-NC para algunos) | Variable por modelo |
| Qwen2 (Jun 2024) | Apache 2.0 (mayoría); Qianwen License (72B inicialmente) | Sí (irrestricto para Apache 2.0) |
| Qwen2.5 (Sep 2024) | **Apache 2.0 (todos open-weight)** | Sí (irrestricto) |
| QwQ-32B (Mar 2025) | **Apache 2.0** | Sí |
| Qwen3 (Abr 2025) | **Apache 2.0 (todos open-weight)** | Sí |

### Implicaciones de Apache 2.0

- Uso comercial completo **sin restricciones** (sin límite de MAU)
- Se puede modificar, distribuir y crear obras derivadas
- Debe incluir atribución (notice de copyright)
- Patent grant incluido
- No hay requisito de open-source para obras derivadas (a diferencia de GPL)
- Compatible con la mayoría de requisitos de licenciamiento empresarial

### Modelos Propietarios

Los modelos **Qwen-Turbo, Plus, Max, Long** están disponibles solo a través de la API de Alibaba Cloud, sujetos a los términos de servicio de Alibaba Cloud. Sin acceso a pesos.

---

## 9. Cómo Acceder a Qwen

### 9.1 HuggingFace (Distribución Principal Open-Weight)

**Organización**: [https://huggingface.co/Qwen](https://huggingface.co/Qwen)

Disponible para cada modelo open-weight en formatos: SafeTensors (FP16/BF16), GPTQ-Int4/Int8, AWQ, GGUF.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "Qwen/Qwen2.5-72B-Instruct"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name, device_map="auto", torch_dtype="auto"
)

messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
]
text = tokenizer.apply_chat_template(
    messages, tokenize=False, add_generation_prompt=True
)
inputs = tokenizer(text, return_tensors="pt").to(model.device)
outputs = model.generate(**inputs, max_new_tokens=512)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

### 9.2 Ollama (Inferencia Local)

```bash
ollama run qwen2.5           # Tamaño default (7B)
ollama run qwen2.5:72b       # Modelo 72B
ollama run qwen2.5:0.5b      # Modelo más pequeño
ollama run qwen2.5-coder:32b # Modelo de código
ollama run qwq               # Modelo de razonamiento
ollama run qwen3              # Qwen3
```

### 9.3 vLLM (Inferencia de Alto Rendimiento)

```python
from vllm import LLM, SamplingParams

llm = LLM(model="Qwen/Qwen2.5-72B-Instruct", tensor_parallel_size=4)
sampling_params = SamplingParams(temperature=0.7, max_tokens=1024)
outputs = llm.generate(["What is the meaning of life?"], sampling_params)
```

### 9.4 Alibaba Cloud Model Studio API (DashScope)

**Endpoint**: `https://dashscope.aliyuncs.com/compatible-mode/v1`

Compatible con el cliente de OpenAI Python (drop-in replacement):

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-dashscope-api-key",
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

response = client.chat.completions.create(
    model="qwen-max",  # o qwen-turbo, qwen-plus, qwen-vl-max, etc.
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ]
)
print(response.choices[0].message.content)
```

**SDK nativo DashScope:**

```python
pip install dashscope

import dashscope
from dashscope import Generation

dashscope.api_key = "your-api-key"

response = Generation.call(
    model="qwen-max",
    messages=[{"role": "user", "content": "Hello!"}],
    result_format="message"
)
print(response.output.choices[0].message.content)
```

**Modelos API disponibles:**

| Nombre API | Tipo | Descripción |
|------------|------|-------------|
| `qwen-turbo` | Chat | Rápido, económico |
| `qwen-plus` | Chat | Rendimiento/costo balanceado |
| `qwen-max` | Chat | Máxima calidad |
| `qwen-long` | Chat | Especialista en contexto largo |
| `qwen-vl-max` | Visión+Chat | Mejor modelo de visión |
| `qwen-vl-plus` | Visión+Chat | Modelo de visión mid-tier |
| `qwen-audio-turbo` | Audio+Chat | Comprensión de audio |
| `qwen-math-plus` | Chat (Math) | Especializado en matemáticas |
| `qwen-coder-plus` | Chat (Code) | Especializado en código |
| `text-embedding-v2` | Embedding | Embeddings de texto |
| `wanx-v1` | Image Gen | Text-to-image |
| `paraformer-v2` | ASR | Reconocimiento de habla |
| `cosyvoice-v1` | TTS | Text-to-speech |
| `sensevoice-v1` | ASR | ASR multi-idioma |

**Precios aproximados (inicios 2025, en CNY por 1M tokens):**

| Modelo | Precio Input | Precio Output |
|--------|-------------|---------------|
| qwen-turbo | ~0.3 CNY | ~0.6 CNY |
| qwen-plus | ~0.8 CNY | ~2.0 CNY |
| qwen-max | ~2.0 CNY | ~6.0 CNY |
| qwen-long | ~0.5 CNY | ~2.0 CNY |
| qwen-vl-max | ~3.0 CNY | ~9.0 CNY |

### 9.5 Proveedores Third-Party

| Proveedor | Modelos Disponibles | Notas |
|-----------|-------------------|-------|
| Together AI | Qwen2.5-72B, Coder-32B, QwQ-32B | Precios competitivos |
| Fireworks AI | Qwen2.5-72B, QwQ-32B | Inferencia rápida |
| Groq | Qwen2.5-32B, QwQ-32B | Ultra-rápido (LPU) |
| Amazon Bedrock | Modelos Qwen2.5 | Integración empresarial |
| Google Vertex AI | Modelos Qwen2.5 | Vía Model Garden |
| Azure AI | Modelos Qwen2.5 | Vía model catalog |
| NVIDIA NIM | Modelos Qwen2.5 | Contenedores optimizados |
| OpenRouter | Varios | Agregador de APIs |

### 9.6 Otras Opciones Locales

| Plataforma | Descripción |
|------------|-------------|
| **LM Studio** | App GUI; soporta modelos GGUF de Qwen |
| **GPT4All** | Soporta modelos GGUF de Qwen |
| **text-generation-webui** (oobabooga) | Soporte completo para todos los formatos Qwen |
| **llama.cpp** | Inferencia directa GGUF |
| **SGLang** | Inferencia de alto rendimiento con soporte Qwen |
| **TGI** (HuggingFace) | Deployment basado en Docker |
| **TensorRT-LLM** | Optimizado para NVIDIA |

---

## 10. Integración con Alibaba Cloud

### 10.1 DashScope / Model Studio

Plataforma MaaS (Model-as-a-Service) de Alibaba Cloud.

**Características:**
- API REST y SDK (Python, Java, Node.js)
- Formato API compatible con OpenAI (drop-in replacement)
- Soporta: generación de texto, embeddings, generación de imágenes, síntesis de voz, comprensión de audio, visión-lenguaje
- Function calling / tool use integrado en la API
- Streaming responses
- Batch inference para procesamiento a gran escala
- Upload de archivos para análisis de documentos
- API de Fine-tuning para personalizar modelos

### 10.2 PAI (Platform for AI)

Plataforma comprehensiva de ML de Alibaba Cloud:

| Componente | Descripción |
|------------|-------------|
| **PAI-QuickStart** | Deployment con un clic de modelos Qwen. Soporta LoRA, QLoRA y full fine-tuning. Auto-scaling |
| **PAI-DSW** | Entorno basado en Jupyter con dependencias Qwen pre-instaladas. Instancias GPU (A10, A100, H100) |
| **PAI-DLC** | Soporte de entrenamiento distribuido. Imágenes Docker pre-built. Soporta DeepSpeed, Megatron-LM, FSDP |

### 10.3 EAS (Elastic Algorithm Service)

Parte de PAI para deployment de modelos:
- Deploy Qwen como servicio API escalable
- Auto-scaling basado en volumen de requests
- Múltiples backends de inferencia (vLLM, TGI, custom)
- Tipos de instancia GPU: V100, A10, A100, H100, H800
- Monitoreo y logging integrado
- Blue-green deployment para actualizaciones sin downtime

### 10.4 Otras Integraciones

| Servicio | Uso con Qwen |
|----------|-------------|
| **Function Compute** | Inferencia serverless para modelos pequeños |
| **OSS (Object Storage)** | Almacenamiento y distribución de pesos de modelos |
| **ACK (Kubernetes)** | Orchestrar deployments multi-GPU con tensor parallelism |
| **Tongyi Qianwen App** | Chatbot de consumo en tongyi.aliyun.com (iOS/Android). Chat, generación de imágenes (Tongyi Wanxiang), análisis de documentos, generación de código. 100M+ usuarios |
| **Lingji** | Asistente AI empresarial sobre Qwen. Personalizable por dominio. Integrado con DingTalk |

---

## 11. Qwen-Agent Framework

### 11.1 Visión General

**Qwen-Agent** es un framework open-source para construir agentes de IA potenciados por modelos Qwen.

**GitHub**: [https://github.com/QwenLM/Qwen-Agent](https://github.com/QwenLM/Qwen-Agent)

```
Entrada → Agente (Qwen LLM) → Selección de Herramienta → Ejecución → Respuesta
               ↑                                             |
               |_____________________________________________|
                           (Bucle multi-turno)
```

### 11.2 Herramientas Built-in

| Herramienta | Descripción |
|-------------|-------------|
| **Code Interpreter** | Ejecutar código Python en sandbox (pandas, numpy, matplotlib) |
| **Image Generation** | Generar imágenes vía Tongyi Wanxiang |
| **Web Search** | Buscar en internet |
| **Document Reader** | Parsear documentos (PDF, Word, etc.) |
| **Custom Tools** | Definir herramientas arbitrarias vía function schemas |

### 11.3 Function Calling

Qwen2.5+ soporta function calling compatible con OpenAI:

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {"type": "string", "description": "City name"},
            "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
          },
          "required": ["location"]
        }
      }
    }
  ]
}
```

Soporta **llamadas paralelas de herramientas** (múltiples tools en un turno).

### 11.4 Tipos de Agentes

| Tipo | Descripción |
|------|-------------|
| **Assistant** | Agente de propósito general con tool use |
| **ReAct** | Bucle Reason-then-Act (razonamiento + tool use) |
| **Router** | Despacha tareas a sub-agentes especialistas |
| **DocQA** | Especializado en QA de documentos con RAG |
| **Custom** | Workflows de agente arbitrarios |

### 11.5 Ejemplo de Uso

```python
from qwen_agent.agents import Assistant

bot = Assistant(
    llm={'model': 'qwen-max', 'api_key': 'YOUR_KEY'},
    system_message='You are a helpful assistant.',
    function_list=['code_interpreter', 'image_gen']
)

messages = [{'role': 'user', 'content': 'Plot a sine wave'}]
for response in bot.run(messages):
    print(response)
```

### 11.6 Qwen-VL como Agente Visual

Qwen2.5-VL permite agentes visuales que pueden:
- **Navegar la web**: Entender screenshots, hacer clic, escribir
- **Operar teléfonos**: Entender UI de teléfonos y realizar acciones
- **Automatización de escritorio**: Interactuar con aplicaciones desktop
- **Extracción de datos de imágenes**: Extraer datos estructurados de screenshots, recibos, formularios

---

## 12. Metodología de Entrenamiento

### 12.1 Infraestructura

- Entrenado en **clusters GPU propietarios de Alibaba Cloud**
- Clusters a gran escala de NVIDIA A100 y H100/H800
- Framework de entrenamiento custom basado en **Megatron-LM** y **DeepSpeed** (con modificaciones de Alibaba)
- Entrenamiento distribuido usando 3D parallelism (tensor, pipeline y data parallelism)
- **Pai-Megatron-Patch**: Extensión custom de Alibaba a Megatron

### 12.2 Pipeline de Datos de Pre-entrenamiento

1. **Web crawl**: CommonCrawl y pipelines de crawl propietarios
2. **Libros y papers académicos**: ArXiv, libros de texto, obras de referencia
3. **Código**: Repositorios públicos de GitHub (~92+ lenguajes), StackOverflow, documentación
4. **Multilingüe**: Datos de 29+ idiomas con ratios de sampling basados en calidad
5. **Datos sintéticos**: Datos generados por modelos para dominios especializados

**Procesamiento de datos:**
- Filtrado de calidad basado en reglas y modelos
- Deduplicación near-exact y fuzzy (MinHash, suffix arrays)
- Filtrado de toxicidad y PII
- Clasificación de idiomas y balanced sampling
- Curriculum learning (datos ordenados de fácil a difícil)

### 12.3 Escala de Pre-entrenamiento

| Generación | Tokens de Pre-entrenamiento |
|------------|---------------------------|
| Qwen (2023) | ~3 billones |
| Qwen1.5 (2024) | ~3+ billones |
| Qwen2 (2024) | ~7 billones |
| Qwen2.5 (2024) | ~18 billones |
| Qwen3 (2025) | ~36 billones |

### 12.4 Pipeline de Post-entrenamiento (Qwen2.5+)

#### Etapa 1: Supervised Fine-Tuning (SFT)
- Millones de pares instrucción-respuesta curados
- Datos multi-turno de conversación
- Demostraciones de tool calling
- Ejemplos de salida estructurada (JSON, tablas, código)
- Rejection sampling: Generar múltiples respuestas, quedarse con la mejor

#### Etapa 2: Reward Modeling
- Entrenado en datos de preferencia humana (comparaciones por pares)
- Los modelos aprenden a puntuar respuestas por calidad, utilidad, inofensividad y honestidad

#### Etapa 3: Alineamiento con RL

| Método | Descripción | Uso |
|--------|-------------|-----|
| **PPO** | Proximal Policy Optimization | Versiones tempranas de Qwen |
| **DPO** | Direct Preference Optimization (más estable, no requiere reward model en inferencia) | Qwen2+ |
| **GRPO** | Group Relative Policy Optimization (muestrea grupo de N respuestas, puntúa, optimiza) | QwQ y Qwen3 |

#### Etapa 4: Rejection Sampling Iterativo
- Generar muchos outputs candidatos → puntuar con reward model → reentrenar en los mejores → múltiples rondas

### 12.5 Tracks de Entrenamiento Especializado

| Modelo | Pre-entrenamiento Adicional | SFT Especializado | RL/DPO |
|--------|---------------------------|-------------------|--------|
| Qwen2.5-Coder | ~5.5T tokens de código | Pares de instrucción de código | Reward model de código |
| Qwen2.5-Math | Corpus pesado en matemáticas | Trazas CoT + TIR | RL de resultados matemáticos |
| Qwen2.5-VL | Billones de pares imagen-texto | Visual instruction tuning | Preferencia visual |
| Qwen2-Audio | Pares audio-texto | Audio instruction tuning | Preferencia de audio |
| QwQ-32B | N/A (desde Qwen2.5-32B) | Trazas de razonamiento | GRPO en math/code |

### 12.6 Distribución de Idiomas (aproximada para Qwen2.5)

| Segmento | Proporción |
|----------|-----------|
| Inglés | ~40-50% |
| Chino | ~30-40% |
| Código | ~10-15% |
| Otros idiomas (27+) | ~5-10% |

Qwen3 expandió a **119 idiomas y dialectos** mediante datos web multilingües curados, corpora paralelos y datos de instrucciones multilingües.

---

## 13. Casos de Uso y Aplicaciones

### IA Conversacional y Servicio al Cliente
- Alimenta los bots de servicio al cliente de Alibaba en Taobao, Tmall, AliExpress
- Clientes empresariales despliegan Qwen para soporte en chino
- Chatbot Tongyi Qianwen con 100M+ usuarios

### Generación de Código y Desarrollo de Software
- Qwen2.5-Coder integrado en IDEs vía extensiones
- Completado, generación, review y debugging de código
- 92+ lenguajes de programación
- Competitivo con GitHub Copilot para desarrolladores chinos

### Inteligencia de Documentos
- Qwen-VL para OCR, parsing de documentos, extracción de formularios
- Fuerte en documentos chinos (contratos, facturas, formularios gubernamentales)
- Procesa documentos escaneados, texto manuscrito, tablas y gráficos

### Matemáticas y Educación STEM
- Qwen-Math para resolver problemas paso a paso
- Tool-Integrated Reasoning (genera Python para cálculos)
- Plataformas educativas de tutoría

### Comunicación Multilingüe
- Traducción entre 29+ idiomas (Qwen2.5) o 119 idiomas (Qwen3)
- Especialmente fuerte en chino-inglés y CJK
- E-commerce transfronterizo (AliExpress, Alibaba.com)

### Creación de Contenido y Marketing
- Copy de marketing, descripciones de productos, contenido de redes sociales
- Escritura creativa fuerte en chino
- Resumen de documentos largos

### Aplicaciones Agentic
- Agentes de navegación web (Qwen-VL para comprensión visual)
- Agentes de uso de computadora (interactuar con UIs desktop/móvil)
- Automatización de tareas multi-paso
- Asistentes de investigación basados en RAG

### Audio
- Transcripción y resumen de reuniones
- Reconocimiento de habla multilingüe
- Análisis y clasificación de contenido de audio
- Análisis musical

### IA Edge y Móvil
- Qwen2.5-0.5B/1.5B/3B para deployment móvil
- Asistentes AI on-device
- Aplicaciones IoT
- Inferencia local preservando privacidad

### Empresarial y Financiero
- Sistemas RAG con embeddings Qwen + LLMs Qwen
- QA de base de conocimiento interna
- Análisis de documentos financieros
- Procesamiento de documentos regulatorios

---

## 14. Ecosistema y Comunidad Open-Source

### GitHub Oficial

**Organización**: [https://github.com/QwenLM](https://github.com/QwenLM)

| Repositorio | Descripción |
|-------------|-------------|
| `Qwen2.5` | Documentación principal y ejemplos |
| `Qwen2.5-VL` | Código del modelo visión-lenguaje |
| `Qwen2.5-Coder` | Documentación del modelo de código |
| `Qwen2.5-Math` | Documentación del modelo de matemáticas |
| `Qwen-Agent` | Framework de agentes |
| `Qwen2-Audio` | Código del modelo de audio |

### Plataformas con Soporte Nativo

| Plataforma | Nivel | Caso de Uso |
|------------|-------|-------------|
| Ollama | First-class | Deployment local fácil |
| vLLM | First-class | Serving de alto throughput |
| llama.cpp | First-class (GGUF) | Inferencia CPU/edge |
| SGLang | First-class | Generación estructurada |
| TGI (HuggingFace) | Soportado | Serving de producción |
| TensorRT-LLM | Soportado | Optimizado NVIDIA |
| ModelScope | First-class | Ecosistema chino |

### ModelScope (Plataforma de Alibaba)

[https://modelscope.cn/organization/qwen](https://modelscope.cn/organization/qwen)

- Todos los modelos Qwen duplicados aquí
- Punto de acceso principal para usuarios en China
- Incluye demos, spaces y datasets

### Adopción de la Comunidad

- **Base para fine-tuning**: Modelos Qwen2.5 entre los más populares para fine-tunes comunitarios (junto a Llama y Mistral)
- **Presencia en leaderboards**: Modelos basados en Qwen frecuentemente en el top del Open LLM Leaderboard de HuggingFace
- **Ecosistema AI chino**: Posición dominante en la comunidad open-source china
- **Adopción empresarial**: Usado por numerosas startups y empresas, especialmente en Asia-Pacífico
- **Descargas HuggingFace**: Cientos de millones de descargas acumuladas across all models

---

## 15. Recursos y Enlaces

| Recurso | URL |
|---------|-----|
| Blog Oficial | https://qwenlm.github.io/blog/ |
| GitHub (principal) | https://github.com/QwenLM |
| GitHub (Qwen2.5) | https://github.com/QwenLM/Qwen2.5 |
| GitHub (Qwen-Agent) | https://github.com/QwenLM/Qwen-Agent |
| HuggingFace | https://huggingface.co/Qwen |
| ModelScope | https://modelscope.cn/organization/qwen |
| Alibaba Cloud Model Studio | https://www.alibabacloud.com/en/solutions/generative-ai |
| DashScope API | https://dashscope.aliyuncs.com |
| DashScope Internacional | https://dashscope-intl.aliyuncs.com/compatible-mode/v1 |
| Tongyi Qianwen (consumer app) | https://tongyi.aliyun.com |
| Qwen2 Technical Report (arXiv) | https://arxiv.org/abs/2407.10671 |
| Qwen2.5 Technical Report (arXiv) | https://arxiv.org/abs/2412.15115 |
| QwQ Blog Post | https://qwenlm.github.io/blog/qwq-32b/ |

---

## Ventanas de Contexto - Resumen Completo

| Modelo | Contexto |
|--------|----------|
| Qwen-7B/14B (2023) | 8K (extendible a 32K) |
| Qwen-72B (2023) | 32K nativo |
| Qwen1.5 (todos) | 32K nativo |
| Qwen2-0.5B/1.5B | 32K |
| Qwen2-7B/72B | 128K |
| Qwen2-57B-A14B (MoE) | 64K |
| Qwen2.5-0.5B/1.5B/3B | 32K |
| Qwen2.5-7B/14B/32B/72B | 128K |
| QwQ-32B-Preview | 32K |
| QwQ-32B | 128K |
| Qwen-Turbo (API) | 128K a 1M |
| Qwen-Long (API) | Hasta 1M |

---

> **Nota**: La información contenida en este documento está basada en datos disponibles hasta Mayo 2025. Para las actualizaciones más recientes, consultar los recursos oficiales listados arriba. Los números de benchmarks son aproximados y pueden variar según la configuración de evaluación utilizada. Los precios de la API están sujetos a cambios.