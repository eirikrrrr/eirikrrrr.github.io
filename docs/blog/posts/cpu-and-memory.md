---
date: 2026-03-31
categories:
  - Topicos
tags:
  - Hardware
  - Conceptos
  - Spanish
---


1. Introducción
2. Modelo mental de ejecución
3. Ciclo de instrucción
4. Registros y ALU
5. Memoria: RAM y caché
6. Pipeline y ejecución moderna
7. Multicore y paralelismo
8. Interrupciones
9. Buses y comunicación
10. Sistema Operativo

10.1. Modo usuario vs kernel

10.2 Syscalls

10.3 sysenter / syscall
    
10.4 Scheduler
    
10.5 Memoria virtual (MMU)

11. Resumen
12. Quiz


# CPU | Fundamentos Basicos (Resumido)

La CPU (Unidad Central de Procesamiento) es el componente encargado de ejecutar instrucciones y procesar datos. No "piensa" ni "entiende" programas; simplemente sigue órdenes muy básicas definidas en binario.

A nivel físico, no es un bloque monolítico, sino un conjunto de subsistemas especializados que trabajan coordinadamente.

---

## 1. Modelo mental de ejecución

Antes de entrar en detalles internos, necesitas una idea clara de cómo funciona todo el sistema:

- Un **programa** es una secuencia de instrucciones almacenadas en memoria.

- La **memoria RAM** es básicamente un array de bytes direccionables.

- La **CPU** es un ejecutor secuencial que:
  1. Lee una instrucción desde memoria
  2. La interpreta
  3. La ejecuta

Ejemplo simplificado en pseudocódigo:

```c
while (true) {
    instruccion = memoria[PC];
    ejecutar(instruccion);
    PC++;
}
```

![CPU-Overview](./assets/images/cpu-theory/cpu-03.jpg)

EL PC (Program Counter) apunta siempre a la siguiente instrucción a ejecutar. Este modelo es solo una simplificación para entender todo lo que viene después.

Con esto dicho, tendras algunas preguntas, en mi caso me preguntaba ¿porque esto no es mas simple?

En la practica, el modelo tiene problemas importantes.

- La CPU es extremadamente rápida

- La RAM es relativamente lenta

- Las instrucciones pueden depender unas de otras

- Hay múltiples programas compitiendo por CPU

Para resolver estos problemas existen:

- Caché

- Pipeline

- Ejecución fuera de orden

- Sistema Operativo

Todo lo que verás a continuación existe para optimizar este modelo basico de CPU que hemos visto antes.


## 2. Ciclo de instrucción

Todo lo que hace la CPU se reduce a repetir constantemente un ciclo conocido como:

**fetch → decode → execute**

Este ciclo es la implementación real del modelo visto anteriormente.

![fetch decode and execution cycle](./assets/images/cpu-theory/cpu-04.png)

<br>

### 2.1 Fetch (búsqueda)

La CPU obtiene la siguiente instrucción desde memoria:

1. El **PC (Program Counter)** contiene la dirección de la próxima instrucción.

2. Esa dirección se envía a memoria.

3. La instrucción se carga en el **IR (Instruction Register)**.

```text

PC → Memoria → IR

```

Despues de esto, el PC normalmente se incrementa para apuntar a la siguiente instrucción.

### 2.2 Decode (decodificación)

La Unidad de Control (CU) interpreta la instrucción:

- Qué operación realizar (ADD, MOV, JMP, etc.)

- Qué operandos usar

- Qué unidades internas activar (ALU, memoria, registros)

Aquí la CPU traduce binario a señales de control internas.

### 2.3 Execute (ejecución)

Se realiza la operación:

- Operaciones aritméticas → ALU

- Acceso a memoria → lectura/escritura

- Control de flujo → saltos (JMP, CALL, etc.)

Nota: Usualmente la CPU requiere llamar valores almacenados previamentes, en esos momentos es cuando entran operaciones del control de flujos como (JMP, CALL, etc).

### 2.4 Ejemplo práctico

Supongamos la instrucción:

```text

ADD AX, BX

```

Flujo simplificado:

Fetch: 

- se carga ```ADD AX, BX``` en el IR

Decode: 

la CU identifica:

- Operación: suma

- Operandos: AX y BX

Execute:

- La ALU suma AX + BX

- El resultado se guarda en AX

## 3. Registros y ALU

La CPU no opera directamente sobre la memoria RAM en la mayoria de los casos. En su lugar, utiliza **registros internos**, que son pequeñas unidades de almacenamiento extremadamente rápidas dentro del propio procesador.

### 3.1 Registros

Los registros son celdas de memoria de muy baja latencia (Nanosegundos o menos), tipicamente del tamaño de la arquitectura. Por ejemplo: 64 bits en sistemas modernos.

Se utilizan para:

- Almacenar datos temporales
- Guardar direcciones de memoria
- Controlar la ejecución del programa

Acceder a un registro es **mucho más rápido** que acceder a RAM.

