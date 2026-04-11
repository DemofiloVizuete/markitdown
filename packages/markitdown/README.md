# MarkItDown

> [!IMPORTANT]
> MarkItDown es un paquete de Python y una utilidad de línea de comandos para convertir varios tipos de archivos a Markdown (por ejemplo, para indexación, análisis de texto, etc.). 
>
> Para más información y documentación completa, consulta el [README.md](https://github.com/microsoft/markitdown) del proyecto en GitHub.

## Instalación

Desde PyPI:

```bash
pip install markitdown[all]
```

Desde código fuente:

```bash
git clone git@github.com:microsoft/markitdown.git
cd markitdown
pip install -e packages/markitdown[all]
```

## Uso

### Línea de comandos

```bash
markitdown path-to-file.pdf > document.md
```

### API de Python

```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("test.xlsx")
print(result.text_content)
```

### Más información

Para más información y documentación completa, consulta el [README.md](https://github.com/microsoft/markitdown) del proyecto en GitHub.

## Marcas registradas

Este proyecto puede contener marcas registradas o logotipos de proyectos, productos o servicios. El uso autorizado de las
marcas registradas o logotipos de Microsoft está sujeto a, y debe cumplir, las
[Directrices de marca y marcas registradas de Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
El uso de marcas o logotipos de Microsoft en versiones modificadas de este proyecto no debe causar confusión ni implicar patrocinio de Microsoft.
Cualquier uso de marcas o logotipos de terceros está sujeto a las políticas de esos terceros.
