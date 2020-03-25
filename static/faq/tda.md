---
title: FAQ - Tipos de Datos Abstractos
permalink: /faq/tda
math: true
---

FAQ - Tipos de Datos Abstractos
=========
{:.no_toc}

* Contenido
{:toc}

## Abstracción

### ¿Qué es el tipo void*?

El tipo `void*` es un tipo que representa un puntero a un tipo no especificado. Es usado cuando el tipo apuntado no es de importancia para la función que la recibe.

Este tipo de punteros puede ser convertido a cualquier otro puntero sin necesidad de un casteo explícito, pero debido a que no se sabe a qué apunta no puede ni desreferenciarse ni ser usado para calcular aritmética de punteros.

### ¿Qué es una estructura en C?

Una estructura en C es un tipo de variables que nos permite almacenar distintos elementos dentro de ellas. Sirven para encapsular lo que refiere a una variable que contendrá distintos atributos. Su uso es con la palabra reservada de C `struct`. Por ejemplo, para una entidad de persona, con edad y nombre, se define la estructura:

```cpp
struct persona {
    unsigned int edad;
    char* nombre;
    int* dni;
}
```

Ahora, ya sabemos que el `struct persona` contiene una edad del tipo `unsigned int` y una cadena para el nombre. También contiene un puntero al dni, que es un entero.

Para acceder a estos miembros de la estructura, se puede acceder tanto al atributo mismo, con el operador punto `juan.nombre` como a lo que apunta un atributo que sea un puntero, con el operador `->`, `juan->dni`.

### ¿Qué significa el error dereferencing pointer to incomplete type?

Este error se da cuando se trata de acceder a un miembro de una estructura de la cual no se tiene información.  Si en el archivo .h aparece únicamente:

``` cpp
typedef struct _mitipo mitipo;
```

No va a ser posible acceder a los miembros internos de esa estructura, ya que es un tipo incompleto.  Sobre las razones de por qué esto está bien en el caso de un TAD, ver la siguiente pregunta.

### ¿Por qué no puedo acceder a un miembro de una estructura de un TAD?

Los miembros de las estructuras (como `pila->cantidad`, por ejemplo) son datos internos de la implementación de cada tipo abstracto de datos, a los que no se debe acceder fuera del archivo `.c` en el que se encuentran las primitivas correspondientes al TAD.

Este es el principio básico de abstracción, que nos permite operar con una pila, una lista o un árbol sin tener en cuenta cómo están implementados, y que si la implementación cambia, el código que utiliza el TAD sigue funcionando sin necesidad de ser modificado.

Cuando se realizan pruebas sobre un TAD para verificar su comportamiento, no es necesario acceder a los miembros de la estructura, ya que lo que se desea verificar es que se comporte de la forma esperada y no cuáles son los valores utilizados internamente por la implementación.  Las pruebas deben funcionar aún si la implementación interna cambia (siempre que se respeten las primitivas).

## Redimensionamiento automático

Estas respuestas son útiles tanto para la pila dinámica, como para la tabla de hash, o cualquier otra estructura que deba agrandarse y achicarse automáticamente.

### ¿Cuánto conviene agrandar una estructura dinámica?

Hacer un realloc es una operación lenta: $$\mathcal{O}(n)$$ (siendo n la cantidad de bytes que se mantienen en el bloque de memoria).  De modo que es importante planificar cuánto se va a agrandar la estructura, para que el tiempo gastado en el redimensionamiento no influya en el comportamiento general de la estructura.