### 3.2 Tipos de registros

Registros de control

Controlan el flujo de ejecución:

- **PC (Program Counter)**: dirección de la siguiente instrucción
- **IR (Instruction Register)**: instrucción actual

Registros de memoria

Intermediarios entre CPU y RAM:

- **MAR (Memory Address Register)**: dirección a acceder
- **MDR (Memory Data Register)**: dato leído o a escribir

Registros de proposito general

Usados directamente por las instrucciones:

- AX, BX, CX, DX (x86)

- RAX, RBX, RCX, etc. Estos son mas usados en arquitecturas modernas


### 3.3 ALU (Arithmetic Logic Unit)

La **ALU** es la unidad encargada de ejecutar operaciones básicas:

- Aritméticas: suma, resta, multiplicación
- Lógicas: AND, OR, XOR, NOT
- Comparaciones

Ejemplo:

```

ADD RAX, RBX

```

Flujo:

Se cargan los valores de RAX y RBX en la ALU
La ALU realiza la suma
El resultado se escribe en RAX


### 3.4 FPU (Floating Point Unit)

La FPU es una unidad especializada para operaciones con números en punto flotante:

- float (32 bits)

- double (64 bits)

Históricamente era un coprocesador separado (ej: [Intel 80387](https://www.dosdays.co.uk/topics/math_coprocessors.php))

### 3.5 Idea simple para comprender la CPU

Esta idea es clave para comprender la CPU, intenta asimilar la siguiente frase.

**La CPU está diseñada para operar sobre registros, no sobre la RAM**

Referencias (Register)

- [Registers](https://clearinfosec.com/cpus-and-registers/)

- [Processor Register](https://en.wikipedia.org/wiki/Processor_register)

- [Logic Unit](https://en.wikipedia.org/wiki/Arithmetic_logic_unit)

- [Floating Point Unit](https://en.wikipedia.org/wiki/Floating-point_unit)

## 3. Jerarquía de caché y velocidad

La RAM es varias órdenes de magnitud más lenta que la CPU. La caché actúa como un buffer ultrarrápido.

![CPU-Overview](./assets/images/cpu-theory/cache_hierarchy_v2.svg)

La CPU busca datos primero en L1. Si hay cache miss, busca en L2, luego L3 y, si no está, en RAM. La penalización por miss puede costar cientos de ciclos. Compiladores y SO modernos buscan maximizar cache hits.

## 4. Pipeline y paralelismo interno

Sin pipeline, cada instrucción se completa antes de iniciar la siguiente. Con pipeline, las etapas se superponen, lo que mejora rendimiento.

Ejemplo de pipeline RISC de 5 etapas:

```bash

Ciclo:    1    2    3    4    5    6    7
Inst 1:  [IF] [ID] [EX] [ME] [WB]
Inst 2:       [IF] [ID] [EX] [ME] [WB]
Inst 3:            [IF] [ID] [EX] [ME] [WB]

```


Donde:
- IF: Fetch
- ID: Decode
- EX: Execute
- ME: Memory Access
- WB: Write Back

Los hazards (dependencias entre instrucciones) pueden generar stalls. Las CPU modernas usan predicción de saltos y ejecución fuera de orden para mitigarlos.

## 5. Comunicación con RAM: bus y controlador

La CPU no accede a RAM directamente. El proceso típico:

1. La CPU coloca la dirección en MAR.
2. El bus de direcciones lleva la dirección al controlador de memoria (integrado en CPU / antiguo Northbridge).
3. El controlador accede a RAM y devuelve el dato en MDR.
4. La CU copia el dato al registro destino.

El ancho del bus (64 bits moderno) y la velocidad (MHz/GHz) determinan el ancho de banda, un cuello de botella crítico.

## 6. Buses y dispositivos externos

Dispositivos externos se conectan mediante buses:

- PCIe: GPU, NVMe.
- SATA/NVMe: almacenamiento.
- USB: periféricos.
- APIC/IOAPIC: interrupciones.

Las interrupciones interrumpen la ejecución para ejecutar ISRs (Interrupt Service Routines) y luego se reanuda.

## 7. SO: intermediario

El sistema operativo administra acceso y recursos:

- Scheduler: decide qué proceso ejecuta y cuándo (Round Robin, CFS, etc.).
- MMU: traduce direcciones virtuales a físicas y protege memoria.
- DMA (Direct Memory Access): permite a dispositivos leer/escribir RAM sin CPU en cada transferencia.

## Resumen Del Funcionamiento

La CPU es el director de orquesta; registros y ALU son músicos, caché es el atril, la RAM es el archivo y el bus es el pasillo. La CU coordina todo ciclo a ciclo, miles de millones de veces por segundo.


## Quiz basico sobre CPU

1 **Concepto básico**

¿Qué es exactamente la CPU?

A) Un dispositivo de almacenamiento

B) El cerebro que ejecuta instrucciones y procesa datos'

C) Un tipo de memoria RAM

D) Un controlador de periféricos

<br>

2 **Componentes internos**

¿Cuál de los siguientes NO es un componente típico de la CPU?

A) Unidad de control

B) Registros

