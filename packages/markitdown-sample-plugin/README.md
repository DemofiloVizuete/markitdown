# MarkItDown Sample Plugin

[![PyPI](https://img.shields.io/pypi/v/markitdown-sample-plugin.svg)](https://pypi.org/project/markitdown-sample-plugin/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown-sample-plugin)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)


Este proyecto muestra cómo crear un plugin de ejemplo para MarkItDown. Las partes más importantes son las siguientes:

A continuación, implementa tu `DocumentConverter` personalizado:

```python
from typing import BinaryIO, Any
from markitdown import MarkItDown, DocumentConverter, DocumentConverterResult, StreamInfo

class RtfConverter(DocumentConverter):

    def __init__(
        self, priority: float = DocumentConverter.PRIORITY_SPECIFIC_FILE_FORMAT
    ):
        super().__init__(priority=priority)

    def accepts(
        self,
        file_stream: BinaryIO,
        stream_info: StreamInfo,
        **kwargs: Any,
    ) -> bool:
	
	# Implement logic to check if the file stream is an RTF file
	# ...
	raise NotImplementedError()


    def convert(
        self,
        file_stream: BinaryIO,
        stream_info: StreamInfo,
        **kwargs: Any,
    ) -> DocumentConverterResult:

	# Implement logic to convert the file stream to Markdown
	# ...
	raise NotImplementedError()
```

Next, make sure your package implements and exports the following:

```python
# The version of the plugin interface that this plugin uses. 
# The only supported version is 1 for now.
__plugin_interface_version__ = 1 

# The main entrypoint for the plugin. This is called each time MarkItDown instances are created.
def register_converters(markitdown: MarkItDown, **kwargs):
    """
    Called during construction of MarkItDown instances to register converters provided by plugins.
    """

    # Simply create and attach an RtfConverter instance
    markitdown.register_converter(RtfConverter())
```


Por último, crea un entrypoint en el archivo `pyproject.toml`:

```toml
[project.entry-points."markitdown.plugin"]
sample_plugin = "markitdown_sample_plugin"
```

Aquí, el valor de `sample_plugin` puede ser cualquier clave, pero idealmente debería ser el nombre del plugin. El valor es el nombre totalmente calificado del paquete que implementa el plugin.


## Instalación

Para usar el plugin con MarkItDown, debe estar instalado. Para instalarlo desde el directorio actual usa:

```bash
pip install -e .
```

Una vez instalado el paquete del plugin, verifica que esté disponible para MarkItDown ejecutando:

```bash
markitdown --list-plugins
```

Para usar el plugin en una conversión, usa el flag `--use-plugins`. Por ejemplo, para convertir un archivo RTF:

```bash
markitdown --use-plugins path-to-file.rtf
```

En Python, los plugins se pueden habilitar así:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=True) 
result = md.convert("path-to-file.rtf")
print(result.text_content)
```

## Marcas registradas

Este proyecto puede contener marcas registradas o logotipos de proyectos, productos o servicios. El uso autorizado de las
marcas registradas o logotipos de Microsoft está sujeto a, y debe cumplir, las
[Directrices de marca y marcas registradas de Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
El uso de marcas o logotipos de Microsoft en versiones modificadas de este proyecto no debe causar confusión ni implicar patrocinio de Microsoft.
Cualquier uso de marcas o logotipos de terceros está sujeto a las políticas de esos terceros.
