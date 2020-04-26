---
layout: post
title: Hacia un sistema de memoria escalable y compatible con RAII
date: 2020-04-26
tags:
  programación
---
<p style='text-align: justify;'>Reservar memoria, menuda movida. Es un problema tan difícil como importante, como se puede ver en el hecho de que la forma en la que se resuelve es un elemento bastante céntrico en el diseño de un lenguaje de programación, y hasta ahora se han dado muchas soluciones pero ninguna totalmente satisfactoria.</p>

<p style='text-align: justify;'>C fue el primero en vérselas con el problema y decidió, en algo que es muy de C, dejarle el marrón al programador. Aquí tienes free, acuérdate de llamarlo. C++ fue un gran paso adelante con la invención del destructor, esa función mágica que se llama cuando un objeto es destruido y que se acuerda de llamar a free por ti. El problema de C++ vino al intentar escalar esta solución a contenedores genéricos, que personalizan el acceso a la memoria dinámica mediante el uso de allocators. Este diseño permite sin coste de ejecución adicional una personalización total de cómo pide memoria un contenedor, pero a cambio escribir contenedores se convirtió en un arte arcano sólo al alcance de magos del código de nivel 10. D hereda el modelo de allocators de C++ y si eso añade más magia que, sí, hace al diseño aún más poderoso y flexible pero es que no hay dios que escriba un contenedor con esto.</p>

<p style='text-align: justify;'>Lenguajes como Java, C# o Haskell han optado por ocultar al programador que existe la memoria dinámica en primer lugar, y se encargan de todo con un sistema de recolección de basura. Esto está muy bien porque simplifica muchísimo la escritura de código, pero elimina al programador la posibilidad de controlar uno de los aspectos más críticos para la velocidad de un programa. Si tienes suerte y un recolector de basura es el modelo que mejor se adapta a tu programa enhorabuena, estás de suerte. Si no, mala suerte y haber elegido otro lenguaje. Esto es especialmente notable en juegos, donde el coste del recolector de basura, que puede tardar el tiempo de varios fotogramas en hacer su trabajo cuando está sobrecargado, es totalmente inaceptable.</p>

<p style='text-align: justify;'>Por último, tal vez la solución más creativa últimamente a este problema la haya dado Jai, el lenguaje de Jonathan Blow, que propone una pila de allocators. Al reservar o liberar memoria, se usa el allocator en lo alto de la pila. Una función que devuelve nueva memoria lo hace con el allocator que estaba en lo alto de la pila al llamar a la función. Esto está muy bien porque permite personalizar muy fácilmente la forma en la que el programa reserva memoria sin complicar el código que se encarga de manejar esa memoria lo más mínimo, pero tiene un problema. No es del todo compatible con RAII, ya que es bastante improbable, excepto en casos muy triviales, asegurarse de que el allocator que estará en lo alto de la pila cuando el objeto que maneja memoria sea destruido sea el mismo que reservó esa memoria. Esto se complica todavía más si tenemos objetos que viven en memoria durante mucho tiempo, que son globales o que viajan entre threads. Tal vez esto no le importe mucho a Jai, un lenguaje que ni siquiera tiene destructores, pero para la mayor parte de los programadores un sistema de memoria que no es compatible con RAII y por lo tanto no es automatizable es inadmisible hoy en día.</p>

<p style='text-align: justify;'>Tal vez se pueda, sin embargo, construir sobre esta idea para hacerla compatible con RAII. Para esto, hay que liberar a la pila de allocators de la responsabilidad de liberar memoria. La pila de allocators sólo se encargará de reservarla. En vez de eso, cada bloque de memoria reservado guardará la función que se tiene que usar para liberar esa memoria, de forma similar a la que usan algunas implementaciones de malloc para guardar el tamaño del bloque.</p>