Para ello, lo que se hace es que al agrandar, el espacio utilizado se multiplique -generalmente por 2, pero puede ser por otro factor, según las condiciones del problema a resolver-, de modo que la incidencia del realloc se distribuya siempre de forma pareja entre los elementos, lo que no sucede en el caso de que la estructura crezca una cantidad constante. Una demostración formal sobre cómo afecta esta redimensión en el costo total de las operaciones puede encontrarse en la [lista de mails](https://groups.google.com/d/msg/fiuba-7541rw-alu/t86CJcNv2UU/Irb7J899DgAJ). 

### ¿Cuándo y cuánto conviene achicar una estructura dinámica?

Si para agrandar la estructura multiplicamos su tamaño por 2, no es una buena idea reducirla a la mitad ni bien llega a la mitad de espacio utilizado, ya que en el caso de que se quisiera agregar un nuevo elemento, sería necesario agrandarla y si se lo quita, otra vez achicarla, por lo que en definitiva, habría una cantidad de elementos para la que las operaciones de agregar o sacar de la estructura serían particularmente lentas.

Es por eso que lo que se puede hacer es reducir la estructura a la mitad, pero sólo cuando la cantidad de espacio ocupada llega a 1/4 de la deseada.  De este modo, se libera la memoria fuera de uso, pero sin el riesgo de caer en un caso patológicamente lento.

Tanto en el caso de agrandar como el de achicar, se pueden tomar criterios distintos para buscar mayor eficiencia en tiempo o en memoria, según el problema a resolver.


## Iteradores

### ¿Qué es un iterador?

Es una construcción que nos permite recorrer los elementos almacenados dentro de una estructura en cierto orden. Por ejemplo, en un arreglo:

``` cpp
double arr[TAM_ARR];
for (size_t i = 0; i < TAM_ARR; i++) {
    // Hacer algo con arr[i]
}
```

La variable `i` hace de iterador del arreglo `v`.

En listas enlazadas, árboles, tablas de hash y otras estructuras ese `size_t i` no es suficiente para recorrer la estructura, por lo que se generaliza esa idea en el concepto de "iterador".

Las operaciones básicas que puede tener un iterador son:
  - crear
  - ver_dato
  - avanzar
  - ¿finalizó?
  - destruir

### ¿Cómo se usan los iteradores de una estructura?

En la materia adoptamos la siguiente convención para todos los iteradores:
  - La creacion de un iterador en una estructura es siempre una operación válida:
  - Si la estructura tiene elementos, el iterador apunta al primer elemento.
  - Si la estructura no tiene elementos, el iterador se crea, pero apuntando al final.
  - Avanzar un iterador cuando no está al final siempre es una operación válida.
  - Avanzar un iterador que está al final es una operacion inválida, por lo que devuelve false.
  - Ver actual es siempre una operación válida. Cuando está al final devuelve siempre `NULL`.

Considerando esta convención, la forma canónica de iterar una estructura es la siguiente:

``` cpp
estructura_iter_t* iter = estructura_iter_crear(estructura);

while(!estructura_iter_finalizo(iter))
{
    void* dato = estructura_iter_ver_actual(iter);
    /* Usar dato. */

    estructura_iter_avanzar(iter);
}

estructura_iter_destruir(iter);
```

O usando la sintaxis de ciclos definidos de C:

``` cpp
estructura_iter_t* iter;

for(iter = estructura_iter_crear(estructura);
  !estructura_iter_finalizo(iter);
  estructura_iter_avanzar(iter))
{
    void* dato = estructura_iter_ver_actual(iter);
    /* Usar dato. */
}

estructura_iter_destruir(iter);
```

Se debe tener en cuenta que si bien `estructura_iter_t* iter` puede ser inicializado en el `for` (como en el ejemplo) o fuera del mismo (al ser declarada la variable), la declaración debe estar siempre fuera del `for` para no perder la referencia y poder de esta forma destruirlo cuando se termina de iterar.

### Una vez que se llega al fin, ¿El iterador no sirve más?

En efecto, los iteradores son descartables, una vez que se termina de recorrer la estructura, sea porque está vacía o se llegó al final, el iterador termina su vida útil, hay que destruirlo y crear uno nuevo.

## Destrucción de datos

### Si tengo una estructura que guarda datos de distintos tipos, ¿Cómo hago para destruirlos?

Cuando una estructura, por ejemplo una lista, se utiliza en un mismo programa para guardar distintos datos, se los almacena como `void*` y es necesario castearlos cuando se los quiere acceder.  En particular es interesante el caso de la función de destrucción, que está definida como `void destruir_dato(void*)`.

Supongamos que se tiene una lista que contiene `mensaje_t*` y otra que contiene `char*`.  Las funciones de destrucción  correspondientes deberán recibir `void*` en ambos casos, y por dentro hacer los casteos:

``` cpp
void destruir_dato_char(void* dato) {
   char* dato_c = dato;
   free(dato_c);
}

void destruir_dato_mensaje(void* dato) {
   mensaje_t* dato_m = dato;
   mensaje_destruir(dato_m);
}
```

O sea, para cada tipo de dato, se lo interpreta para liberarlo según corresponda.

### Si una lista guarda listas, ¿cómo se hace para destruirla?

Como para cualquier otro tipo de listas, para destruir una lista de listas, el usuario que está usando la estructura deberá definir una función para la destrucción de las listas contenidas dentro de la otra lista.  Suponiendo que las listas de adentro contienen mensajes, podría ser de la siguiente forma:

``` cpp
void destruir_dato_lista(lista_dato_t dato) {
    lista_t* lista = (lista_t*) dato;
    lista_destruir(lista, destruir_dato_mensaje);
}
```

### ¿Qué se debe hacer cuando el tipo de datos no necesita ser destruido?

Si dentro del código de la estructura de datos se verifica que la función se haya recibido:

``` cpp
if (destruir_dato) destruir_dato(dato);
```

Se puede llamar a la función de destrucción simplemente pasando `NULL` como parámetro:

``` cpp
lista_destruir(lista, NULL)
```

### ¿Por qué no reciben la función de destrucción las funciones que eliminan de a uno?

En varias estructuras, como listas, colas, pilas o árboles, a la función de destruir se le pasa una función para destruir el dato, mientras que a la función de borrar (desapilar, desencolar, etc) no se le pasa esta función.

Esto se debe a que las funciones que eliminan elementos de las estructuras no los tienen que destruir, ya que los guardan en la referencia al dato recibida por puntero, y es el usuario el encargado de liberar el dato de ser necesario.
