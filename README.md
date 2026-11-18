# TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![vLLM](https://img.shields.io/badge/vLLM-0.33.0-green.svg)](https://github.com/vllm-project/vllm)
[![LiteLLM](https://img.shields.io/badge/LiteLLM-main--latest-orange.svg)](https://github.com/BerriAI/litellm)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791.svg)](https://www.postgresql.org/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

Variante cuantizada en **FP8 Dynamic W8A8** del modelo **[OLMo 3.1 32B Think](https://huggingface.co/allenai/OLMo-3.1-32B-Think)**, servida con **vLLM** detrás de una pasarela **LiteLLM** con trazabilidad en **PostgreSQL**. Está orientada a generación de texto en español, razonamiento guiado, cumplimiento de instrucciones y salida estructurada en JSON.

> ⚠️ **Estado:** publicable con advertencias. Optimizado para **uso individual o baja concurrencia (1–2 solicitudes simultáneas)**. Para alta concurrencia, ajusta la configuración de vLLM.

---

## Tabla de contenidos

- [Características](#características)
- [Arquitectura](#arquitectura)
- [Requisitos previos](#requisitos-previos)
- [Instalación](#instalación)
- [Uso](#uso)
- [Rendimiento](#rendimiento)
- [Evaluación de calidad](#evaluación-de-calidad)
- [Limitaciones](#limitaciones)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Contribución](#contribución)
- [Licencia](#licencia)
- [Contacto](#contacto)
- [Agradecimientos](#agradecimientos)
- [Cómo citar](#cómo-citar)

---

## Características

| Característica | Detalle |
|----------------|---------|
| **Cuantización** | FP8 Dynamic W8A8 (sin calibración manual) |
| **Contexto** | Hasta 65.536 tokens |
| **Idioma** | Español (100 % de cumplimiento en pruebas) |
| **Salida estructurada** | JSON válido (100 % en los casos evaluados) |
| **Razonamiento** | Separación razonamiento/respuesta mediante marcadores configurables |
| **API** | Compatible con OpenAI, expuesta a través de LiteLLM |
| **Trazabilidad** | Registro de peticiones en PostgreSQL |
| **Optimizaciones** | Prefix caching, chunked prefill, caché KV en FP8 E4M3 |

---

## Arquitectura

```
Cliente (SDK OpenAI / curl)
        │
        ▼
LiteLLM Proxy  :4000   ──►  PostgreSQL 16 (logs y trazabilidad)
        │
        ▼
vLLM (OpenAI API)  :8000
        │
        ▼
OLMo 3.1 32B Think · FP8 Dynamic W8A8 · 4 × GPU (tensor parallel)
```

Los clientes se conectan siempre a LiteLLM, que registra cada petición en PostgreSQL y la reenvía a vLLM. Conectar directamente a vLLM evita la trazabilidad y no es recomendable en entornos compartidos.

---

## Requisitos previos

| Componente | Requisito recomendado |
|------------|----------------------|
| **GPU** | NVIDIA con soporte FP8 (probado en 4× RTX 2000 Ada, 64 GB de VRAM en total) |
| **VRAM** | ≥ 32 GB (pesos FP8 y caché KV, según contexto y concurrencia) |
| **CPU** | 8 vCPUs |
| **RAM** | 20 GiB |
| **Almacenamiento** | ≥ 110 GiB NVMe |
| **Sistema operativo** | Ubuntu 24.04 LTS |
| **Docker** | 24 o superior, con Compose v2 |
| **NVIDIA Container Toolkit** | Instalado y configurado |
| **vLLM** | `vllm/vllm-openai:v0.33.0` |
| **LiteLLM** | Proxy desde `main-latest` |
| **PostgreSQL** | 16 |

---

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/towinai/torquemada-olmo31-fp8.git
cd torquemada-olmo31-fp8
```

### 2. Descargar los pesos del modelo

Los pesos cuantizados se publican en Hugging Face: [TOWIN-AI/OLMo-3.1-32B-THINK-FP8-DYNAMIC](https://huggingface.co/TOWIN-AI/OLMo-3.1-32B-THINK-FP8-DYNAMIC).

```bash
huggingface-cli download TOWIN-AI/OLMo-3.1-32B-THINK-FP8-DYNAMIC \
  --local-dir ./models/OLMo-3.1-32B-Think-FP8-W8A8-dynamic
```

### 3. Configurar variables de entorno

Crea un archivo `.env` a partir de `.env.example`. **No lo subas nunca al repositorio.**

```ini
VLLM_API_KEY=<clave-vllm>
LITELLM_MASTER_KEY=<clave-litellm>
POSTGRES_PASSWORD=<contraseña-postgres>
MODEL_PATH=./models/OLMo-3.1-32B-Think-FP8-W8A8-dynamic
SERVED_MODEL_NAME=TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC
```

```bash
chmod 600 .env
```

### 4. Levantar los servicios

```bash
docker compose up -d
docker compose ps
docker compose logs -f vllm
```

Compose levanta tres servicios: `vllm` (puerto 8000), `litellm` (puerto 4000) y `postgres` (versión 16).

### 5. Verificar el servicio

```bash
# Modelos disponibles a través de LiteLLM
curl -s http://localhost:4000/v1/models \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

---

## Uso

Los clientes deben apuntar a LiteLLM (`:4000`), no a vLLM directamente.

### Cliente Python (SDK de OpenAI)

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:4000/v1",
    api_key=os.environ["LITELLM_MASTER_KEY"],
)

resp = client.chat.completions.create(
    model="TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC",
    messages=[
        {"role": "system", "content": "Responde en español."},
        {"role": "user", "content": "Resume en dos líneas qué es NIS2."},
    ],
    max_tokens=512,
    temperature=0.7,
    extra_body={"min_p": 0.1, "repetition_penalty": 1.1},
)

print(resp.choices[0].message.content)
```

### Parámetros recomendados

| Parámetro | Valor sugerido | Motivo |
|-----------|----------------|--------|
| `temperature` | 0.1–0.7 | Precisión (bajo) / creatividad (alto) |
| `min_p` | 0.1 | Reduce bucles |
| `repetition_penalty` | 1.1 | Evita repeticiones |
| `top_p` | 0.9–0.95 | Muestreo nucleus estándar |
| `max_tokens` | Según la tarea | Ajustar al caso de uso |

### Ejemplo: salida JSON

```bash
curl -s http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC",
    "messages": [
      {"role": "system", "content": "Devuelve ÚNICAMENTE un objeto JSON válido, sin texto adicional."},
      {"role": "user", "content": "Genera un JSON con los campos id (entero), estado (string: \"ok\") y riesgo (string: \"bajo\")."}
    ],
    "max_tokens": 256,
    "temperature": 0.1,
    "min_p": 0.1
  }'
```

---

## Rendimiento

Pruebas de concurrencia con la configuración de referencia:

| Concurrencia | Throughput agregado | TTFT |
|--------------|---------------------|------|
| 1 solicitud | 21,48 tok/s | 130,65 ms |
| 2 solicitudes | 42,48 tok/s | 111,83 ms |
| 4 solicitudes | 42,54 tok/s | 6.272,51 ms |
| 8 solicitudes | 54,43 tok/s | 20.665,21 ms |

> ⚠️ Para 4 o más solicitudes, optimiza `--max-num-seqs`, `--gpu-memory-utilization` y `--tensor-parallel-size` en vLLM, y repite la prueba de carga antes de un despliegue en producción.

---

## Evaluación de calidad

Auditoría interna con **7 casos** y juez automático:

| Métrica | Resultado |
|---------|-----------|
| Truncados | 0 % |
| Marcador de respuesta correcto | 100 % |
| JSON válido (casos JSON) | 100 % |
| Cumplimiento del español | 100 % |
| Intentos medios por caso | 1,14 |
| Fidelidad media del juez | 4,33 / 5 |
| Relevancia media del juez | 4,67 / 5 |

> ⚠️ Es una muestra pequeña con juez automático. No constituye una evaluación exhaustiva.

---

## Limitaciones

- **Alta concurrencia:** no recomendado para 4 o más solicitudes sin optimización.
- **Alucinaciones:** verifica siempre la información generada.
- **Sesgos y toxicidad:** no evaluados de forma exhaustiva.
- **Cuantización FP8:** puede introducir ruido en generaciones largas.
- **Evaluación limitada:** 7 casos con juez automático.

---

## Estructura del repositorio

```
.
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── docker-compose.yml
├── .env.example
├── litellm/
│   └── config.yaml
├── scripts/
│   ├── benchmark.py
│   └── quantize_fp8_dynamic.py
├── docs/
│   ├── deployment.md
│   └── model_card.md
└── examples/
    ├── chat_completion.py
    └── json_generation.py
```

---

## Contribución

Las contribuciones son bienvenidas. Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para el flujo completo. En resumen:

1. Haz un **fork** del repositorio.
2. Crea una rama: `git checkout -b feature/mi-mejora`.
3. Realiza tus cambios y añade pruebas si procede.
4. Abre un **Pull Request** describiendo el cambio y su motivación.

Para reportar errores, abre un **Issue** con la versión de vLLM, la configuración de hardware, los logs relevantes (sin credenciales) y los pasos para reproducirlo.

> 🔒 No incluyas claves API, contraseñas ni datos personales en issues ni en PRs.

---

## Licencia

Distribuido bajo la **licencia Apache 2.0**. Consulta [LICENSE](LICENSE).

---

## Contacto

**TOWIN AI**

- Correo: [info@towinia.es](mailto:info@towinia.es)
- GitHub: [towinai](https://github.com/towinai)
- Hugging Face: [TOWIN-AI](https://huggingface.co/TOWIN-AI)

---

## Agradecimientos

- **Allen Institute for AI (AI2)** por el modelo base OLMo 3.1 32B.
- **vLLM** por el motor de inferencia.
- **LiteLLM** por la pasarela de API.

---

## Cómo citar

```bibtex
@misc{towinai2026torquemada,
  author       = {{TOWIN AI}},
  title        = {TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC},
  year         = {2026},
  howpublished = {GitHub / Hugging Face},
  note         = {Variante cuantizada FP8 Dynamic W8A8 de OLMo 3.1 32B Think}
}
```
