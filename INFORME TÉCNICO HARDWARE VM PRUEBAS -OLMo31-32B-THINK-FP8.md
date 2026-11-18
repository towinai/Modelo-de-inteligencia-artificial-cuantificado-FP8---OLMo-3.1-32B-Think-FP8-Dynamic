### Entorno de Ejecución e Infraestructura (IA)

| Componente | Especificación Técnica |
| :--- | :--- |
| **Aceleración Gráfica (GPU)** | 4× NVIDIA RTX 2000 Ada Generation (16 GB GDDR6 por GPU — **64 GB VRAM totales**) mediante PCI Passthrough directo |
| **Procesador (CPU)** | 8 vCPUs dedicadas con acceso directo a instrucciones nativas sobre arquitectura **AMD EPYC 7702P** (Zen 2, 2.0 GHz base / hasta 3.35 GHz boost, caché L3 de 128 MB por socket) |
| **Memoria RAM** | 20 GiB DDR4 ECC a 3200 MHz |
| **Almacenamiento** | 110 GiB en unidad NVMe de alto rendimiento mediante interfaz VirtIO SCSI |
| **Interfaz de Red** | Adaptador virtualizado de alto rendimiento (VirtIO) respaldado por conectividad redundante a 10 Gbps |
| **Arquitectura de Virtualización** | Entorno virtualizado bajo arquitectura `q35` con firmware UEFI (OVMF) y soporte PCIe 4.0 |

### Resumen Destacado para Publicación

* **Capacidad de cómputo gráfico:** 64 GB de VRAM GDDR6 dedicados (4× NVIDIA RTX 2000 Ada de 16 GB) con aislamiento directo por hardware (PCI Passthrough), optimizados para inferencia de modelos de lenguaje de gran tamaño (32B parámetros en cuantización INT8 / W8A8).
* **Procesamiento y memoria:** 8 núcleos físicos dedicados de arquitectura AMD EPYC (Zen 2) con soporte completo de instrucciones vectoriales nativas y 20 GiB de RAM DDR4 ECC a 3200 MHz.
* **Almacenamiento de baja latencia:** Unidad NVMe empresarial de 110 GiB para carga rápida de pesos del modelo y gestión de contexto[cite: 3, 4].