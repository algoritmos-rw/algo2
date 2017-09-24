---
layout: page
title: TP1: Wachencoin
permalink: '/tps/tp1'
---


![](tp1logo.png)

Trabajo práctico 1
================    


El trabajo práctico está divido en dos partes, si bien están relacionadas:

* implementación de funciones auxiliares para strings (`strutil`)
* implementación de un sistema similar a Bitcoin (*Wachencoin*)


strutil
-------
explicación normal de lo que tienen que utilizar    

Interludio: Bitcoin
-------------------   
**Bitcoin** es un sistema de pagos digital descentralizado que no requiere un
respaldo externo (en billetes, por ejemplo) del dinero que utiliza.

En Bitcoin y sus derivados, los pagos son en realidad programas (*scripts*)
que se escriben en un *lenguaje de pila*.    

Un lenguaje de pila opera básicamente de dos formas: apilando y desapilando
datos del stack, para poder usar funciones sobre ellos. Un ejemplo puede ser
la polaca inversa que se utilizaba en las antiguas calculadoras digitales.  

Aunque Bitcoin tiene una arquitectura compleja para garantizar transferencias
seguras e instantáneas entre miembros de la red, en este TP sólo nos interesan
las estructuras de datos que pueden ser útiles en una red de este tipo.

Wachencoin
----------    

Vamos a implementar un programa que mantiene un estado de los 
saldos de las billeteras de algunos clientes y permite que se envíen pagos
entre ellos utilizando un lenguaje de pila¹.   

*Curiosidad*. La unidad más pequeña de las bitcoins se llama *satoshi*, por
su creador. Parece justo, entonces, llamar a la pequeña unidad de una 
Wachencoin un *rosita*.  

En Wachencoin, los programas tienen la siguiente forma:    

```
<id_usuario>
<coordenadas>
validar_usuario
<id_usuario>
<dinero>
validar_pago
<dinero>
<user_id_2>
<user_id>
pagar
```     
(**nota**: arriba está escrito en varias líneas por claridad; en realidad
el programa se escribe en una sola línea separada por el caracter `;` -- ver
ejemplos abajo.)  

Para clarificar, marcamos las siguientes operaciones:    

* `validar_usuario`: desapila dos elementos y revisa que las coordenadas
(contraseña) sean apropiadas para el usuario. 
* `validar_pago`: desapila dos elementos y revisa que el usuario tenga
fondos suficientes para hacer la operación. 
* `pagar`: desapila tres elementos y hace el movimiento de fondos. 
* por defecto: empuja un dato al stack.

**Atención**: Si bien la de arriba es una transacción bien escrita, no es
la *única* posible. También habrá transacciones que hacen pagos a múltiples
personas; o de varias personas a una sola.   

Además, cada instrucción puede **terminar** la ejecución del programa si 
devuelve algún error. Esto es particularmente importante para las funciones
de validación; si algún chequeo no se cumple, entonces se termina el 
programa y no se realiza el pago.   

Es necesario implementar una **interfaz** del programa, que leerá por entrada 
entrada estándar los siguientes comandos:       

* `agregar_pago <id> <monto> <código>`, que agregará a la cola de 
procesamiento la transacción. El código se manifiesta en una sola línea, 
separando las distintas instrucciones con el caracter punto y coma (`;`). 
* `pagos_pendientes`: devuelve la cantidad y monto total de los pagos sin 
procesar. 
* `procesar <numero>`: procesa hasta un número definido por parámetro de pagos
de los pendientes. Si algún pago falla, se debe avisar. 
* `guardar_cuentas <archivo>`: guarda en un archivo el estado *actual* de las
cuentas. 
* `finalizar`: finaliza el programa (si quedan pagos sin procesar, no se 
procesan).


El estado inicial de las cuentas (número, saldo, coordenadas) se carga de un 
archivo pasado por parámetro al programa, que tiene el siguiente formato:   

```
0, 13.37, 14f6c9dae22
1, 58.92, 916f4c31aaa
2, 9301.92, e9a7f54270d
(...)
```  
Los números de cuenta son consecutivos empezando desde 0.   


**Restricciones sobre la complejidad**: todas las operaciones del lenguaje de 
pila deben ser $O(1)$. La operación de agregar un pago debe ser $O(1)$, la de
pagos pendientes $O(P)$ siendo P la cantidad de transacciones sin procesar, 
y la de guardar cuentas debe ser $O(C)$ (con C la cantidad de cuentas).   


### Ejemplos    

a completar luego de haberlo implementado

Más información y links
------------------------   

Esta implementación de *Wachencoin* es bastante sencilla pero introduce 
algunos conceptos fundamentales de criptomonedas. A continuación se enumeran
recursos que permiten entender otros conceptos involucrados no incluídos en
el TP y para nada necesarios para su resolución:

* [Double spending problem](https://en.wikipedia.org/wiki/Double-spending)
* [Funciones de hash SHA](https://es.wikipedia.org/wiki/SHA-2)
* [Arbol de Merkle](https://en.wikipedia.org/wiki/Merkle_tree)
* [Criptografía de clave pública](https://www.youtube.com/watch?v=GSIDS_lvRv4)
* [Criptografía con RSA](http://pub.gajendra.net/2012/09/an_explanation_of_the_rsa_cryptosystem)
* [Majority Rule](http://hackingdistributed.com/2014/06/19/bitcoin-and-voting-power/)
* [Ejemplos de programas en Bitcoin Scripting Language](https://en.bitcoin.it/wiki/Transaction#Pay-to-PubkeyHash)   


-----

¹ el atento lector se habrá dado cuenta de que no es necesario utilizar un
lenguaje de pila para realizar los pagos. Sin embargo, el propósito del TP
es *implementar* ese lenguaje de pila, y utilizarlo para realizar pagos 
-- no al revés.