C) Disco duro

D) ALU

<br>

3 **ALU**

¿Qué hace la ALU?

A) Controla el flujo de datos

B) Gestiona memoria virtual

C) Ejecuta operaciones aritméticas y lógicas

D) Controla dispositivos de entrada

<br>

4 **Ciclo de instrucción**

¿Cuál es el orden correcto?

A) Execute → Fetch → Decode

B) Fetch → Decode → Execute

C) Decode → Execute → Fetch

D) Fetch → Execute → Decode

<br>

5 **Frecuencia**

La frecuencia de una CPU (GHz) indica:

A) Cantidad de núcleos

B) Tamaño de la caché

C) Consumo energético

D) Velocidad de ejecución de ciclos por segundo

<br>

6 **Multicore**

¿Qué ventaja principal tienen múltiples núcleos?

A) Mayor capacidad de almacenamiento

B) Ejecutar múltiples tareas en paralelo

C) Reducir el tamaño físico del chip

D) Aumentar la velocidad de red

<br>

7 **Caché**

¿Cuál es el propósito de la memoria caché?

A) Acelerar acceso a datos frecuentes

B) Guardar archivos del usuario

C) Sustituir la RAM

D) Almacenar el sistema operativo completo

<br>

8 **Tipos de caché**

¿Cuál suele ser más rápida?

A) L1

B) L3

C) L2

D) Todas iguales

<br>

9 **Hyperthreading / SMT**

¿Qué hace el hyperthreading?

A) Permite que un núcleo ejecute múltiples hilos

B) Reduce el consumo energético

C) Aumenta la frecuencia automáticamente

D) Duplica los núcleos físicos

<br>

10 **Bottleneck**

Si la CPU está al 100% pero la GPU al 40%, ¿qué ocurre?

A) Problema de RAM

B) Bottleneck en GPU

C) No hay problema

D) Bottleneck en CPU

## Solucion Quiz

```bash

1. b
2. c
3. c
4. b
5. d
6. b
7. a
8. a
9. a
10. d

```

# CPU | Instrucciones De CPU (ASM, Opcode & RAM)

Las "instrucciones" que la CPU ejecuta, son solo datos binarios, el formato de todas las instrucciones es el mismo siempre y esta compuesto por dos elementos:

- La operacion a ejecutar

- Los operando o valores.


Antes de continuar quiero que observes el nivel en el que se encuentran las instrucciones de CPU respecto a los lenguajes de programacion.

![level-programming-lang](./assets/images/cpu-theory/cpu-01.jpg)

<br>

Nota: No todas las instrucciones binarias pueden ser representadas 1:1 como codigo maquina.

En la imagen podemos apreciar una instruccion "add" escrita en ASM (Assembly), esta instruccion tiene un opcode (operation code) el cual representa la operacion que va a realizar, y los demas valores restantes son los que usara.

![level-programming-lang](./assets/images/cpu-theory/cpu-02.png)

La RAM es la memoria principal del ordenador, es un espacio amplio que almacena datos que utilizan los programas que se ejecutan en el ordenador. Esto incluye tnto el codigo del programa en si mismo como el codigo que constituye el nucleo del sistema operativo. 

La CPU siempre lee el codigo maquina directamente desde la RAM, y el codigo no se puede ejecutar si no esta cargado en la RAM.

La CPU almacena un puntero de instruccion que apunta a la ubicacion en la RAM de donde se va a recuperar la siguiente instruccion. Tras ejecutar cada instruccion, la CPU mueve el puntero y repite el proceso, este es el ciclo que ya hablamos anteriormente conocido como fetch-decode-execute (FDE).

Tras ejecutar una instruccion, el puntero avanza hasta situarse despues de dicha instruccion en la RAM, de modo que ahora apunta a la siguiente instruccion. Asi es como se ejecuta el codigo.

El puntero de instruccion se almacena en un registro. Los registros son compartimentos de almacenamiento que la CPU puede leer y escribir con extrema rapidez. Cada arquitectura de CPU tiene un conjunto de registros fijos, que se utilizan para todo.

## Kernel/Supervisor mode and User mode

Usualmente en los procesadores existen dos modos de control (modo privilegiado o anillo), un procesador esta en control de lo que esta permitido hacer. Las arquitecturas modernas usan al menos dos opciones Modo Kernel y modo usuario.

- En el modo kernel: La CPU tiene permisos para ejecutar cualquier instruccion y acceder a cualquier memoria. 

- En modo usuario, solo un subconjunto de instrucciones son permitidas, I/O y la memoria esta limitada.




## Referencias

- [CPU.Land](https://cpu.land/the-basics)

- [Lenovo CPU](https://www.lenovo.com/us/en/glossary/how-does-a-cpu-work/)