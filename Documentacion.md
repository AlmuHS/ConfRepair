**Autómata de reparación de ficheros de configuración**

**Objetivo**

El objetivo del autómata es evitar que errores producidos durante la edición de diversos ficheros de configuración afecten a la integridad del sistema operativo, haciendo que este, o una parte importante del mismo, deje de funcionar con normalidad.

Ademas, para evitar incomodidades a los usuarios, solo se restauraran aquellas lineas que sean erróneas, manteniendo el resto de cambios que pudieran ser correctos.

Este autómata se centrara en sistemas operativos basados en UNIX, con un enfoque especial en GNU/Linux.

**Funcionamiento básico**

Este autómata de corrección de ficheros se basara en uso de versiones.

Para ello, el autómata dispondrá de un registro, que le permitirá obtener la ultima versión correcta de cualquier fichero de configuración a corregir.

Cuando el autómata detecte que un fichero de configuración ha sido editado, iniciara el proceso de revisión para detectar posibles errores.

Si detecta algún error, se usará el registro para obtener la ultima versión correcta del fichero, y reemplazar la linea errónea por su ultimo valor correcto.

Una vez el fichero este correcto, el autómata actualizara los datos del registro con la nueva versión del fichero.

**Consideraciones**

- El registro no almacenara copias físicas de los ficheros, sino que solo guardara los datos necesarios para poderlos regenerar u obtener los últimos valores correctos de sus parámetros.

- El autómata podrá reparar errores tanto sintácticos como semánticos, pudiendo ser estos últimos dependientes del sistema operativo o de la maquina donde se ejecute. 

- Al reparar los errores, hay que considerar la posibilidad de que la reparación de una linea errónea genere un nuevo error en una linea previamente considerada correcta, la cual también habrá que reparar.

También cabe la posibilidad de que el error este producido por una linea eliminada, caso en el cual habrá que restaurarla, teniendo cuidado con los anteriores casos de error,

- Si la instalación del autómata se realiza sobre un sistema operativo "en sucio", habrá que realizar un paso previo de revisión y corrección de los ficheros existentes.

Dado que no dispondríamos de ninguna versión previa para realizar la corrección, este mecanismo es muchísimo mas complejo que el anterior.

Por razones de eficiencia, este nuevo autómata de corrección de ficheros, sin uso de versiones, solo se usara durante la ejecución inicial, usando para el resto de casos el autómata inicial basado en versiones.

- Dada la cantidad de variantes existentes en cada sistema operativo, con miles de ficheros de configuración existentes, nos centraremos unicamente en un pequeño conjunto de ficheros, correspondiente a los componentes mas críticos del SO, los cuales habrá que decidir antes de implementar el autómata.

**Extras**

En un futuro, se podrían añadir nuevas funcionalidades para restaurar ficheros eliminados, o comprobar relaciones entre ficheros que pudieran causar error

**Posibles implementaciones**

**1. Implementación completa con extras**

Esta implementación abarca todas las funcionalidades del autómata, pero a su vez es la menos eficiente y la que menos casuística abarca en la resolución de errores dentro del fichero.

Se basa en chequeos periódicos, en los cuales detecta modificaciones en el árbol de ficheros y directorios, y posibles errores en los mismos.Este chequeo permite no solo detectar ficheros erróneos, sino también ficheros eliminados que debieran ser restaurados. Ademas, podrá ser instalado en un sistema "en sucio"

En el primer chequeo se usara un sistema de detección de errores sin uso de versiones, para corregir  todos los posibles errores existentes en el árbol de ficheros, corregirlos, e inicializar el registro.

La revisión de los ficheros existentes se realiza linea a linea: el autómata va recorriendo el fichero y, mediante una determinada algorítmica de procesamiento de lenguaje, detecta los posibles errores e inicia el proceso de reparación.

Si esa reparación se realiza durante el chequeo inicial, el proceso de reparación se realiza mediante otra algorítmica de procesamiento de lenguaje.

En  el resto de casos, el proceso de reparación usa una copia de la anterior versión del fichero, generada a partir de los datos almacenados en el registro, y realiza una búsqueda para obtener el anterior valor de la linea modificada y restaurarla en el fichero erróneo.

Una vez terminado de recorrer el fichero, se vuelve a recorrer otra vez desde el principio, para detectar posibles nuevos errores originados por dicha restauración.

Este proceso se repite tantas veces como sea necesario, hasta que se consiga leer el fichero completo sin encontrar error, caso en el cual se pasaría a considerar el fichero como correcto.

Dado que la revisión se realiza basándose en el fichero modificado, no se contempla la posibilidad de restaurar lineas borradas, lo cual podría seguir siendo una fuente de errores en ficheros considerados correctos.

Ademas, al realizarse con chequeos periódicos, es posible que los errores no sean detectados a tiempo y el autómata falle.

**2. Implementación simplificada sin registro**

Esta implementación es una variante de la anterior, pero limitando el problema a una instalación en limpio, y usando como registro el propio fichero actual.

Ademas, en vez de realizarse chequeos periódicos para detectar errores, el autómata se inicia al detectar la edición de uno de los ficheros.

El funcionamiento es el siguiente:

Damos por hecho que el sistema esta completamente correcto, con lo cual nos saltamos el chequeo inicial. Ademas, asumimos que el fichero que esta editando el usuario también esta inicialmente correcto, con lo cual no necesitamos registro para almacenar la versión anterior.

Cuando el usuario solicita una edición del fichero, el autómata genera una copia de seguridad del fichero actual, la cual se usara mas adelante para restaurar los errores.

Una vez la edición del fichero termina, el autómata inicia la revisión del mismo, de forma similar a la implementación anterior, leyendo el fichero linea a linea y repitiendo el proceso hasta que el fichero completo este correcto.

En caso de detectar un error, se usa la copia de seguridad antes mencionada para buscar el anterior valor correcto y restablecerlo en el fichero.

Como se puede percibir, esta implementación abarca menos funcionalidades que la anterior, dado que ni siquiera plantea el que el fichero pudiera estar corrupto o inaccesible (o cualquier otra situación anómala que haga que el fichero este incorrecto) y no puede resolver lineas borradas. Tampoco podría restaurar ficheros eliminados.

A cambio, al iniciarse la reparación tan pronto termina la edición, la posibilidad de fallos del sistema generados por estos errores se reduce.

**3. Implementación avanzada mediante Base de Datos**

En esta implementación, usaremos las propiedades de las bases de datos para crear el registro de ficheros.

El registro usara una Base de Datos, con una tabla por fichero, en la que se indicaran los diferentes parámetros que admite cada fichero (un parámetro por tupla), si el parámetro es opcional o no, sus restricciones tanto sintácticas como semánticas, y los parámetros que dependen de cada uno.

Si un parámetro opcional no tiene valor asignado, este puede dejarse en blanco o ponerse a "null".

Las restricciones podrán ser estáticas o dinámicas, según dependan del propio programa o de otras condiciones de hardware o software.

Las restricciones estáticas son aquellas que no dependan de ningún factor externo al programa,  como las sintácticas, o las que van asociadas a rangos de valores admitidos para un determinado parámetro, entre otras.

En cambio, las restricciones dinámicas dependen de factores relacionados con el hardware de la maquina o con el propio sistema operativo donde se ejecuta el programa. Ejemplos de estas podrían ser las relacionadas con la ruta de un fichero, que permitan comprobar si la ruta es correcta; u otra que compruebe si una determinada resolución de pantalla es admitida por la tarjeta gráfica.

Cuando el autómata detecta que un fichero ha sido modificado, intenta actualizar los valores almacenados en la base de datos con los nuevos datos del fichero.

Para ello, ejecuta un diff, en el cual se comprueba las diferencias entre los valores almacenados en el registro y los valores actuales del fichero.

Posteriormente, se realiza una simulación intentando insertar los nuevos valores en la tabla correspondiente al fichero, y anotando las diferentes situaciones erróneas para su posterior resolución.

Las diferentes situaciones que se pueden dar son las siguientes: 

1. Si un valor no se puede insertar por incumplir la restricción, este valor se considera erróneo, y deberá restaurarse en el fichero con el valor almacenado en el registro.

    1. Si el parámetro restaurado tiene dependencias con otros parámetros que han sido modificados, deberán restaurarse también sus valores.

2. Si un parámetro considerado correcto incumple las dependencias con otros parámetros no modificados,  este pasara a considerarse incorrecto y se deberá restaurar.

3. Si un parámetro obligatorio ha sido borrado, deberá restaurarse con el ultimo valor guardado en el registro, revisando previamente sus dependencias. Si los nuevos valores de sus dependencias incumplen las restricciones de este parámetro, se deberá proceder como en el caso 1.1.

4. Si en el fichero hay un nuevo parámetro de nombre irreconocible, se deberá buscar algún parámetro de nombre parecido en el registro.

    2. Si existe un parámetro de nombre parecido, se deberá restaurar el nombre del parámetro en el fichero por el encontrado en el registro y revisarse el valor del mismo según los anteriores casos.

    3. Si no se encuentra ningún parámetro de nombre parecido, se borrara su linea del fichero.

Finalmente, si después de la simulación el porcentaje de restauraciones supera el 70% del total, se restaura el fichero completo sin contar los posibles cambios correctos.

En caso contrario, se aplican los cambios según los casos anteriores, y se actualizan los valores en el registro.

Esta implementación también podría permitir restaurar ficheros críticos borrados, a partir de los datos almacenados en el registro y, aunque en este caso no se considerara, detectar posibles dependencias entre ficheros.

A cambio, su casuística es mucho mas compleja, y el uso de la base de datos puede ser poco eficiente.

Otra dificultad añadida podría ser la resolución de estas dependencias, dado que el modelo relacional no provee de ningún mecanismo para establecer dependencias entre tuplas de una misma tabla.

**Lista de servicios críticos a evaluar**

En esta lista pondremos aquellos servicios que vamos a considerar críticos para el sistema, en los cuales se va a centrar nuestro autómata para reparar sus ficheros

Esta lista no lleva ningún orden definido, solo indicamos los servicios, sin priorizar ninguno

* Núcleo y modulos

* Gestor de Arranque

* Init

* Gestión de usuarios y permisos

* Servicios de red

* Interprete de comandos y terminal

* Sistema de ficheros

* Autentificación

* sudo/su

* Gestor de paquetes

* Base de Datos

