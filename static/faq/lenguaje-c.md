---
title: FAQ - Lenguaje C
permalink: /faq/lenguaje-c
---

FAQ - Lenguaje C
=========
{:.no_toc}

* Contenido
{:toc}

## ¿Para qué se usa el tipo size_t?

`size_t` es un tipo entero sin signo devuelto por el operador `sizeof` y es usado para representar el tamaño de construcciones en bytes. Este tipo está definido de manera tal de garantizar que siempre va a poder almacenar el tamaño del tipo más grande posible, por lo que también garantiza que va a poder almacenar cualquier índice de cualquier arreglo.

Estas características lo convierten en el tipo adecuado para manejar tamaños e índices.

## Cadenas

### ¿Cómo comparo si dos cadenas son iguales?
En C las cadenas de caracteres son vectores que tienen caracteres como elementos.  C no sabe comparar vectores, de ningún tipo.  Al hacer una comparación del tipo: `cadena1 == cadena2`, lo que se compara es que las direcciones de memoria de ambas variables sean iguales, es decir que sólo van a ser iguales cuando realmente sean el mismo puntero.

Para poder comparar el contenido de dos cadenas, es necesario usar la función `strcmp(cadena1, cadena2)`, que devuelve 0 si son iguales, menor que 0 si la primera es menor y mayor que 0 si la primera es mayor.  En este caso no importa que las cadenas ocupen o no la misma porción de memoria.

### ¿Cómo copio dos cadenas?

La sentencia:

``` cpp
char* cad_1 = cad_2;
```

No crea una copia de una cadena, sino una copia de la referencia a la cadena. Para hacer una copia de una cadena es necesario hacer:

``` cpp
strcpy(buf_destino, cad_origen);
```

Siendo `buf_destino` una posición de memoria tal que pueda albergar la cadena a copiar, típicamente reservada con `malloc` y `strlen` (teniendo en cuenta el espacio necesario para alojar el fin de cadena).

Desde el curso se recomienda la creación de una función auxiliar `strdup` que encapsule esta lógica.