![Visualización](https://raw.githubusercontent.com/asielorz/blog/master/images/memoria-free.png)

<p style='text-align: justify;'>Esto permite disociar el estado de la pila de allocators del momento en el que se tiene que liberar la memoria, de forma que el manejo de la pila se limita únicamente a la selección de la función de reserva de memoria a usar, simplificando mucho el impacto que puede tener en la corrección del programa. Liberar memoria puede de nuevo automatizarse en destructores. Lo único que hay que hacer es leer del bloque la función que hay que usar para liberar la memoria y luego liberar el bloque usando esa función. Simple.</p>

<p style='text-align: justify;'>Otra ventaja de este sistema, con respecto a los allocators de C++, es que el manejo de memoria deja de ser parte del tipo. Esto, además de reducir el número de instanciaciones de templates en un programa que uso muchos allocators distintos, permite a contenedores con diferentes fuentes de memoria interoperar. En C++ ahora mismo no es posible copiar dos vectores con diferente allocator con el operador =. En este sistema sería trivial.</p>

<p style='text-align: justify;'>También nos permite tener muy fácilmente allocators que no pueden liberar. Esto es muy útil cuando hablamos de pilas de memoria temporal que son vaciadas enteras de vez en cuando. Para esto, sólo necesitamos usar de función de liberar memoria una que no haga nada, y listo. Incluso se podría optimizar usando alguno de los bits libres del pointer para marcar que no tiene memoria que liberar y así poder ahorrarse escribir la función en bloque de memoria.</p>

En resumen, el sistema nos queda algo así.
- Cada thread tiene una pila de allocators.
- Para reservar memoria se usa el allocator en lo alto de la pila.
- Cuando se reserva un bloque de memoria, se escribe también al bloque la función que se tiene que usar para liberar esa memoria.
- Al salir de una función la pila de allocators tiene que estar en el mismo estado que cuando se entró.
- Una función que devuelve memoria dinámica tiene que haber reservado esa memoria con el allocator en lo alto de la pila al entrar a la función.
- Pueden existir allocators que no pueden liberar memoria. En estos casos intentar liberar el bloque de memoria no hace nada y es responsabilidad del programador asegurarse de que esa memoria es liberada correctamente por otros medios.

**Segunda parte: La interfaz de las funciones allocate y deallocate**

Las interfaces de malloc y free son algo mejorables.

```cpp
void * malloc(size_t size);
void free(void * ptr);
```

<p style='text-align: justify;'>En primer lugar, malloc no sabe cuáles son los requerimientos de alineamiento de la memoria a reservar. En la mayoría de sistemas, la memoria estará alineada al menos a 16 bytes, lo cual es suficiente para la mayoría de los tipos, pero debería ser posible especificar este número en cualquier caso. Es cierto que C11 introduce aligned_alloc que resuelve este problema.</p>

```cpp
void * aligned_alloc(size_t alignment, size_t size);
```

<p style='text-align: justify;'>Otro gran problema es que free no coge el tamaño del bloque. Para poder hacer esto, malloc escribe el tamaño al principio del bloque, antes del pointer que se le da al usuario. Sin embargo, en la mayoría de los casos (por no decir todos) el tamaño del bloque a liberar es conocido por el programa en el momento en el que se llama a free, de forma que esta información podría pasarse como parámetro en lugar de escribirla. Esto ahorraría memoria y además simplificaría al usuario el escribir sus propios allocators, ya que casi cualquier allocator que emule la interfaz de free va a estar obligado a repetir este patrón de escribir el tamaño antes del bloque.</p>

<p style='text-align: justify;'>Por último, hay muchos sistemas de reserva de memoria que funcionan por bloques de tamaño fijo. Se divide un gran trozo de memoria en bloques iguales de tamaño fijo, y cuando el usuario pide memoria se le da el bloque más pequeño que sea de un tamaño igual o mayor a la memoria solicitada. Esto significa que hay allocators que pueden devolver memoria de más, y puede ser útil hacer saber esta información al usuario. Por ejemplo, supongamos que tenemos un <code>vector&lt;int&gt;</code> en el que se hace <code>reserve(13)</code>. El vector pedirá un bloque de memoria de 13x4=52 bytes. Sin embargo, supongamos que en nuestro allocator el bloque más pequeño que puede albergar 52 bytes es de 64. Si podemos comunicar esta información de vuelta al vector, este puede establecer su capacidad en 16 en vez de en 13, ganando 3 ints más gratis, ya que ya estaban ahí de todas formas.</p>

<p style='text-align: justify;'>Por lo tanto, la interfaz de las funciones de reservar y devolver memoria nos quedan así.</p>

```cpp
pair<void *, size_t> allocate(size_t size, size_t alignment);
void deallocate(void * ptr, size_t size, size_t alignment);
```