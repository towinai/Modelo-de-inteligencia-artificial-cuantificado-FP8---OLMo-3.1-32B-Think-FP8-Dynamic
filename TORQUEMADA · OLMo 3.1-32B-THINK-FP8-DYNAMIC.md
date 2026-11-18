# Ficha de Modelo: TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC

## Resumen

**TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC** es una variante cuantizada del modelo **OLMo 3.1 32B Thinking**, optimizada para inferencia eficiente mediante cuantización **FP8 Dynamic W8A8**. Está orientada a generación de texto en español, razonamiento guiado, cumplimiento de instrucciones y salida estructurada en JSON.

El modelo se sirve mediante **vLLM** con una API compatible con OpenAI y está pensado para entornos de inferencia con GPU NVIDIA. La variante FP8 Dynamic evita la calibración manual y permite reducir el uso de memoria frente a configuraciones de mayor precisión, manteniendo un buen nivel de calidad en las pruebas realizadas.

---

## Identidad del modelo

| Campo | Valor |
|---|---|
| **Nombre público** | `TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC` |
| **Alias de servicio** | `TORQUEMADA-FP8` |
| **Modelo base** | OLMo 3.1 32B Thinking |
| **Tipo de modelo** | Modelo de lenguaje autorregresivo |
| **Cuantización** | FP8 Dynamic W8A8 |
| **Formato de pesos** | Safetensors |
| **Contexto máximo configurado** | 65.536 tokens |
| **Idioma principal** | Español |
| **Licencia** | La del modelo base OLMo 3.1 32B. Revisar términos aplicables antes de su redistribución. |
| **Estado** | Publicable con advertencias de rendimiento en alta concurrencia |

---

## Capacidades principales

- Generación de respuestas en español con alto cumplimiento del idioma.
- Razonamiento explícito con separación entre razonamiento y respuesta final mediante marcadores configurables.
- Generación de **JSON válido** cuando se solicita salida estructurada.
- Cumplimiento de instrucciones y formato.
- Soporte de contexto largo: hasta **65.536 tokens** en la configuración de referencia.
- Compatible con API OpenAI mediante vLLM.
- Adecuado para tareas de análisis, resumen, clasificación, generación de informes y asistentes conversacionales en español.

---

## Uso previsto

El modelo está previsto para:

- Asistentes conversacionales en español.
- Análisis y resumen de documentación técnica o normativa.
- Generación de respuestas estructuradas en JSON.
- Tareas de apoyo a la decisión con supervisión humana.
- Prototipos y entornos de baja concurrencia.

Ejemplos de uso validados en pruebas internas:

- Consultas sobre RGPD y minimización de datos.
- Análisis básico de logs y clasificación de incidentes NIS2.
- Generación de objetos JSON con campos definidos.

---

## Fuera de alcance y limitaciones

- **No recomendado para alta concurrencia sin optimización previa.** Con la configuración de referencia, el rendimiento se degrada a partir de 4 solicitudes simultáneas.
- **No sustituye la validación humana** en contextos legales, regulatorios, médicos, financieros o de seguridad.
- **No se han evaluado sesgos, toxicidad o seguridad** de forma exhaustiva.
- **Riesgo de alucinación**: como todo modelo de lenguaje, puede generar información incorrecta o no verificada.
- **La cuantización FP8 puede introducir ruido numérico** en generaciones muy largas o en configuraciones agresivas de caché KV.
- **La licencia y condiciones de redistribución** deben confirmarse con el modelo base antes de publicación definitiva.
- El benchmark realizado es limitado: **7 casos evaluados**, con juez automático. No constituye una evaluación exhaustiva.

---

## Entorno de referencia

Configuración orientativa utilizada para las pruebas:

| Componente | Especificación |
|---|---|
| **GPU** | 4× NVIDIA RTX 2000 Ada Generation, 16 GB GDDR6 por GPU, 64 GB VRAM totales |
| **CPU** | 8 vCPUs sobre AMD EPYC 7702P |
| **RAM** | 20 GiB DDR4 ECC |
| **Almacenamiento** | 110 GiB NVMe |
| **Software de inferencia** | vLLM con API compatible OpenAI |
| **Cuantización** | FP8 Dynamic W8A8 |
| **Caché KV** | FP8 E4M3 en la configuración de referencia |
| **Paralelismo tensorial** | 4 GPUs |
| **Memoria GPU objetivo** | 0.92 de utilización |
| **Máximo de secuencias** | 3 en la configuración de referencia |
| **Máximo de tokens por lote** | 8.192 |
| **Tamaño de bloque** | 32 |
| **Prefix caching** | Activado |
| **Chunked prefill** | Activado |

> Nota: estos valores son orientativos y pueden requerir ajuste según el hardware, la carga y los requisitos de latencia.

---

## Evaluación de calidad

Resultados de la auditoría interna con **7 casos evaluados**:

| Métrica | Resultado |
|---|---|
| **Casos evaluados** | 7 |
| **Truncados** | 0% |
| **Marcador de respuesta correcto** | 100% |
| **JSON válido** | 100% en los casos JSON evaluados |
| **Campos JSON completos** | 100% |
| **Cumplimiento del español** | 100% |
| **Intentos medios por caso** | 1,14 |
| **Juez válido** | 100% |
| **Fidelidad media del juez** | 4,33 / 5 |
| **Relevancia media del juez** | 4,67 / 5 |
| **Razonamiento medio** | 6.254,7 caracteres |
| **Respuesta media** | 270,1 caracteres |

### Valoración

- **Calidad de respuesta:** buena, con alta relevancia y fidelidad.
- **Formato:** excelente cumplimiento de JSON y marcadores de respuesta.
- **Idioma:** cumplimiento del español del 100% en la muestra evaluada.
- **Estabilidad:** sin truncamientos ni errores de petición en la ejecución válida.

---

## Rendimiento en estrés

Pruebas de concurrencia con la configuración de referencia:

| Concurrencia | Throughput agregado | TTFT |
|---:|---:|---:|
| **1 solicitud** | 21,48 tok/s | 130,65 ms |
| **2 solicitudes** | 42,48 tok/s | 111,83 ms |
| **4 solicitudes** | 42,54 tok/s | 6.272,51 ms |
| **8 solicitudes** | 54,43 tok/s | 20.665,21 ms |

### Interpretación

- **1–2 solicitudes simultáneas:** rendimiento aceptable para uso individual o baja concurrencia.
- **4 solicitudes:** el throughput agregado se estanca y la latencia de primera respuesta aumenta de forma significativa.
- **8 solicitudes:** la latencia se degrada críticamente; no se recomienda esta configuración sin ajustes.

### Recomendaciones de optimización

Para cargas concurrentes se recomienda:

- Aumentar `max_num_seqs` según la memoria disponible.
- Ajustar `gpu_memory_utilization`.
- Validar configuraciones alternativas de paralelismo tensorial.
- Monitorizar latencia, uso de VRAM y longitud de cola.
- Realizar pruebas de carga antes de un despliegue en producción.

---

## Ejemplo de uso

El modelo expone una API compatible con OpenAI. Sustituye las variables de entorno por tus propios valores; **no incluyas claves en repositorios públicos**.

```bash
curl -s "$ENDPOINT/v1/chat/completions" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC",
    "messages": [
      {
        "role": "system",
        "content": "Eres un asistente experto en normativa y análisis técnico. Responde en español."
      },
      {
        "role": "user",
        "content": "Explica brevemente qué es el principio de minimización de datos del RGPD."
      }
    ],
    "max_tokens": 512,
    "temperature": 0.7,
    "min_p": 0.1,
    "repetition_penalty": 1.1
  }'