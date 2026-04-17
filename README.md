# BEQR

## Implementación de la Instrucción BEQR (Branch if Equal Register)

Proyecto en CircuitVerse: https://circuitverse.org/users/417773/projects/beqr 

Este módulo implementa la lógica de hardware para la instrucción de salto condicional BEQR en nuestra arquitectura de 16 bits. Este modulo no modifica registros, sino que le indica al PC a que direccion de memoria dirigirse. 

### Por qué necesitamos a BEQR?

En nuestra ISA, contamos con la instrucción clásica BEQ (Branch if Equal), la cual evalúa si dos registros son iguales y, de serlo, realiza un salto. Sin embargo, BEQ realiza un salto relativo sumando un offset de solo 4 bits al PC, siendo esto un limitante importante ya que solo podemos saltar unas pocas instrucciones hacia adelante o hacia atrás(+8 a -7).

La instrucción BEQR soluciona esta limitación haciendo un salto absoluto. Si la condición se cumple (rs == rt), el PC no suma un offset, sino que se copia directamente los 16 bits exactos almacenados en un tercer registro (rd). Esto nos permite alcanzar cualquier dirección en todo el espacio de memoria (0x0000 a 0xFFFF).

### Lógica del Comparador de 16 bits

Para determinar si los registros RS y RT contienen el mismo valor, diseñe un comparador de 16 bits implementado mediante una compuerta XOR y una cascada OR.

Comparación bit a bit: Los dos inputs de 16 bits (RS y RT) ingresan a una compuerta XOR de 16 bits. Si ambos registros son idénticos, el resultado de esta operación serán 16 ceros.

Detección de diferencias (Cascada OR): La salida de la XOR se separa mediante un splitter y sus 16 cables se conectan a una cascada de compuertas OR (dos compuertas de 8 entradas que luego convergen en una final de 2 entradas). Esta estructura actúa como un detector: si existe tan solo un bit distinto entre los registros, la cascada emitirá un 1.

Señal de Igualdad (EQ): Finalmente, la salida de la cascada OR atraviesa una compuerta inversora NOT. De este modo, la señal final (EQ) será 1 cuando todos los bits de los inputs hayan sido iguales. 

### Lógica de Control y Enrutamiento de Datos
Este subcircuito no fuerza la escritura en el PC. Su trabajo es emitir una señal de validación y proponer una dirección de destino para que el MUX global del procesador ejecute el salto. Como se observa en el esquema principal, esto se resuelve en dos partes:

El Control (ACTIVAR_SALTO):
Para indicarle al sistema que debe saltar, generamos una señal de 1 bit. Esto se implementa utilizando una compuerta AND de 6 entradas que exige el cumplimiento simultáneo de tres condiciones:

Decodificación del Opcode: El bus de 4 bits (OPCODE) se separa con un splitter. Para detectar el Codop 11 (1011), el bit 2 pasa por una compuerta inversora (NOT), mientras que los bits 0, 1 y 3 ingresan directo a la AND.

Validación de Modo: El pin MODE ingresa directamente, exigiendo un 1 lógico para confirmar que es la instrucción BEQR y no su variante relativa (BEQ) que comparten codop.

Condición de Igualdad: Ingresa la señal de éxito proveniente de nuestro subcircuito comparador, confirmando que RS y RT son idénticos.
Solo si todo esto se cumple, la salida ACTIVAR_SALTO se pone en alto (1).

El Dato Propuesto (NUEVO_PC):
Independientemente de si la condición de salto se cumple o no, el módulo siempre prepara la dirección a la que se saltaría. Dado que BEQR indica un salto absoluto al valor de un registro, los 16 bits que ingresan por el pin RD_VALUE se rutean mediante un cable directo hacia la salida NUEVO_PC. No hay compuertas lógicas en este trayecto; es un paso limpio. El procesador tomará este valor de 16 bits y lo cargará en el PC única y exclusivamente si la señal ACTIVAR_SALTO está encendida.

### Fuentes y Creditos

Apuntes de compañeros (ya que falte el miercoles): Utilizados como fuente principal para la asignación del Codop 11 y para comprender la lógica esperada en la integración con el procesador global.

"Computer Organization and Design" (Patterson & Hennessy): El "libro dorado" de arquitectura.

"Introduction to Computing Systems: From Bits and Gates to C and Beyond" (Yale Patt & Sanjay Patel): Bibliografía base del curso para comprender la arquitectura educativa LC-3 de 16 bits, en la cual se inspira fuertemente nuestro procesador.
