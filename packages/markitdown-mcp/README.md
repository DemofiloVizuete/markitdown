# MarkItDown-MCP

> [!IMPORTANT]
> El paquete MarkItDown-MCP está pensado para **uso local**, con agentes locales de confianza. En particular, al ejecutar el servidor MCP con Streamable HTTP o SSE, se enlaza por defecto a `localhost` y no queda expuesto a otras máquinas de la red o de Internet. En esta configuración, está pensado como alternativa directa al transporte STDIO, que puede ser más conveniente en algunos casos. NO enlaces el servidor a otras interfaces a menos que entiendas las [implicaciones de seguridad](#consideraciones-de-seguridad) de hacerlo.


[![PyPI](https://img.shields.io/pypi/v/markitdown-mcp.svg)](https://pypi.org/project/markitdown-mcp/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown-mcp)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)

El paquete `markitdown-mcp` proporciona un servidor MCP ligero en STDIO, Streamable HTTP y SSE para invocar MarkItDown.

Expone una herramienta: `convert_to_markdown(uri)`, donde `uri` puede ser cualquier URI `http:`, `https:`, `file:` o `data:`.

## Instalación

Para instalar el paquete, usa pip:

```bash
pip install markitdown-mcp
```

## Uso

Para ejecutar el servidor MCP usando STDIO (por defecto), usa el siguiente comando:


```bash	
markitdown-mcp
```

Para ejecutar el servidor MCP usando Streamable HTTP y SSE, usa el siguiente comando:

```bash	
markitdown-mcp --http --host 127.0.0.1 --port 3001
```

## Ejecución en Docker

Para ejecutar `markitdown-mcp` en Docker, crea la imagen con el Dockerfile incluido:
```bash
docker build -t markitdown-mcp:latest .
```

Y ejecútala con:
```bash
docker run -it --rm markitdown-mcp:latest
```
Esto será suficiente para URIs remotas. Para acceder a archivos locales, necesitas montar el directorio local dentro del contenedor. Por ejemplo, si quieres acceder a archivos en `/home/user/data`, puedes ejecutar:

```bash
docker run -it --rm -v /home/user/data:/workdir markitdown-mcp:latest
```

Una vez montado, todos los archivos de `data` estarán accesibles en `/workdir` dentro del contenedor. Por ejemplo, si tienes un archivo `example.txt` en `/home/user/data`, en el contenedor estará disponible en `/workdir/example.txt`.

## Acceso desde Claude Desktop

Se recomienda usar la imagen Docker al ejecutar el servidor MCP para Claude Desktop.

Sigue [estas instrucciones](https://modelcontextprotocol.io/quickstart/user#for-claude-desktop-users) para acceder al archivo `claude_desktop_config.json` de Claude.

Edítalo para incluir la siguiente entrada JSON:

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

Si quieres montar un directorio, ajústalo según corresponda:

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
	"run",
	"--rm",
	"-i",
	"-v",
	"/home/user/data:/workdir",
	"markitdown-mcp:latest"
      ]
    }
  }
}
```

## Depuración

Para depurar el servidor MCP puedes usar la herramienta `MCP Inspector`.

```bash
npx @modelcontextprotocol/inspector
```

Luego puedes conectarte al inspector mediante el host y puerto indicados (por ejemplo, `http://localhost:5173/`).

Si usas STDIO:
* selecciona `STDIO` como tipo de transporte,
* introduce `markitdown-mcp` como comando, y
* haz clic en `Connect`

Si usas Streamable HTTP:
* selecciona `Streamable HTTP` como tipo de transporte,
* introduce `http://127.0.0.1:3001/mcp` como URL, y
* haz clic en `Connect`

Si usas SSE:
* selecciona `SSE` como tipo de transporte,
* introduce `http://127.0.0.1:3001/sse` como URL, y
* haz clic en `Connect`

Por último:
* haz clic en la pestaña `Tools`,
* haz clic en `List Tools`,
* haz clic en `convert_to_markdown`, y
* ejecuta la herramienta con cualquier URI válida.

## Consideraciones de seguridad

El servidor no admite autenticación y se ejecuta con los privilegios del usuario que lo inicia. Por esta razón, cuando se ejecuta en modo SSE o Streamable HTTP, el servidor se enlaza por defecto a `localhost`. Aun así, es importante reconocer que el servidor puede ser accedido por cualquier proceso o usuario de la misma máquina local, y que la herramienta `convert_to_markdown` puede usarse para leer cualquier archivo al que tenga acceso el usuario del servidor, o datos de la red. Si necesitas seguridad adicional, considera ejecutar el servidor en un entorno aislado, como una máquina virtual o contenedor, y asegúrate de configurar correctamente los permisos de usuario para limitar el acceso a archivos sensibles y segmentos de red. Sobre todo, NO enlaces el servidor a otras interfaces (no localhost) a menos que entiendas las implicaciones de seguridad de hacerlo.

## Marcas registradas

Este proyecto puede contener marcas registradas o logotipos de proyectos, productos o servicios. El uso autorizado de las
marcas registradas o logotipos de Microsoft está sujeto a, y debe cumplir, las
[Directrices de marca y marcas registradas de Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
El uso de marcas o logotipos de Microsoft en versiones modificadas de este proyecto no debe causar confusión ni implicar patrocinio de Microsoft.
Cualquier uso de marcas o logotipos de terceros está sujeto a las políticas de esos terceros.
