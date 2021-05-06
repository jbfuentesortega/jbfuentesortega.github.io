---
layout: post
title: 'Copy-on-write no se compone muy bien'
date: 2021-5-6
tags:
  informática
  programación
---
<p style='text-align: justify;'>Copy-on-write es un patrón muy útil para optimizar copias y compartir memoria entre estructuras de datos muy parecidas sin comprometer la relación parte-todo de estas estructuras. Es decir, haciendo que el hecho de que parte de la estructura es compartida sea un detalle de implementación invisible y no una propiedad de la estructura. Esta es una propiedad similar a la que consiguen arquitecturas funcionales con las llamadas estructuras de datos persistentes, que explotan la inmutabilidad para poder compartir gran parte del contenido de contenedores casi idénticos. El concepto es simple. Se le añade al objeto un contador de referencias. Además, cada vez que se va a llevar a cabo una operación que va a mutar el objeto, se comprueba el contador de referencias, y si es mayor que uno, se crea una copia. Copy-on-write tiene además la ventaja, para lenguajes como C++, de poder hacerlo con una sintaxis más favorable al estilo procedural, además de ser relativamente fácil de implementar de forma genérica. Es decir, podemos definir un tipo <code>CopyOnWrite&lt;T&gt;</code> para todo tipo regular <code>T</code>, que es isomórfico con <code>T</code> e implementa esta optimización. Además, esta interfaz permite ahorrar operaciones costosas como reservas de memoria en situaciones en las que el objeto pertenece a una única referencia, que la contraparte funcional no puede evitar.</p>

<p style='text-align: justify;'>La gran desventaja de copy-on-write con respecto a la interfaz puramente funcional nos la encontramos cuando queremos hacer una estructura compuesta por tipos copy-on-write, que sea asimismo copy-on-write, es decir, cuando queremos tener una estructura jerárquica compuesta por tipos copy-on-write. Esto supone dos problemas principales. En primer lugar, comprobar si un objeto está siendo referido por una sola referencia deja de ser trivial, ya que hay que tener en cuenta los contadores de referencias de los nodos superiores en toda la jerarquía hasta la raíz.</p>

![Estructura](https://raw.githubusercontent.com/asielorz/blog/master/images/copy-on-write-1.png)

<p style='text-align: justify;'>En el dibujo de arriba, podría parecer que los nodos F y G pertenecen a un solo objeto, ya que su contador de referencias es 1. Sin embargo, existen dos accesos hasta F y G desde el exterior porque, aunque a ambos sólo se puede acceder desde E, a E se puede acceder desde dos sitios, A y H.</p>

<p style='text-align: justify;'>En segundo lugar, copiar un objeto para poder modificarlo deja de ser trivial también, ya que no vale con copiar el objeto en sí. Hay que copiar todos los nodos hacia arriba en la jerarquía hasta llegar hasta aquel que no pertenece a una única referencia. Esto se entiende fácilmente mirando al dibujo de arriba. Supongamos que desde A se modifica G. Ahora, A tiene que contener G', pero H sigue teniendo que contener el antiguo valor de G. Si únicamente se reemplaza G por G', esto modificaría el valor de H, y por lo tanto perdería la relación parte-todo mencionada antes. Es necesario reemplazar también E.</p>

![Copia](https://raw.githubusercontent.com/asielorz/blog/master/images/copy-on-write-2.png)

<p style='text-align: justify;'>Por supuesto, sería posible conseguir esto haciendo que cada nodo conociera a sus nodos superiores, pero esto sería muy costoso y además haría el uso más engorroso y limitaría los casos en los que se puede usar. El código que sería deseable escribir al describir una estructura compuesta por objetos copy-on-write es algo parecido a esto:</p>

```cpp
struct MyType
{
    CopyOnWrite<A> foo;
    CopyOnWrite<B> bar;
    CopyOnWrite<C> quux;
};
```

<p style='text-align: justify;'>Y luego poder tener (o no) un <code>CopyOnWrite&lt;MyType&gt;</code>.</p>

<p style='text-align: justify;'>Esto es ahora mismo un problema imposible. Si uno quiere construir este tipo de estructuras jerárquicas heterogéneas es necesario recurrir a patrones de la programación funcional: interfaces inmutables y propagación de los cambios devolviendo valores recursivamente, lo cual a veces resulta tedioso. Tal vez sería interesante intentar resolver este problema. O tal vez sería un ejercicio de tripodología felina.</p>
