# mDynamic.h

Antes de leer esta documentación se recomienda visitar este repositorio: [mDynamic.h](https://github.com/MrDave1999/mDynamic.h).

Anteriormente se había explicado de como declarar un array de tipo "union" pero como variable global.

## <a name="indice"></a> Índice
- [Parámetro de tipo union donde la función no reserva directamente la memoria de una matriz](#params1)
- [Parámetro de tipo union donde la función si reserva directamente la memoria de una matriz](#params2)
- [Parámetro de tipo NO union ](#params3)

## <a name= "params1"></a> Parámetro de tipo union donde la función no reserva directamente la memoria de una matriz

Antes hacíamos esto:
```C
#define MAX_MATRIX 1
#include <mDynamic.h>

enum { MA }; //id de la matriz

union ptrs
{
	int** pa;
};

union ptrs mx[MAX_MATRIX]; //Array global

int main(void)
{
  //code
  return 0;
}
```
¿Qué pasaría sí lo declaramos como local? ¿Cómo sería el algoritmo para poder modificar el contenido de cada elemento de la matriz?
Así quedaría la estructura:
```C
#define MAX_MATRIX 1
#include <mDynamic.h>

enum { MA }; 

union ptrs
{
	int** pa;
};

int main(void)
{
	union ptrs mx[MAX_MATRIX] = {NULL};
	return 0;
}
```
Se debe inicializar todos los elementos del arreglo "mx" a NULL, porqué son punteros dobles y con eso detectamos que el puntero no apunta a nada. Hacer este tipo de inicialización es buena práctica, la tendrás que usar sí quieres que la macro `REALLOC_ROWS` actúe como `ALLOC_RC`.

La pregunta del millón, ¿cómo modificó/accedo a cada elemento de la matriz después de la asignación de memoria?
Tendríamos algo así:
```C

void printdat(union ptrs*const mx)
{
	unsigned int i, j;
	for (i = 0; i != getrows(MA); ++i)
	{
		for (j = 0; j != getcols(MA); ++j)
			printf("%d\t", &mx[MA].pa[i][j]);
		printf("\n");
	}
}
```
En ese código de ejemplo que muestra la función, únicamente imprime las direcciones de memoria de cada dato/objeto de la matriz.
Con esa función podríamos modificar, acceder a los elementos de dicha matriz, claro, antes de haber reservado memoria para la matriz.
Nos damos cuenta también que la subrutina tiene un parámetro que básicamente es un puntero de un nivel de direccionamiento indirecto (a esto me refiero que el puntero apunta hacia una dirección de memoria) y que nos servirá para poder acceder a cualquier contenido de la matriz dinámica.

El puntero "mx" recibe la dirección de memoria del primer elemento del arreglo tipo union y a partir de esto, podemos lograr usar esta sintaxis:
```C
&mx[MA].pa[i][j]
```
Ahora debemos llamar la función en el "main"; pero antes de eso debemos reservar memoria para la matriz A:
```C
int main(void)
{
	union ptrs mx[MAX_MATRIX] = {NULL};
	setrows(MA, 3)
	setcols(MA, 2)
	ALLOC_RC(MA, mx, pa)
	printdat(&mx[0]);
	FREE_MEMORY_ALL(mx, pa)
	return EXIT_SUCCESS;
} 
```
Podemos notar el argumento que le pasamos al parámetro "mx" de la función "printdat"; sin embargo, ¿qué pasaría si le quito el ampersand a ese argumento?
```C
printdat(mx[0]);
```
¿La función funcionaría igual? ¿Compilaría? Lo más probable es que nos dé como mínimo una advertencia y es evidente que imprimirá direcciones de memoria que no son las que buscamos. Así que ¡Cuidado!

[Volver al índice](#indice)

## <a name= "params2"></a> Parámetro de tipo union donde la función si reserva directamente la memoria de una matriz

Modificaremos esa función y ahora haremos que la subrutina/función reserve la memoria para la matriz A.
Código:
```C
unsigned char printdat(union ptrs*const mx)
{
	unsigned int i, j;
	setrows(MA, 3)
	setcols(MA, 2)
	ALLOC_RC(MA, mx, pa)
	for (i = 0; i != getrows(MA); ++i)
	{
		for (j = 0; j != getcols(MA); ++j)
			printf("%d\t", &mx[MA].pa[i][j]);
		printf("\n");
	}
	return EXIT_SUCCESS;
}
```
La macro `ALLOC_RC` obliga que la función retorne un valor para poder saber si hubo alguna falla en la asignación de memoria para la matriz A. En este caso el tipo de valor de retorno de la subrutina es `unsigned char` (ocupa sólo 1 byte).
Al momento de llamar la función en `int main` sería así:
```C
int main(void)
{
	union ptrs mx[MAX_MATRIX] = {NULL};
	if (printdat(&mx[0]) == EXIT_FAILURE)
	{
		getchar(); //Pausa el programa 
		return EXIT_FAILURE;
	}
	FREE_MEMORY_ALL(mx, pa)
	return EXIT_SUCCESS;
}   
```
La restricción `printdat(&mx[0]) == EXIT_FAILURE` es necesario, ya que con eso descubrimos sí hubo una falla o no en la asignación de memoria.

[Volver al índice](#indice)

## <a name= "params3"></a> Parámetro de tipo NO union 

¿Sería posible que el parámetro "mx" apunte a un "integer"? Es posible, claro. La desventaja es que sí lo haces de esta forma deberás reservar memoria para la matriz A antes de llamar dicha función. 
El código quedaría así:
```C
void printdat(int**const pa)
{
	unsigned int i, j;
	for (i = 0; i != getrows(MA); ++i)
	{
		for (j = 0; j != getcols(MA); ++j)
			printf("%d\t", &pa[i][j]);
		printf("\n");
	}
}
```
Claro, antes usabas esta expresión para acceder al contenido de la matriz:
```C
mx[MA].pa
```
Ahora ya no se puede, porqué esa expresión sólo es válida cuando el puntero apunte a `ptrs`.
Por esa razón usamos esta expresión:
```C
pa[i][j]
```
El puntero "pa" no puede recibir la dirección de memoria del primer elemento del array tipo union, sino recibirá el contenido de dicha dirección y eso debe, porqué sólo necesitamos la dirección de memoria del primer elemento del búfer de punteros de la matriz A (el búfer de punteros básicamente es donde se guarda las direcciones de memoria del primer elemento de cada columna de dicha matriz).
Al momento de llamar la función en el "main", quedaría así:
```C
int main(void)
{
	union ptrs mx[MAX_MATRIX] = {NULL};
	setrows(MA, 3)
	setcols(MA, 2)
	ALLOC_RC(MA, mx, pa)
	printdat(mx[0].pa);
	FREE_MEMORY_ALL(mx, pa)
	return EXIT_SUCCESS;
}   
```
Date cuenta que en ningún momento se le agrega el ampersand a `mx[0].pa`. Estaría erróneo sí llega a suceder y eso se debe porqué estarías pasando la dirección de memoria del puntero doble (en este caso es el primer elemento del arreglo) y lo que necesita el parámetro realmente es, la dirección de memoria al que apunte dicho puntero doble. 

[Volver al índice](#indice)
