# Servicios web libres de la Sede Electrónica del Catastro

**Versión:** 2.6

## Control de cambios

| Versión | Motivo del cambio | Fecha |
|---|---|---|
| 1.0 | Se separan los documentos de ayuda de servicios web. Creación de un documento único para los servicios web libres. | 13-01-2014 |
| 2.0 | Migración de servicios libres de ASMX a WCF, permitiendo SOAP y REST (GET/POST). | 27-10-2022 |
| 2.1 | Inclusión de respuestas en formato JSON (apartado 2.3.2). | 28-12-2022 |
| 2.2 | Inclusión de la etiqueta `<dtip>` (denominación de tipología) en el apartado 2.1.5. | 23-06-2023 |
| 2.3 | Inclusión del Anexo III (Ejemplos de uso). | 04-12-2023 |
| 2.4 | Actualización de enlaces al dominio `www.catastro.hacienda.gob.es`. | 22-05-2024 |
| 2.5 | Se ofrece información de la finca en los datos de un inmueble (apartados 2.1.5, 2.1.6 y 2.1.7). | 09-10-2025 |
| 2.6 | Inclusión de información de códigos INE en el listado de vías (apartado 2.1.3). | 01-12-2025 |

## Índice

1. [Introducción](#1-introducción)
2. [Servicios que proporcionan datos catastrales no protegidos](#2-servicios-que-proporcionan-datos-catastrales-no-protegidos)
3. [Anexo I: Errores en servicios web](#anexo-i-errores-en-servicios-web)

## 1. Introducción

Este documento describe los servicios web basados en el estándar SOAP 1.1 (`Simple Object Access Protocol`) que proporciona la Dirección General del Catastro a través de su sede electrónica con información catastral no protegida, accesible por cualquier ciudadano.

Referencia SOAP 1.1:
http://www.w3.org/TR/2000/NOTE-SOAP-20000508

Como novedad, estos servicios se migraron a tecnología WCF (`Windows Communication Foundation`), permitiendo también su uso mediante HTTP con REST (GET y POST).

Los esquemas XML de respuesta incluyen campos para informar de posibles errores de procesamiento (código y descripción).

## 2. Servicios que proporcionan datos catastrales no protegidos

Los servicios se organizan en dos bloques:

- Callejero y datos catastrales no protegidos.
- Conversor de coordenadas.

Esquemas XML de respuesta:
http://www.catastro.hacienda.gob.es/ws/esquemas.htm

### 2.1 Servicio de callejero y datos catastrales no protegidos

Permite consultar provincias, municipios, vías y números, así como datos no protegidos de inmuebles (excepto titularidad y valor). Se puede acceder por:

- Denominaciones (provincia, municipio, vía, etc.).
- Códigos (Catastro / INE).

#### 2.1.1 Listado de PROVINCIAS

**Descripción**

Devuelve el listado de provincias españolas en las que tiene competencia la Dirección General del Catastro.

**Parámetros de entrada**

- Ninguno.

**Esquema de salida**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_provinciero.xsd

**Estructura XML (resumen)**

```xml
<consulta_provinciero>
   <control>
      <cuprov>...</cuprov>
   </control>
   <provinciero>
      <prov>
         <cpine>...</cpine>
         <np>...</np>
      </prov>
   </provinciero>
</consulta_provinciero>
```

#### 2.1.2 Listado de MUNICIPIOS de una provincia

**Acceso por denominaciones**

Devuelve municipios de una provincia (filtro opcional por nombre de municipio) con códigos de Hacienda e INE y disponibilidad cartográfica.

**Parámetros**

- `Provincia` (obligatorio)
- `Municipio` (opcional)

**Acceso por códigos**

**Parámetros**

- `CodigoProvincia` (obligatorio)
- `CodigoMunicipio` (opcional)
- `CodigoMunicipioIne` (opcional)

**Esquema de salida común**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_municipiero.xsd

#### 2.1.3 Listado de VÍAS de un municipio

**Acceso por denominaciones**

Devuelve vías del municipio (filtro opcional por nombre y tipo de vía) incluyendo códigos DGC e INE.

**Parámetros**

- `Provincia` (obligatorio)
- `Municipio` (obligatorio)
- `TipoVia` (opcional)
- `NomVia` (opcional)

**Acceso por códigos**

**Parámetros**

- `CodigoProvincia` (obligatorio)
- `CodigoMunicipio` o `CodigoMunicipioIne` (uno obligatorio)
- `CodigoVia` (opcional)

**Esquema de salida común**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_callejero.xsd

#### 2.1.4 Listado de NÚMEROS de una vía

**Comportamiento**

Si el número existe, devuelve su referencia catastral. Si no existe, devuelve error y números cercanos (hasta 5 por arriba y 5 por abajo), junto con sus referencias.

**Acceso por denominaciones - parámetros**

- `Provincia` (obligatorio)
- `Municipio` (obligatorio)
- `TipoVia` (obligatorio)
- `NomVia` (obligatorio)
- `Numero` (obligatorio)

**Acceso por códigos - parámetros**

- `CodigoProvincia` (obligatorio)
- `CodigoMunicipio` o `CodigoMunicipioIne` (uno obligatorio)
- `CodigoVia` (obligatorio)
- `Numero` (obligatorio)

**Esquema de salida**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_numerero.xsd

#### 2.1.5 Consulta de DATOS CATASTRALES NO PROTEGIDOS por localización

**Qué devuelve**

Puede devolver:

1. Lista de candidatos (si provincia/municipio/vía/número no existen).
2. Lista de inmuebles que coinciden con la búsqueda.
3. Datos completos de un único inmueble.

**Datos principales que puede incluir**

- Tipo de inmueble (UR/RU)
- Referencia catastral
- Domicilio tributario
- Uso, superficie, coeficiente de participación y antigüedad
- Unidades constructivas
- Datos de finca e información gráfica
- Superficie de elementos comunes
- Subparcelas (cultivo, intensidad, superficie)

**Acceso por denominaciones - parámetros**

- `Provincia` (obligatorio)
- `Municipio` (obligatorio)
- `TipoVia` (obligatorio)
- `NomVia` (obligatorio)
- `Numero` (obligatorio)
- `Bloque`, `Escalera`, `Planta`, `Puerta` (opcionales)

**Acceso por códigos - parámetros**

- `CodigoProvincia` (obligatorio)
- `CodigoMunicipio` o `CodigoMunicipioIne` (uno obligatorio)
- `CodigoVia` (obligatorio)
- `Numero` (obligatorio)
- `Bloque`, `Escalera`, `Planta`, `Puerta` (opcionales)

**Esquema de salida**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_dnp.xsd

#### 2.1.6 Consulta de DATOS CATASTRALES NO PROTEGIDOS por referencia catastral

Es igual al servicio 2.1.5, cambiando los parámetros de entrada.

**Acceso por denominaciones - parámetros**

- `Provincia` (opcional)
- `Municipio` (opcional)
- `RefCat` (obligatorio, 14/18/20 posiciones)

**Acceso por códigos - parámetros**

- `CodigoProvincia` (opcional)
- `CodigoMunicipio` y/o `CodigoMunicipioIne` (opcionales)
- `RefCat` (obligatorio, 14/18/20 posiciones)

Nota: con `RefCat` de 14 posiciones (finca), se devuelve la lista de inmuebles de esa finca.

#### 2.1.7 Consulta de DATOS CATASTRALES NO PROTEGIDOS por polígono-parcela

Es igual al servicio 2.1.5, cambiando los parámetros de entrada.

**Acceso por denominaciones - parámetros**

- `Provincia` (obligatorio)
- `Municipio` (obligatorio)
- `Poligono` (obligatorio)
- `Parcela` (obligatorio)

**Acceso por códigos - parámetros**

- `CodigoProvincia` (obligatorio)
- `CodigoMunicipio` o `CodigoMunicipioIne` (uno obligatorio)
- `Poligono` (obligatorio)
- `Parcela` (obligatorio)

### 2.2 Servicio de coordenadas

Permite:

- Obtener referencia catastral a partir de coordenadas X, Y.
- Obtener coordenadas X, Y a partir de referencia catastral.
- Obtener referencias catastrales por proximidad a coordenadas.

#### 2.2.1 Localización de referencia catastral por coordenadas X,Y

**Parámetros**

- `SRS` (obligatorio)
- `Coordenada_X`, `Coordenada_Y` (obligatorios)

**Sistemas de referencia admitidos**

| SRS | Descripción |
|---|---|
| EPSG:4230 | Geográficas en ED50 |
| EPSG:4326 | Geográficas en WGS80 |
| EPSG:4258 | Geográficas en ETRS89 |
| EPSG:32627 | UTM huso 27N en WGS84 |
| EPSG:32628 | UTM huso 28N en WGS84 |
| EPSG:32629 | UTM huso 29N en WGS84 |
| EPSG:32630 | UTM huso 30N en WGS84 |
| EPSG:32631 | UTM huso 31N en WGS84 |
| EPSG:25829 | UTM huso 29N en ETRS89 |
| EPSG:25830 | UTM huso 30N en ETRS89 |
| EPSG:25831 | UTM huso 31N en ETRS89 |
| EPSG:23029 | UTM huso 29N en ED50 |
| EPSG:23030 | UTM huso 30N en ED50 |
| EPSG:23031 | UTM huso 31N en ED50 |

**Esquema de salida**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_coordenadas.xsd

#### 2.2.2 Localización de coordenadas X,Y por referencia catastral

**Parámetros**

- `Provincia` (opcional; obligatoria si se indica municipio)
- `Municipio` (opcional)
- `SRS` (opcional)
- `RefCat` (obligatorio, 14 posiciones)

**Esquema de salida**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_coordenadas.xsd

#### 2.2.3 Localización de referencias catastrales por proximidad a coordenadas X,Y

**Parámetros**

- `SRS` (obligatorio)
- `Coordenada_X`, `Coordenada_Y` (obligatorios)

**Esquema de salida**

http://www.catastro.hacienda.gob.es/ws/esquemas/consulta_coordenadas_distancias.xsd

### 2.3 Invocación de los servicios

#### 2.3.1 Introducción

**WSDL**

- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc?singleWsdl
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejeroCodigos.svc?singleWsdl
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCoordenadas.svc?singleWsdl

**Endpoints SOAP**

- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc/soap
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejeroCodigos.svc/soap
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCoordenadas.svc/soap

**Ayuda REST (GET/POST)**

- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc/rest/help
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejeroCodigos.svc/rest/help
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCoordenadas.svc/rest/help

#### 2.3.2 Formato de salida JSON

Los servicios devuelven XML por defecto. Para JSON en REST:

- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc/json/help
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejeroCodigos.svc/json/help
- http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCoordenadas.svc/json/help

#### 2.3.3 Servicios de callejero y datos catastrales no protegidos

**Servicio por denominación**

URL base:
http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc

Métodos:

- `ConsultaProvincia`
- `ConsultaMunicipio`
- `ConsultaVia`
- `ConsultaNumero`
- `Consulta_DNPLOC`
- `Consulta_DNPRC`
- `Consulta_DNPPP`

**Servicio por códigos**

URL base:
http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejeroCodigos.svc

Métodos:

- `ConsultaProvincia`
- `ConsultaMunicipioCodigos`
- `ConsultaViaCodigos`
- `ConsultaNumeroCodigos`
- `Consulta_DNPLOC_Codigos`
- `Consulta_DNPRC_Codigos`
- `Consulta_DNPPP_Codigos`

Invocación admitida: SOAP, REST GET y REST POST.

#### 2.3.4 Servicio de coordenadas

URL base:
http://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCoordenadas.svc

Métodos:

- `Consulta_RCCOOR` (RC por coordenadas)
- `Consulta_RCCOOR_Distancia` (RC próximas por distancia a coordenadas)
- `Consulta_CPMRC` (coordenadas por provincia, municipio y RC)

Invocación admitida: SOAP, REST GET y REST POST.

## Anexo I: Errores en servicios web

Posibles errores al llamar a los servicios web.

### Notación

- `MAN`: errores generales a todos los servicios web.
- `CAL`: errores de servicios `OVCCallejero`, `OVCCallejeroCodigos` y `OVCCoordenadas`.

### Errores generales

| Código | Tipo envío | Descripción |
|---|---|---|
| 1 | MAN | Error en el servidor |

### Errores de localización RC (`OVCSWLocalizacionRC`)

**Códigos detectados (tipo `CAL`)**

`1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 15, 16, 17, 20, 21, 22, 23, 24, 31, 32, 33, 34, 35, 36, 41, 42, 43, 51, 53, 61, 62, 63, 64, 71, 72, 73, 74, 75, 76, 77, 78, 79`

**Mensajes de error documentados**

- Error en el servidor.
- La referencia catastral es obligatoria.
- La referencia catastral debe tener 14, 18 o 20 posiciones.
- La referencia catastral no está correctamente formada.
- No existe ningún inmueble con los parámetros indicados.
- Error al consultar la referencia.
- La referencia catastral debe ser una combinación de letras y números hasta un total de 20 caracteres.
- El huso geográfico debe ser una secuencia de hasta 10 caracteres.
- La referencia catastral no existe.
- La provincia es obligatoria.
- La provincia no existe.
- La provincia debe ser una secuencia de hasta 25 caracteres.
- Error al buscar las coordenadas.
- Para esas coordenadas no hay referencia disponible.
- Error al consultar el municipio.
- El municipio es obligatorio.
- El municipio no existe.
- El código de municipio debe ser una secuencia de hasta 3 dígitos.
- El municipio debe ser una secuencia de hasta 40 caracteres.
- El tipo de vía es obligatorio.
- La vía es obligatoria.
- La vía no existe.
- El nombre de la vía debe ser una secuencia de hasta 25 caracteres.
- El tipo de vía debe ser una secuencia de hasta 5 caracteres.
- El código de vía debe ser una secuencia de hasta 5 dígitos.
- El número es obligatorio.
- El número debe ser una secuencia de hasta 4 dígitos.
- El número no existe.
- El bloque debe ser una secuencia de hasta 4 caracteres.
- Error en el formato del bloque, escalera, planta o puerta.
- El polígono es obligatorio.
- La parcela es obligatoria.
- El polígono debe ser una secuencia de hasta 3 dígitos.
- La parcela debe ser una secuencia de hasta 5 dígitos.
- Error al recuperar el domicilio.
- No hay cartografía disponible.
- Error al buscar la parcela.
- Error al generar el retorno.
- El campo SRS es obligatorio.
- La coordenada X es obligatoria.
- La coordenada X debe ser un valor numérico.
- La coordenada Y debe ser un valor numérico.
- La coordenada Y es obligatoria.

## Anexo II: Tipos de vía

| Código | Descripción | Código | Descripción |
|---|---|---|---|
| AC | Acceso | LA | Lago |
| AG | Agregado | LD | Lado, ladera |
| AL | Aldea, alameda | LG | Lugar |
| AN | Andador | MA | Malecón |
| AR | Área, arrabal | MC | Mercado |
| AU | Autopista | ML | Muelle |
| AV | Avenida | MN | Municipio |
| AY | Arroyo | MS | Masías |
| BJ | Bajada | MT | Monte |
| BL | Bloque | MZ | Manzana |
| BO | Barrio | PB | Poblado |
| BQ | Barranquil | PC | Placeta |
| BR | Barranco | PD | Partida |
| CA | Cañada | PI | Particular |
| CG | Colegio, cigarral | PJ | Pasaje, pasadizo |
| CH | Chalet | PL | Polígono |
| CI | Cinturón | PM | Páramo |
| CJ | Calleja, callejón | PQ | Parroquia, parque |
| CL | Calle | PR | Prolongación, continuac. |
| CM | Camino, carmen | PS | Paseo |
| CN | Colonia | PT | Puente |
| CO | Concejo, colegio | PU | Pasadizo |
| CP | Campa, campo | PZ | Plaza |
| CR | Carretera, carrera | QT | Quinta |
| CS | Caserío | RA | Raconada |
| CT | Cuesta, costanilla | RB | Rambla |
| CU | Conjunto | RC | Rincón, rincona |
| CY | Caleya | RD | Ronda |
| CZ | Callizo | RM | Ramal |
| DE | Detrás | RP | Rampa |
| DP | Diputación | RR | Riera |
| DS | Diseminados | RU | Rua |
| ED | Edificios | SA | Salida |
| EM | Extramuros | SC | Sector |
| EN | Entrada, ensanche | SD | Senda |
| EP | Espalda | SL | Solar |
| ER | Extrarradio | SN | Salón |
| ES | Escalinata | SU | Subida |
| EX | Explanada | TN | Terrenos |
| FC | Ferrocarril | TO | Torrente |
| FN | Finca | TR | Travesía |
| GL | Glorieta | UR | Urbanización |
| GR | Grupo | VA | Valle |
| GV | Gran vía | VD | Viaducto |
| HT | Huerta, huerto | VI | Vía |
| JR | Jardines | VL | Vial |
|  |  | VR | Vereda |

## 3. Anexo III: Ejemplos

A continuación se muestran ejemplos de uso de los servicios libres por las opciones disponibles.

### Petición HTTP(S) por GET

**Con salida en JSON**

https://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc/json/Consulta_DNPRC?RefCat=2749704YJ0624N0001DI

**Con salida en XML**

https://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc/rest/Consulta_DNPRC?RefCat=2749704YJ0624N0001DI

### Petición HTTP(S) por POST

**Con salida en XML**

https://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc/rest/Consulta_DNPRC

**Cuerpo de entrada**

```xml
<ConsultaRest_DNPRC_In xmlns="http://www.catastro.hacienda.gob.es/">
   <RefCat>2749704YJ0624N0001DI</RefCat>
</ConsultaRest_DNPRC_In>
```

Importante: es necesario indicar el `namespace` en la etiqueta `xmlns`.

### Petición SOAP por POST

**Endpoint**

https://ovc.catastro.meh.es/OVCServWeb/OVCWcfCallejero/COVCCallejero.svc?singleWsdl

**Envelope de ejemplo**

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:cat="http://www.catastro.hacienda.gob.es/">
   <soapenv:Header/>
   <soapenv:Body>
      <cat:RefCat>2749704YJ0624N0001DI</cat:RefCat>
   </soapenv:Body>
</soapenv:Envelope>
```

Para más información, consultar el apartado 2.3 de este documento.

