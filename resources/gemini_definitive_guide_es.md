# Guía de Modelos Gemini y Patrones Enterprise

Esta guía consolida los aprendizajes de los reportes técnicos oficiales y las mejores prácticas de implementación.

## 1. Modelos Gemini Disponibles en Vertex AI

### Gemini 2.5 (Estable)
- **`gemini-2.5-flash`**: Optimizado para tareas de alta frecuencia. Baja latencia, ideal para producción.
- **`gemini-2.5-pro`**: Ventana de contexto de hasta 1M de tokens. Procesamiento de video (3h), audio largo y PDFs complejos.

### Gemini 3 (Preview)
- **`gemini-3.1-pro-preview`**: El modelo más avanzado de Google en razonamiento. Doble rendimiento en benchmarks de razonamiento (`ARC-AGI-2`) vs la serie 3.0.
- **Pensamiento Dinámico**: Niveles de pensamiento controlables para equilibrar calidad y costo.

> **Nota**: Los modelos preview (`-preview`) pueden cambiar sin previo aviso. Para producción, usar las versiones estables (2.5).

### Embeddings
- **`gemini-embedding-001`**: 3072 dimensiones. Recomendado para búsqueda semántica y RAG.
- **`multimodal-embedding-001`**: Embeddings combinados de texto + imagen.

---

## 2. Parámetros Críticos

### Configuración de Pensamiento (`thinking_config`)
Permite controlar qué tanto "reflexiona" el modelo antes de responder.
- `minimal`: Baja latencia, ideal para chats rápidos.
- `low`: Equilibrio para seguimiento de instrucciones simples.
- `medium`: Estándar para la mayoría de las tareas.
- `high`: Razonamiento profundo (default para Gemini 3.1 en tareas complejas).

```python
config = GenerateContentConfig(
    thinking_config={"thinking_level": "high"}
)
```

### Resolución de Medios (`media_resolution`)
Crucial para búsquedas multimodales en videos y audios largos.
- Usa resoluciones bajas para escaneo rápido de eventos en videos de 1 hora o más.

---

## 3. Patrones Enterprise

### Context Caching
Para optimizar costos en ventanas de contexto gigantes (1M+ tokens).
- **Caso de uso**: Bases de código completas o bibliotecas de documentos PDF que se consultan repetidamente.

### RAG Multimodal
Flujo completo para documentos con contenido mixto:
1. **Extracción**: Usa `PyMuPDF` para separar texto de imágenes/tablas del PDF.
2. **Descripción**: Gemini genera descripciones textuales de cada imagen/tabla extraída.
3. **Embeddings**: Genera embeddings multimodales para indexar tanto el contenido visual como el textual.
4. **Etiquetado**: En el prompt, etiqueta las opciones (ej: "Imagen 1: Gráfico de ventas") para reducir alucinaciones visuales.

### Salida Estructurada (JSON)
Para integrar Gemini en flujos automatizados:
```python
config = GenerateContentConfig(
    response_mime_type="application/json"
)
```

---

## 4. Mejores Prácticas de Búsqueda
- **Ranking RRF**: Combina búsqueda semántica y por tokens con `rrf_ranking_alpha=0.5`.
- **Reasoning**: Para búsquedas con lógica compleja (ej: discrepancias en contratos), usa `gemini-3.1-pro-preview` con `thinking_level="high"`.
- **Temperatura baja (0.2)**: Para comparación de imágenes y extracción precisa de datos.
- **Temperatura media (0.4-0.6)**: Para descripciones creativas de video o generación de contenido.
