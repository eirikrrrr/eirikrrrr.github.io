---
date: 2026-03-31
categories:
  - Topicos
tags:
  - Hardware
  - Conceptos
  - Spanish
---


# CPU | Fundamentos Basicos (Resumido).

La CPU (Unidad Central de Procesamiento) no es un bloque monolítico, sino un sistema con subsistemas que se comunican constantemente, **es el cerebro** de la operacion.

![CPU-Overview](./assets/images/cpu-theory/cpu_architecture_overview_v2.svg)

## 1. Ciclo de instrucción

Todo lo que hace una CPU se reduce a repetir un ciclo llamado fetch-decode-execute.

- **Fetch**: la CU (Control Unit) lee la siguiente instrucción desde la memoria RAM usando el Program Counter (PC), un registro que apunta a la dirección actual. La instrucción viaja por el bus de datos hasta el Instruction Register (IR).

- **Decode**: la CU interpreta el código binario y determina qué operación ejecutar y con qué operandos.

- **Execute**: la CU activa los elementos necesarios como: ALU (operaciones matematicas, suma, resta, etc), acceso a memoria, actualización de registros.

## 2. ALU y registros

La ALU (Arithmetic Logic Unit) ejecuta operaciones aritméticas y lógicas. En la mayoría de la arquitectura no opera sobre RAM directamente; opera sobre registros internos, pequeñas celdas de 64 bits dentro de la CPU.

Componentes relevantes:

- PC (Program Counter): dirección de la siguiente instrucción.
- IR (Instruction Register): instrucción actual.
- MAR (Memory Address Register): dirección de memoria a leer/escribir.
- MDR (Memory Data Register): dato transferido desde/hacia RAM.

Registros de propósito general: AX, BX, CX, etc.

La FPU (Floating Point Unit) es la unidad para operaciones de punto flotante (float, double). Antes era un co-procesador separado (ej. 80387), hoy está integrado en el mismo chip de CPU.

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