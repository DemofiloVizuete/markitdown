# MarkItDown OCR Plugin

Plugin de LLM Vision para MarkItDown que extrae texto de imágenes incrustadas en archivos PDF, DOCX, PPTX y XLSX.

Usa el mismo patrón `llm_client` / `llm_model` que MarkItDown ya admite para descripciones de imágenes, sin requerir nuevas bibliotecas de ML ni dependencias binarias.

## Características

- **Convertidor PDF mejorado**: Extrae texto de imágenes dentro de PDFs, con fallback de OCR de página completa para documentos escaneados
- **Convertidor DOCX mejorado**: OCR para imágenes en documentos Word
- **Convertidor PPTX mejorado**: OCR para imágenes en presentaciones PowerPoint
- **Convertidor XLSX mejorado**: OCR para imágenes en hojas de cálculo Excel
- **Preservación de contexto**: Mantiene la estructura y el flujo del documento al insertar el texto extraído

## Instalación

```bash
pip install markitdown-ocr
```

El plugin usa cualquier cliente compatible con OpenAI que ya tengas. Instala uno si aún no lo tienes:

```bash
pip install openai
```

## Uso

### Línea de comandos

```bash
markitdown document.pdf --use-plugins --llm-client openai --llm-model gpt-4o
```

### API de Python

Pasa `llm_client` y `llm_model` a `MarkItDown()` exactamente igual que harías para descripciones de imágenes:

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)

result = md.convert("document_with_images.pdf")
print(result.text_content)
```

Si no se proporciona `llm_client`, el plugin igualmente se carga, pero el OCR se omite de forma silenciosa y se usa el convertidor estándar integrado.

### Prompt personalizado

Sobrescribe el prompt de extracción por defecto para documentos especializados:

```python
md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
    llm_prompt="Extract all text from this image, preserving table structure.",
)
```

### Cualquier cliente compatible con OpenAI

Funciona con cualquier cliente que siga la API de OpenAI:

```python
from openai import AzureOpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=AzureOpenAI(
        api_key="...",
        azure_endpoint="https://your-resource.openai.azure.com/",
        api_version="2024-02-01",
    ),
    llm_model="gpt-4o",
)
```

## Cómo funciona

Cuando se llama a `MarkItDown(enable_plugins=True, llm_client=..., llm_model=...)`:

1. MarkItDown detecta el plugin mediante el grupo de entry points `markitdown.plugin`
2. Llama a `register_converters()`, reenviando todos los kwargs, incluidos `llm_client` y `llm_model`
3. El plugin crea un `LLMVisionOCRService` a partir de esos kwargs
4. Se registran cuatro convertidores mejorados con OCR con **prioridad -1.0**, antes de los convertidores integrados con prioridad 0.0

Cuando se convierte un archivo:

1. El convertidor OCR acepta el archivo
2. Extrae las imágenes incrustadas del documento
3. Cada imagen se envía al LLM con un prompt de extracción
4. El texto devuelto se inserta en línea, preservando la estructura del documento
5. Si falla la llamada al LLM, la conversión continúa sin el texto de esa imagen

## Formatos de archivo compatibles

### PDF

- Las imágenes incrustadas se extraen por posición (mediante `page.images` / XObjects de página) y se aplican OCR en línea, intercaladas con el texto circundante en orden vertical de lectura.
- Los **PDF escaneados** (páginas sin texto extraíble) se detectan automáticamente: cada página se renderiza a 300 DPI y se envía al LLM como imagen de página completa.
- Los **PDF malformados** que pdfplumber/pdfminer no pueden abrir (por ejemplo, EOF truncado) se reintentan con renderizado de página de PyMuPDF, para recuperar contenido igualmente.

### DOCX

- Las imágenes se extraen mediante relaciones de partes del documento (`doc.part.rels`).
- El OCR se ejecuta antes del pipeline DOCX→HTML→Markdown: se inyectan tokens de marcador de posición en el HTML para que el convertidor markdown no escape los marcadores OCR, y al final esos placeholders se reemplazan por bloques con formato `*[Image OCR]...[End OCR]*` tras la conversión.
- El flujo del documento (encabezados, párrafos, tablas) se preserva completamente alrededor de los bloques OCR.

### PPTX

- Se admiten formas de imagen, placeholders con imágenes e imágenes dentro de grupos.
- Las formas se procesan por diapositiva en orden de lectura de arriba a la izquierda.
- Si hay un `llm_client` configurado, primero se solicita una descripción al LLM; OCR se usa como fallback cuando no se devuelve descripción.

### XLSX

- Las imágenes incrustadas en hojas (`sheet._images`) se extraen por cada hoja.
- La posición de celda se calcula a partir de las coordenadas del anclaje de imagen (columna/fila → notación de letras de Excel).
- Las imágenes se listan bajo una sección `### Images in this sheet:` después de la tabla de datos de la hoja; no se intercalan en las filas de la tabla.

### Formato de salida

Cada bloque OCR extraído se envuelve como:

```text
*[Image OCR]
<extracted text>
[End OCR]*
```

## Solución de problemas

### Falta texto OCR en la salida

La causa más probable es que falte `llm_client` o `llm_model`. Verifica:

```python
from openai import OpenAI
from markitdown import MarkItDown

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),   # required
    llm_model="gpt-4o",    # required
)
```

### El plugin no carga

Confirma que el plugin está instalado y detectado:

```bash
markitdown --list-plugins   # should show: ocr
```

### Errores de API

El plugin propaga los errores de la API del LLM como advertencias y continúa la conversión. Comprueba tu API key, cuota y que el modelo elegido soporte entradas de visión.

## Desarrollo

### Ejecutar pruebas

```bash
cd packages/markitdown-ocr
pytest tests/ -v
```

### Compilar desde código fuente

```bash
git clone https://github.com/microsoft/markitdown.git
cd markitdown/packages/markitdown-ocr
pip install -e .
```

## Contribuir

Las contribuciones son bienvenidas. Consulta el [repositorio de MarkItDown](https://github.com/microsoft/markitdown) para ver las guías.

## Licencia

MIT. Consulta [LICENSE](LICENSE).

## Registro de cambios

### 0.1.0 (Versión inicial)

- OCR con LLM Vision para PDF, DOCX, PPTX y XLSX
- Fallback de OCR de página completa para PDFs escaneados
- Inserción de texto en línea con preservación de contexto
- Reemplazo de convertidores basado en prioridad (sin cambios de código requeridos)
