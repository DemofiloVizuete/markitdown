1

Introduccion

Los modelos de lenguaje de gran tamano (LLM) se estan convirtiendo en un bloque fundamental para desarrollar agentes
poderosos que usan LLM para razonamiento, uso de herramientas y adaptacion a nuevas observaciones (Yao et al., 2022; Xi
et al., 2023; Wang et al., 2023b) en muchas tareas del mundo real. Dado el crecimiento de las tareas que podrian
beneficiarse de los LLM y el aumento de la complejidad de dichas tareas, una aproximacion intuitiva para escalar la potencia de
los agentes es usar multiples agentes que cooperen. Trabajos previos sugieren que varios agentes pueden ayudar
a fomentar el pensamiento divergente (Liang et al., 2023), mejorar la veracidad y el razonamiento (Du et al., 2023),
y proporcionar validacion (Wu et al., 2023). A la luz de esta intuicion y de la evidencia temprana prometedora, resulta
interesante plantear la siguiente pregunta: ¿como podemos facilitar el desarrollo de aplicaciones con LLM
que puedan abarcar un amplio espectro de dominios y complejidades basadas en el enfoque multiagente?

Nuestra idea es usar conversaciones multiagente para lograrlo. Hay al menos tres razones que con-
firman su viabilidad y utilidad general gracias a los avances recientes en LLM: primero, como los LLM
optimizados para chat (por ejemplo, GPT-4) muestran capacidad de incorporar retroalimentacion, los agentes LLM pueden cooperar
mediante conversaciones entre si o con humano(s), por ejemplo, un dialogo donde los agentes ofrecen y solicitan ra-
zonamiento, observaciones, criticas y validacion. Segundo, debido a que un solo LLM puede mostrar un amplio
rango de capacidades (especialmente cuando se configura con el prompt y ajustes de inferencia correctos),
las conversaciones entre agentes configurados de forma distinta pueden ayudar a combinar esas capacidades de LLM
de forma modular y complementaria. Tercero, los LLM han demostrado capacidad para resolver tareas complejas
cuando estas se dividen en subtareas mas simples. Las conversaciones multiagente pueden habilitar esta
particion e integracion de forma intuitiva. ¿Como podemos aprovechar las ideas anteriores y
soportar distintas aplicaciones con el requisito comun de coordinar multiples agentes, poten-
cialmente respaldados por LLM, personas o herramientas con diferentes capacidades? Buscamos un
marco de conversacion multiagente con abstraccion generica e implementacion efectiva, con la flexibili-
dad para satisfacer distintas necesidades de aplicacion. Lograrlo requiere responder dos preguntas criticas:
(1) ¿Como podemos disenar agentes individuales que sean capaces, reutilizables, personalizables y eficaces en
la colaboracion multiagente? (2) ¿Como podemos desarrollar una interfaz simple y unificada que pueda
adaptarse a una amplia variedad de patrones de conversacion entre agentes? En la practica, aplicaciones de
distinta complejidad pueden necesitar conjuntos diferentes de agentes con capacidades especificas, y pueden requerir distintos
patrones de conversacion, como dialogos de uno o varios turnos, distintos modos de participacion humana y
conversacion estatica frente a dinamica. Ademas, los desarrolladores pueden preferir la flexibilidad de programar
interacciones entre agentes en lenguaje natural o en codigo. No abordar adecuadamente estas dos preguntas
limitaria el alcance de aplicabilidad y generalidad del marco.
Aunque existe exploracion contemporanea de enfoques multiagente,3 presentamos AutoGen, un
marco generalizado de conversacion multiagente (Figura 1), basado en los siguientes conceptos nuevos.
1 Agentes personalizables y conversables. AutoGen usa un diseno generico de agentes que puede apro-
vechar LLM, entradas humanas, herramientas o una combinacion de ellas. El resultado es que los desarrolladores pueden
crear facil y rapidamente agentes con diferentes roles (por ejemplo, agentes para escribir codigo, ejecutar codigo,
incorporar feedback humano, validar salidas, etc.) seleccionando y configurando un subconjunto de capacidades
integradas. El backend del agente tambien puede ampliarse facilmente para permitir comportamientos mas personalizados.
Para que estos agentes sean adecuados para la conversacion multiagente, cada agente se hace conversable:
puede recibir, reaccionar y responder mensajes. Si se configura correctamente, un agente puede mantener
multiples turnos de conversacion con otros agentes de forma autonoma o solicitar entradas humanas en cier-
tas rondas, habilitando agencia humana y automatizacion. El diseno de agente conversable aprovecha la
gran capacidad de los LLM mas avanzados para incorporar feedback y progresar mediante chat,
y tambien permite combinar capacidades de LLM de manera modular. (Seccion 2.1)

2 Programacion de conversaciones. Una idea fundamental de AutoGen es simplificar y unificar los flu-
jos complejos de aplicaciones con LLM como conversaciones multiagente. Por eso AutoGen adopta un paradigma de progra-
macion centrado en estas conversaciones entre agentes. Nos referimos a este paradigma como
programacion de conversaciones, que agiliza el desarrollo de aplicaciones complejas mediante dos
pasos principales: (1) definir un conjunto de agentes conversables con capacidades y roles especificos (como
se describio arriba); (2) programar el comportamiento de interaccion entre agentes mediante computo y control
centrados en la conversacion. Ambos pasos pueden lograrse mediante una fusion de lenguaje natural y lenguaje de pro-
gramacion para construir aplicaciones con una amplia variedad de patrones de conversacion y comportamientos de agentes.
AutoGen ofrece implementaciones listas para usar y tambien permite extension y experimentacion sencillas
en ambos pasos. (Seccion 2.2)

3Nos remitimos al Apendice A para una discusion detallada.

2