Pueden leer más sobre sobre cadenas en el [apunte del curso](https://drive.google.com/open?id=1J3uJd0SLZ1DHvPTf7H4ZaKPTwi2mRC0X).

### ¿Qué es getline? ¿Cómo uso getline?

[`getline()`](http://man7.org/linux/man-pages/man3/getline.3.html) es una función que lee una linea de un archivo.

`ssize_t getline(char** buffer, size_t* capacidad, FILE* archivo);`

La principal ventaja de esta función es que automáticamente reserva la memoria dinámica necesaria para almacenar la línea. No se necesita invocar manualmente malloc() a mano, ni preocuparse por los tamaños de los buffers.

Sí se necesita, no obstante, liberar memoria al terminar: getline() llama a malloc() pero transfiere la responsabilidad de la memoria al usuario.

La reserva de memoria dinámica se consigue cuando el puntero buffer apunta a NULL, y el puntero capacidad apunta a 0:

```cpp
char* buffer = NULL; size_t capacidad = 0; 
ssize_t leidos = getline(&buffer, &capacidad, archivo);
// Hago uso de la linea 
free(buffer);
```

La función getline() se encuentra definida en la cabecera stdio.h. Como es una función de POSIX.1-2008, hay que declarar un identificador para indicar que la queremos usar:

```cpp
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
```

Para más información y especificaciones de la función, se puede invocar al comando `man` en la terminal, para ver el manual de esta. `man getline`.

#### ¿Qué es POSIX?

Portable Operating System Interface (POSIX) es una familia de estándares especificados por el Instituto de Ingeniería Eléctrica y Electrónica (IEEE), una asociación sin fines de lucro dedicada a estandarización, para mantener la compatibilidad entre distintos sistemas operativos. 

Al hacer `#include <stdio.h>` lo que le decimos al pre-procesador es que en mi programa yo quiero poder utilizar lo definido por C en esa cabecera de la biblioteca estándar.

Por fuera de la existencia del estándar de C que define y especifica las funciones que utilizamos en la biblioteca estándar (`stdio.h`, `stdlib.h`, etc.), el estándar POSIX lo extiende con funciones y comportamientos nuevos.

Por otro lado, al hacer `#define _POSIX_C_SOURCE 200809L` y luego `#include <stdio.h>` (respetar el orden), lo que le decimos al pre-procesador es que en mi programa yo quiero poder utilizar la especificación de POSIX de 2008 de la biblioteca estándar (de C). Entre otras cosas, esta especificación incluye `getline()` y `getdelim()`.

## ¿Qué es un puntero a función?

Los punteros a función son variables que apuntan a funciones para que puedan ser invocadas sin conocer su nombre. En C, la sintaxis para declararlas es diferente al resto de los tipos. Por ejemplo, dada la siguiente función, que recibe dos cadenas y devuelve un entero:

```c
int mi_funcion(char *a, char *b) {
    return a[0] == b[0];
}
```

Es posible declarar una variable cuyo tipo sea "puntero a función que recibe dos cadenas y devuelve un entero", y asignar a esa variable la dirección de la función:

``` cpp
int (*mi_puntero)(char*, char*);  // Declara la variable (puntero).
mi_puntero = mi_funcion;          // Asigna un valor al puntero.
```

Una vez se ha inicializado la variable correctamente, se puede invocar como si el puntero mismo fuera una función:

```c
int ret = mi_puntero("hola", "adios");
```

Estas funciones se usan típicamente para delegar la ejecución de un fragmento de código a otra función. Por ejemplo, para aplicarle una operación a todos los elementos de un arreglo de enteros podríamos tener la función:

``` cpp
/* Aplica la operación pasada por parámetro a todos los elementos del arreglo. */
void aplicar(int arreglo[], size_t cant, int (*operacion)(int));
```

Entonces si tuviéramos una función para multiplicar por dos:

``` cpp
int multiplicar_por_dos(int numero)
{
    return numero * 2;
}
```

Podríamos multiplicar por dos todos los elementos del arreglo usando un puntero a la función:

``` cpp
int arreglo[3] = { 2, 5, 4 };
aplicar(arreglo, 3, multiplicar_por_dos);
```

## ¿Qué es un wrapper?
En la materia usamos los punteros a función para garantizar un comportamiento que no dependa del tipo de datos que se maneje. Por ejemplo, si quisiéramos destruir todos los elementos de una estructura podríamos crear una funcion:

``` cpp
void estructura_destruir(estructura_t* estructura, void (*f_dest)(void*));
```

que le aplique la función de destrucción a todos los elementos que están almacenados en ella. Si estos elementos también son genéricos y tienen su propia función de destrucción:

``` cpp
void elemento_destruir(elemento_t* elem);
```

No se puede invocar directamente a la primitiva `estructura_destruir` con `elemento_destruir`, porque sus firmas son diferentes. Lo que se suele hacer es crear una función wrapper que enmascare el comportamiento:

``` cpp
void elemento_destruir_wrapper(void* elem) {
    elemento_destruir(elem);
}
```

De esta manera, la función `elemento_destruir_wrapper` es genérica y puede ser usada con la función `estructura_destruir`.

## ¿Qué es typedef?

Typedef es una característica de C que nos permite darle un alias a cualquier tipo de C. Por ejemplo, si quisiéramos representar la edad de una persona podríamos querer abstraernos de si representamos esa edad como un entero, un entero sin signo, un entero corto (`short`), etc. y además tener consistencia en todo nuestro código de usar siempre el mismo tipo para todas las variables que representen una edad. Para esto podemos 'crear' un tipo nuevo en C que se refiera a la edad de las personas, supongamos que como base quisiéramos hacer uso del tipo `unsigned int` para crear nuestro tipo `edad`, pero a su vez queremos dejarle explícito al lector que éste es un tipo definido por el programador, por lo que por convención le agregamos el sufijo `_t` (que se lee como "tipo") siendo nuestro nuevo tipo `edad_t`.

Para esto la sintaxis correspondiente es:

```cpp
typedef unsigned int edad_t;
```

Siempre la sintaxis para definir un nuevo tipo en C _es como_ si se tratara de declarar una variable con determinado nombre, pero anteponiendo la palabra reservada `typedef`. Entonces, en vez de declarar una nueva variable lo que se "declara" es un nuevo tipo, es decir: `typedef tipo_existente mi_nuevo_tipo;`.

Y luego podemos, como si fuese cualquier otro tipo de datos de C, declarar variables, hacer uso de ellas y demás como:

```cpp
edad_t luis = 14;
// Equivale a unsigned int luis = 14
```

La **ventaja** de utilizar `typedef` es que se agrega una capa de abstracción sobre el tipo: en vez de preguntarme qué representa determinada variable de tipo `unsigned int`, el mismo tipo `edad_t` me aclara exactamente para qué sirve ese tipo. _Sintácticamente_ escribir `unsigned int` y `edad_t` en mi código van a ser exactamente lo mismo, pero _semánticamente_ son dos tipos totalmente diferentes y no son intercambiables. Además de la abstracción simplifica la mantenibilidad del código, por ejemplo, si el día de mañana hiciera la asunción de que la edad de las cosas no puede superar los 255 años, tranquilamente podría reemplazar la definición del `typedef` por un `unsigned char` y toda mi implementación se actualizaría consistentemente a esa nueva representación apenas modificando una línea en mi código.

Por otro lado la **desventaja** de utilizar `typedef` es que se enmascaran los tipos básicos. Entonces cuando vemos una variable de tipo `edad_t` tal vez en vez de abstraernos tenemos que ir a buscar su definición para saber si es un tipo entero, una estructura, etc. y cómo definirla o utilizarla. A veces _menos es más_.

`typedef` puede servirnos para otras cosas, como para omitir la palabra `struct` cada vez que hacemos una estructura en C, dándole ahora a nuestras estructuras el rango de tipo:

```cpp
struct persona {
    edad_t edad;
    char *nombre;
};

typedef struct persona persona_t;
```

Cuando se utilizan estructuras para encapsular tipos abstractos de datos suele mantenerse de forma privada la definición de la estructura. En estos casos la definición de la estructura queda en el archivo _.c_ mientras que la definición del tipo se publica en una cabecera _.h_, permitiendo que la interfaz pública del dato use el nombre del tipo.

Si se trata de un `struct` que no se expone públicamente (por ejemplo, por estar limitado a un único _.c_ y declararse ahí), se puede combinar declaración y `typedef` en una sola instrucción:

```cpp
typedef struct {
    edad_t edad;
    char *nombre;
} persona_t;
```

Como ejemplo también podemos hacer uso de `typedef` para simplificar la declaración de punteros a función:

```cpp
// recibe_puntero_a_funcion es una funcion que recibe un puntero
// funcion_recibida es un puntero a funcion que recibe void* y devuelve bool

// Versión A, sin typedef.
int recibe_puntero_a_funcion(bool (*funcion_recibida)(void*));

// Version B, con typedef
// Hago un tipo de funciones, funcionvoidp_t que se refiere a las funciones que
// reciben void* y devuelven bool

typedef bool (*funcionvoidp_t)(void*);

int recibe_puntero_a_funcion(funcionvoidp_t funcion_recibida);
```

## Tipos enumerativos y diccionarios

### ¿Cuándo se usan los tipos enumerativos?

Los tipos enumerativos en C permiten definir un valor _con nombre propio_ entre un conjunto cerrado de casos posibles. Por ejemplo, para modelar los días de la semana, los enumerativos nos permiten crear variables a las que podremos asignarle la etiqueta `LUNES` en vez de convenir asignar `0` o `1` o la letra `'L'`. Cada vez que haya que modelar en qué estado está una variable entre un número finito de posibilidades donde esas posibilidades se identifican con un nombre usaremos tipos enumerativos. (Puede expresarse esto de otra manera: Cada vez que estés inventando valores arbitrarios para categorizar cosas estás haciendo mal las cosas y deberías haber definido un enumerativo.)

La declaración de un tipo enumerativo es simplemente una secuencia de valores:

```cpp
enum reino {ANIMALIA, PLANTAE, FUNGI, BACTERIA, PROTISTA, ARCHAEA};

enum reino especimen_a = ANIMALIA;
```

(Al igual que con el tipo `struct` podríamos haber hecho un `typedef` para redefinir `enum reino` a algo más cómodo y sintético como `reino_t`.)

_Sólo_ porque va a utilizarse más adelante se menciona lo siguiente: Las etiquetas tienen valores numéricos, consecutivos y comienzan por el número cero.

Si bien internamente el tipo `enum` se representa como un entero y los valores de las etiquetas son numéricos en _la mayor parte_ de las ocasiones **está mal** preocuparse por cuánto vale ese valor (sea forzándolo o asignándolo con un literal). También siendo enteros podría asignárseles cualquier valor numérico no incluido entre las etiquetas definidas, esto sería **muy grave** dado que representaría una violación al invariante del enumerativo. Ambas cosas debe evitarse a toda costa. (Y puede expresarse también de otra manera: Si es importante el valor numérico preguntate si no deberías haber elegido una cosa distinta a un enumerativo.)

Cuando utilizamos enumerativos el compilador puede hacer chequeos sobre los valores de los mismos. Por ejemplo, si entráramos en un `switch` con un enumerativo (esto se puede porque `switch` funciona sobre enteros y un enumerativo **es** un entero) el compilador podría avisarnos si nos estamos olvidando de incluir un `case` para un valor puntual del mismo.

Es importante destacar que los `enum` sirven para hacer más claro y legible el código y son una herramienta **para el programador**. Como tales no hay herramientas para convertir, por ejemplo, la etiqueta `ANIMALIA` en la cadena `"ANIMALIA"` ni hay herramientas para leer un enumerativo.

### Diccionarios para traducir enumerativos a cadenas (o a cualquier tipo)

Si necesitáramos traducir entre la representación interna del enumerativo y otras representaciones adecuadas para interactuar con el exterior vamos a aprovechar la propiedad de la numeración de las etiquetas. Si un enumerativo tiene `N` etiquetas sabemos que las mismas serán numéricas y numeradas entre `0` y `N-1` por lo que se hace inmediato aparear enumerativos a arreglos. Es decir, se pueden construir "diccionarios" que _asocian_ un valor arbitrario en algún tipo con un enumerativo según la posición en el mismo del valor.

Por ejemplo:

```cpp
const char *nombres_reino[] = {"animalia", "plantae", "fungi", "bacteria", "protista", "archaea"};

char *reino_a_cadena(enum reino r) {
    return nombres_reino[r];
}
```

Notar que la precondición implícita de la función es asumir que se cumple el invariante del enumerativo, es decir, que el mismo contiene una etiqueta válida de la definición del `enum`.

Si quisiéramos realizar el proceso contrario, esto es, implementar una función `enum reino cadena_a_reino(const char *s);` podríamos iterar comparando los elementos de `nombre_reino` hasta que encontremos la cadena y sencillamente devolver el índice del vector (que es entero y en el rango del enumerativo, por lo que no hay inconveniente en castearlo a tipo `enum reino`). Ahora bien, van a surgir dos complicaciones: la primera es que no tenemos manera de preguntarle al enumerativo cuántas etiquetas posee (esto podría saltearse preguntándoselo al vector con `sizeof(nombres_reino)/sizeof(nombres_reino[0])`); y la segunda es que no sabríamos qué devolver en el caso de que la cadena `s` no coincidiera con ningún elemento de `nombres_reino`. Si bien ambos problemas pueden evadirse, probablemente convenga manejarlos de otra forma.

### Diccionarios para traducir a enumerativo

En el caso de que supiéramos que lo que hay que traducir viene bien formado y ya pasó por un proceso previo de verificación que garantiza la correctitud de lo datos (suele ser una etapa recomendable en cualquier programa) bien podemos hacer el reverso que para traducir de enumerativo a otra cosa:

```cpp
// Precondición: La cadena pertenece a nombre_reino
enum reino cadena_a_reino(const char *s) {
    size_t cantidad = sizeof(nombre_reino)/sizeof(nombre_reino[0]);
    for(size_t i = 0; i < cantidad; i++)
        if(!strcmp(nombre_reino[i], s))
            return i;
}
```

Notar que `i` es de tipo `size_t` y la función devuelve `enum reino`, no hay nada peligroso en este casteo porque por construcción estamos **asegurando** que `i` _entra_ en un `enum reino`. Notar además que no hay un `return` al final de la función, más allá de la precondición en una implementación real podríamos devolver un reino por omisión, por ejemplo `return ANIMALIA;`.

En el caso de que pueda haber una categoría no definida dentro de mi enumerativo, y esto sea algo plausible de ocurrir a tal punto de que _tenga sentido_ incluir ese caso como un caso particular del enumerativo, podemos agregar una etiqueta más al enumerativo que represente el estado indefinido. (Preguntarse: ¿Necesito agregarla o debería resolver ese problema antes de asignar el enumerativo?)

Por ejemplo:

```cpp
typedef enum {OPC_AYUDA, OPC_VERSION, OPC_RECURSIVO, OPC_TODOS, OPC_INDEFINIDA} opcion_t;
const char opciones_letras[] = {'h', 'v', 'r', 'a'};
const char *opciones_cadenas[] = {"help", "version", "recursive", "all"};

opcion_t leer_opcion_letra(char opcion) {
    for(size_t i = 0; i < OPC_INDEFINIDA; i++)
        if(opciones_letras[i] == opcion)
            return i;
    return OPC_INDEFINIDA;
}
```

Notar que el diccionario `letras_opciones` no define valor para `OPC_INDEFINIDA`, esto es porque esa no es una opción que se ingrese sino una condición de error. Notar que agregamos prefijos a las etiquetas de nuestro enumerativo, esto es importante porque dado que las etiquetas se definen internamente _exactamente igual_ que si se hubiera escrito `#define OPC_AYUDA 0`, etc. no puede haber dos etiquetas con el mismo nombre en el programa, para evitar esto es preferible usar prefijos únicos para las etiquetas de un tipo.

### Diccionarios para traducir cosas en cosas

A veces podemos tener que traducir un conjunto finito de cosas que están en un formato A en otro formato B. Un camino para realizar esto puede ser primero convertir de A en un enumerativo adecuado y luego convertir del enumerativo en B. También podemos omitir el paso intermedio sencillamente construyendo un diccionario mediante el apareo de dos arreglos donde coincidan los índices de los elementos equivalentes.

Por ejemplo:
```cpp
char *opcion_letra_a_cadena(char letra) {
    size_t cantidad = sizeof(opciones_letras)/sizeof(opciones_letras[0]);
    for(size_t i = 0; i < cantidad; i++)
        if(opciones_letras[i] == letra)
            return opciones_cadenas[i];
    return NULL;
}
```

Si bien podríamos pensar a `i` como un elemento de `opcion_t` la realidad es que ese fragmento funciona independientemente de la declaración del enumerativo.

Con este enfoque se pueden hacer múltiples arreglos asociados por el índice. Ahora bien los mismos quedan desacoplados en el código y es el programador el que los asocia. Otra manera de resolver el problema es implementando un vector que ya contenga los pares a traducir:

```cpp
struct opcion {
    char letra;
    char *cadena;
};

const struct opcion opciones[] = {
        {'h', "help"},
        {'v', "version"},
        {'r', "recursive"},
        {'a', "all"}
    };

char *opcion_letra_a_cadena(char letra) {
    size_t cantidad = sizeof(opciones)/sizeof(opciones[0]);
    for(size_t i = 0; i < cantidad; i++)
        if(opciones.letra[i] == letra)
            return opciones.cadena[i];
    return NULL;
}
```

### Diccionarios para traducir enumerativos en acciones

Si quisiéramos realizar acciones diferentes en función de los valores de un enumerativo una de las maneras sería implementando un `switch` donde cada `case` _atienda_ una (o varias) de las etiquetas del enumerativo y realice una acción. Si bien esto no está mal puede pasar que el `switch` quede como un bloque monolítico de muchas líneas de código que hacen cosas disímiles entre sí.

Otra forma de resolver esto es implementando un vector de punteros a función asociado a la resolución de cada una de las opciones, es decir:

```cpp
// Manejadores de las acciones:
bool accion_ayuda();
bool accion_version();
bool accion_recursivo();
bool accion_todos();

typedef bool (*manejador_t)(void);

bool ejecutar_accion(opcion_t opcion) {
    manejador_t manejadores[] = {accion_ayuda, accion_version, accion_recursivo, accion_todos};

    if(opcion == OPC_INDEFINIDA)
        return false;

    return manejadores[opcion]();
}
```

En este caso se cambia el enfoque monolítico por un enfoque modular con funciones que saben hacer cada acción. Este esquema _requiere_ que todas las funciones tengan la misma firma. En el caso de que las funciones por construcción necesiten parámetros no homogeneos puede forzarse en la interfaz que los mismos se enmascaren de una forma homogenea (como por ejemplo pasándolos como `void *`). También pueden declararse diccionarios auxiliares que contengan más información, como por ejemplo el número de los argumentos de cada manejador (o declarar un único diccionario que encapsule toda esa información en un `struct`). En el caso de que esto sea demasiado engorroso, se recomienda implementar un `switch` y no hacer uso de diccionarios.
