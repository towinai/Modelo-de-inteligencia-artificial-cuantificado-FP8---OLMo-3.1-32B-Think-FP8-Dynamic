# Informe de Evaluación del Modelo

**Modelo evaluado:** `TORQUEMADA-OLMo-3.1-32B-THINKING-FP8-W8A8-DYNAMIC`
**Fecha del informe:** 18 de noviembre de 2026

---

## 1. Resumen de resultados

### 1.1 Calidad de las respuestas

| Métrica | Valor | Evaluación |
|---|---|---|
| Fidelidad media (juez) | 4.33 / 5 | ✅ Buena: respuestas precisas y alineadas con las referencias |
| Relevancia media (juez) | 4.67 / 5 | ✅ Excelente: respuestas pertinentes y contextualizadas |
| Cumplimiento del idioma español | 100 % | ✅ Perfecto: responde en español sin errores |
| Marcador de respuesta | 100 % | ✅ Perfecto: separa correctamente el razonamiento de la respuesta final |
| JSON válido | 100 % | ✅ Perfecto: genera JSON válido con los campos solicitados |

**Conclusión:** El modelo cumple los estándares de calidad de TOWINIA en precisión, relevancia y formato.

### 1.2 Rendimiento

| Métrica | Valor | Evaluación |
|---|---|---|
| Throughput (1 solicitud) | 21.48 tok/s | ✅ Aceptable para una sola solicitud |
| Throughput (2 solicitudes) | 42.48 tok/s | ✅ Buena escalabilidad con 2 solicitudes simultáneas |
| Throughput (4 solicitudes) | 42.54 tok/s | ⚠️ Estancado: no escala linealmente |
| Throughput (8 solicitudes) | 54.43 tok/s | ⚠️ Mejora limitada; el rendimiento por solicitud disminuye notablemente |
| TTFT (1 solicitud) | 130.65 ms | ✅ Excelente: primera respuesta muy rápida |
| TTFT (8 solicitudes) | 20 665.21 ms | ❌ Deficiente: latencia inaceptable con alta concurrencia |

**Conclusión:**
- **Uso individual o baja concurrencia (1–2 solicitudes):** funciona bien.
- **Alta concurrencia (4 o más solicitudes):** no recomendado sin optimizaciones adicionales, como ajustar `max_num_seqs` o `gpu-memory-utilization` en vLLM.

### 1.3 Estabilidad

| Métrica | Valor | Evaluación |
|---|---|---|
| Truncamientos | 0 % | ✅ Sin truncamientos en las respuestas |
| Intentos medios por caso | 1.14 | ✅ Pocos reintentos, solo en casos complejos |
| Errores en peticiones | 0 | ✅ Todas las peticiones se completaron con éxito |

**Conclusión:** El modelo es estable y no presenta fallos en la generación de respuestas.

---

## 2. Puntos débiles y recomendaciones técnicas

### 2.1 Rendimiento en alta concurrencia

- **Problema:** el throughput no escala linealmente con el número de solicitudes simultáneas. Con 8 solicitudes, el throughput agregado es de 54.43 tok/s, frente a los aproximadamente 171.84 tok/s esperables (8 × 21.48).
- **Causas probables:**
  - Límite de `max_num_seqs` en vLLM, actualmente configurado en 3 (ver `docker-compose.yml`).
  - Memoria GPU insuficiente: el modelo FP8 podría estar saturando la GPU.
- **Soluciones propuestas:**
  - Aumentar `max_num_seqs` a 8 o 16.
  - Reducir `gpu-memory-utilization` a 0.85 para evitar cuellos de botella.
  - Probar `tensor-parallel-size=2` en lugar de 4 para reducir el uso de memoria por GPU.

### 2.2 Latencia en alta concurrencia

- **Problema:** el TTFT supera los 20 segundos con 8 solicitudes simultáneas.
- **Causa probable:** cola de peticiones en vLLM, ya que el procesamiento es secuencial debido a `max_num_seqs=3`.
- **Solución:** aumentar `max_num_seqs`. El prefix caching y el chunked prefill ya están activados en la configuración.

### 2.3 Configuración del script de benchmark

- **Problema:** el script se conecta directamente a vLLM en `localhost:8000` y no usa LiteLLM. Las peticiones no quedan registradas en PostgreSQL, por lo que no hay trazabilidad.
- **Solución:**
  - Cambiar el endpoint a `http://localhost:4000/v1` (LiteLLM).
  - Configurar la clave de API de LiteLLM en la variable `LLM_API_KEY`.

> **Nota de seguridad:** el informe original incluía una clave de API en texto plano. Se recomienda rotarla y no compartirla en documentos públicos ni repositorios.

---

## 3. Conclusión final: ¿vale la pena publicarlo?

**Sí, con matices.**

| Criterio | ¿Cumple? | Notas |
|---|---|---|
| Calidad de las respuestas | ✅ Sí | Fidelidad y relevancia altas, formato correcto |
| Idioma (español) | ✅ Sí | 100 % de cumplimiento |
| Estabilidad | ✅ Sí | Sin errores ni truncamientos |
| Rendimiento (baja concurrencia) | ✅ Sí | 21–42 tok/s, aceptable para uso individual |
| Rendimiento (alta concurrencia) | ❌ No | Throughput bajo y latencia alta con 4 o más solicitudes |
| Trazabilidad (logs) | ❌ No | El script no usa LiteLLM, por lo que no hay registros en PostgreSQL |

### 3.1 Recomendaciones para la publicación

1. **Hugging Face:** publicable con advertencias sobre el rendimiento en alta concurrencia. Incluir en la descripción:
   - Benchmarks de rendimiento (21–42 tok/s).
   - Ejemplos de respuestas en español.
   - Configuración recomendada para vLLM.
2. **GitHub:** publicable con el script de benchmark actualizado para usar LiteLLM (`localhost:4000`), e instrucciones claras para configurar `LLM_API_KEY`.
3. **Mejoras previas a la publicación:**
   - Optimizar vLLM para alta concurrencia (aumentar `max_num_seqs`).
   - Probar `tensor-parallel-size=2` para evaluar la mejora del throughput.
   - Añadir pruebas con casos de uso real de TOWINIA.

### 3.2 Resumen ejecutivo

| Aspecto | Valoración | Acción recomendada |
|---|---|---|
| Calidad | ⭐⭐⭐⭐⭐ (5/5) | Publicar sin cambios |
| Rendimiento (1–2 solicitudes) | ⭐⭐⭐⭐ (4/5) | Publicar con advertencia |
| Rendimiento (4 o más solicitudes) | ⭐⭐ (2/5) | Optimizar vLLM antes de publicar |
| Estabilidad | ⭐⭐⭐⭐⭐ (5/5) | Publicar sin cambios |
| Trazabilidad | ⭐ (1/5) | Usar LiteLLM en el script |

### 3.3 Veredicto final

**"Publicable con optimizaciones"**

- **Uso interno en TOWINIA:** totalmente recomendado, dada la calidad y estabilidad excelentes.
- **Publicación pública (Hugging Face / GitHub):** sí, pero con advertencias sobre el rendimiento en alta concurrencia. Se recomienda optimizar vLLM antes.
