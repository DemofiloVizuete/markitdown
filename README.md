# MarkItDown

[![PyPI](https://img.shields.io/pypi/v/markitdown.svg)](https://pypi.org/project/markitdown/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)

> [!TIP]
> MarkItDown ahora ofrece un servidor MCP (Model Context Protocol) para integrarse con aplicaciones de LLM como Claude Desktop. Consulta [markitdown-mcp](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp) para más información.

> [!IMPORTANT]
> Cambios incompatibles entre 0.0.1 y 0.1.0:
> * Las dependencias ahora se organizan en grupos de funcionalidades opcionales (más detalles abajo). Usa `pip install 'markitdown[all]'` para mantener un comportamiento compatible con versiones anteriores.
> * convert\_stream() ahora requiere un objeto tipo archivo binario (por ejemplo, un archivo abierto en modo binario o un objeto io.BytesIO). Este es un cambio incompatible respecto a la versión anterior, donde también aceptaba objetos tipo archivo de texto, como io.StringIO.
> * La interfaz de la clase DocumentConverter cambió para leer desde flujos tipo archivo en lugar de rutas de archivo. *Ya no se crean archivos temporales*. Si mantienes un plugin o un DocumentConverter personalizado, probablemente necesites actualizar tu código. Si solo usas la clase MarkItDown o la CLI (como en estos ejemplos), no deberías tener que cambiar nada.

MarkItDown es una utilidad ligera de Python para convertir distintos tipos de archivos a Markdown para su uso con LLM y flujos relacionados de análisis de texto. En este sentido, se parece a [textract](https://github.com/deanmalmgren/textract), pero con enfoque en conservar la estructura y el contenido importantes del documento en Markdown (incluyendo: encabezados, listas, tablas, enlaces, etc.). Aunque la salida suele ser razonablemente presentable y legible para personas, está pensada para ser consumida por herramientas de análisis de texto, por lo que puede no ser la mejor opción para conversiones de alta fidelidad orientadas a lectura humana.

Actualmente, MarkItDown admite conversión desde:

- PDF
- PowerPoint
- Word
- Excel
- Imágenes (metadatos EXIF y OCR)
- Audio (metadatos EXIF y transcripción de voz)
- HTML
- Formatos basados en texto (CSV, JSON, XML)
- Archivos ZIP (itera sobre su contenido)
- URLs de YouTube
- EPubs
- ... y más

## ¿Por qué Markdown?

Markdown es extremadamente cercano al texto plano, con marcado o formato mínimos, pero aun así
proporciona una forma de representar estructuras importantes de documentos. Los LLM más usados,
como GPT-4o de OpenAI, "_hablan_" Markdown de forma nativa y con frecuencia incorporan Markdown
en sus respuestas sin que se les pida. Esto sugiere que han sido entrenados con enormes cantidades
de texto en formato Markdown y lo entienden bien. Como beneficio adicional, las convenciones de
Markdown también son muy eficientes en tokens.

## Requisitos previos
MarkItDown requiere Python 3.10 o superior. Se recomienda usar un entorno virtual para evitar conflictos de dependencias.

Con una instalación estándar de Python, puedes crear y activar un entorno virtual con los siguientes comandos:

```bash
python -m venv .venv
source .venv/bin/activate
```

Si usas `uv`, puedes crear un entorno virtual con:

```bash
uv venv --python=3.12 .venv
source .venv/bin/activate
# NOTA: Asegúrate de usar 'uv pip install' y no solo 'pip install' para instalar paquetes en este entorno virtual
```

Si usas Anaconda, puedes crear un entorno virtual con:

```bash
conda create -n markitdown python=3.12
conda activate markitdown
```

## Instalación

Para instalar MarkItDown, usa pip: `pip install 'markitdown[all]'`. Como alternativa, puedes instalarlo desde el código fuente:

```bash
git clone git@github.com:microsoft/markitdown.git
cd markitdown
pip install -e 'packages/markitdown[all]'
```

## Uso

### Línea de comandos

```bash
markitdown path-to-file.pdf > document.md
```

O usa `-o` para especificar el archivo de salida:

```bash
markitdown path-to-file.pdf -o document.md
```

También puedes canalizar contenido:

```bash
cat path-to-file.pdf | markitdown
```

### Dependencias opcionales
MarkItDown tiene dependencias opcionales para activar distintos formatos de archivo. Antes, en este documento, instalamos todas con la opción `[all]`. Sin embargo, también puedes instalarlas individualmente para tener más control. Por ejemplo:

```bash
pip install 'markitdown[pdf, docx, pptx]'
```

instalará solo las dependencias para archivos PDF, DOCX y PPTX.

Actualmente, están disponibles las siguientes dependencias opcionales:

* `[all]` Instala todas las dependencias opcionales
* `[pptx]` Instala dependencias para archivos de PowerPoint
* `[docx]` Instala dependencias para archivos de Word
* `[xlsx]` Instala dependencias para archivos de Excel
* `[xls]` Instala dependencias para archivos de Excel antiguos
* `[pdf]` Instala dependencias para archivos PDF
* `[outlook]` Instala dependencias para mensajes de Outlook
* `[az-doc-intel]` Instala dependencias para Azure Document Intelligence
* `[audio-transcription]` Instala dependencias para transcripción de audio en archivos wav y mp3
* `[youtube-transcription]` Instala dependencias para obtener transcripciones de videos de YouTube

### Plugins

MarkItDown también admite plugins de terceros. Los plugins están deshabilitados por defecto. Para listar los plugins instalados:

```bash
markitdown --list-plugins
```

Para habilitar plugins, usa:

```bash
markitdown --use-plugins path-to-file.pdf
```

Para encontrar plugins disponibles, busca en GitHub el hashtag `#markitdown-plugin`. Para desarrollar un plugin, consulta `packages/markitdown-sample-plugin`.

#### Plugin markitdown-ocr

El plugin `markitdown-ocr` agrega compatibilidad OCR a los convertidores de PDF, DOCX, PPTX y XLSX, extrayendo texto de imágenes incrustadas mediante LLM Vision, con el mismo patrón `llm_client` / `llm_model` que MarkItDown ya usa para descripciones de imágenes. No requiere nuevas bibliotecas de ML ni dependencias binarias.

**Instalación:**

```bash
pip install markitdown-ocr
pip install openai  # o cualquier cliente compatible con OpenAI
```

**Uso:**

Pasa el mismo `llm_client` y `llm_model` que usarías para descripciones de imágenes:

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

Si no se proporciona `llm_client`, el plugin igual se carga, pero OCR se omite silenciosamente y se usa el convertidor estándar incorporado.

Consulta [`packages/markitdown-ocr/README.md`](packages/markitdown-ocr/README.md) para documentación detallada.

### Azure Document Intelligence

Para usar Microsoft Document Intelligence en la conversión:

```bash
markitdown path-to-file.pdf -o document.md -d -e "<document_intelligence_endpoint>"
```

Puedes encontrar más información sobre cómo configurar un recurso de Azure Document Intelligence [aquí](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/how-to-guides/create-document-intelligence-resource?view=doc-intel-4.0.0)

### API de Python

Uso básico en Python:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False) # Ponlo en True para habilitar plugins
result = md.convert("test.xlsx")
print(result.text_content)
```

Conversión con Document Intelligence en Python:

```python
from markitdown import MarkItDown

md = MarkItDown(docintel_endpoint="<document_intelligence_endpoint>")
result = md.convert("test.pdf")
print(result.text_content)
```

Para usar modelos de lenguaje grandes para descripciones de imágenes (por ahora solo para archivos pptx e imágenes), proporciona `llm_client` y `llm_model`:

```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o", llm_prompt="optional custom prompt")
result = md.convert("example.jpg")
print(result.text_content)
```

### Docker

```sh
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

## Contribuciones

Este proyecto da la bienvenida a contribuciones y sugerencias. La mayoría de las contribuciones requieren
que aceptes un Contributor License Agreement (CLA), declarando que tienes el derecho de concedernos
los derechos para usar tu contribución, y que efectivamente lo haces. Para más detalles, visita https://cla.opensource.microsoft.com.

Cuando envías un pull request, un bot de CLA determinará automáticamente si necesitas proporcionar
un CLA y decorará el PR según corresponda (por ejemplo, con una comprobación de estado o comentario).
Solo sigue las instrucciones del bot. Solo tendrás que hacerlo una vez para todos los repositorios que usan nuestro CLA.

Este proyecto adoptó el [Código de Conducta de Código Abierto de Microsoft](https://opensource.microsoft.com/codeofconduct/).
Para más información, consulta las [Preguntas frecuentes del Código de Conducta](https://opensource.microsoft.com/codeofconduct/faq/) o
contacta a [opencode@microsoft.com](mailto:opencode@microsoft.com) si tienes preguntas o comentarios adicionales.

### Cómo contribuir

Puedes ayudar revisando issues o ayudando a revisar PRs. Cualquier issue o PR es bienvenido, pero también hemos marcado algunos como 'open for contribution' y 'open for reviewing' para facilitar las contribuciones de la comunidad. Por supuesto, estas son solo sugerencias y puedes contribuir de la forma que prefieras.

<div align="center">

|            | Todos                                                       | Especialmente necesita ayuda de la comunidad                                                                                               |
| ---------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Issues** | [Todos los issues](https://github.com/microsoft/markitdown/issues) | [Issues abiertos para contribución](https://github.com/microsoft/markitdown/issues?q=is%3Aissue+is%3Aopen+label%3A%22open+for+contribution%22) |
| **PRs**    | [Todos los PRs](https://github.com/microsoft/markitdown/pulls)     | [PRs abiertos para revisión](https://github.com/microsoft/markitdown/pulls?q=is%3Apr+is%3Aopen+label%3A%22open+for+reviewing%22)              |

</div>

### Ejecución de pruebas y comprobaciones

- Ve al paquete de MarkItDown:

  ```sh
  cd packages/markitdown
  ```

- Instala `hatch` en tu entorno y ejecuta las pruebas:

  ```sh
  pip install hatch  # Otras formas de instalar hatch: https://hatch.pypa.io/dev/install/
  hatch shell
  hatch test
  ```

  (Alternativa) Usa el Devcontainer, que ya tiene todas las dependencias instaladas:

  ```sh
  # Reabre el proyecto en Devcontainer y ejecuta:
  hatch test
  ```

- Ejecuta las comprobaciones de pre-commit antes de enviar un PR: `pre-commit run --all-files`

### Contribuir con plugins de terceros

También puedes contribuir creando y compartiendo plugins de terceros. Consulta `packages/markitdown-sample-plugin` para más detalles.

## Marcas registradas

Este proyecto puede contener marcas registradas o logotipos de proyectos, productos o servicios. El uso autorizado de las
marcas registradas o logotipos de Microsoft está sujeto a, y debe cumplir, las
[Directrices de marca y marcas registradas de Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
El uso de marcas o logotipos de Microsoft en versiones modificadas de este proyecto no debe causar confusión ni implicar patrocinio de Microsoft.
Cualquier uso de marcas o logotipos de terceros está sujeto a las políticas de esos terceros.
