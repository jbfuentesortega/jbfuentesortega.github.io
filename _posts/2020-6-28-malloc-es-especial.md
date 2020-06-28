---
layout: post
title: malloc es especial
date: 2020-06-28
tags:
  programación
---
<p style='text-align: justify;'>La programación funcional es un estilo de programación que modela los cálculos como la evaluación de expresiones, a diferencia de la programación imperativa, donde los cálculos suceden a través de la ejecución de instrucciones que mutan el estado del programa. Una función típica en un lenguaje funcional no es un conjunto de instrucciones que se ejecutan juntas, es decir, una subrutina, sino una asociación de un conjunto de valores a otro valor. Para esto es necesario que una función no mute sus parámetros y tampoco lea o escriba a estado mutable global, o interactúe con el exterior accediendo al sistema de archivos o a la red. Llamaremos a este tipo de cambios efectos secundarios, y a las funciones que no causan efectos secundarios funciones puras.</p>

<p style='text-align: justify;'>Sin embargo, es bastante común en lenguajes funcionales encontrar funciones que devuelven listas de cosas. Mientras que una función de ordenar imperativa tiene más bien esta pinta, <code>void sort(list&lt;T&gt; & v)</code>, en un lenguaje funcional lo que nos encontramos es más bien <code>sort: [T] -> [T]</code>. Esta interfaz coge una lista y devuelve una lista nueva, con los elementos ordenados. Como el número de elementos en la lista es desconocido de antemano, para poder hacer esto es necesario que esta función pida al sistema operativo memoria dinámica, a través de malloc o un mecanismo similar. malloc es una función que claramente tiene efectos secundarios. No hay más que abrir el explorador de tareas para ver cómo sube la cantidad de RAM ocupada por el programa. Por no hablar de que tiene que mantener una lista la memoria que está disponible y que ya ha dado, y que esta lista es global. Y, aún así, nadie considera llamar a malloc un “efecto secundario”.</p>

<p style='text-align: justify;'>Lo que malloc hace es, dado un tamaño en bytes, devolver un nuevo bloque de memoria de ese tamaño. Pero es que tiene una serie de propiedades curiosas, que le hacen parecer una función pura.</p>

- Dado el mismo tamaño, siempre devuelve un bloque igual, siempre y cuando no consideremos a dirección en memoria una propiedad importante.
- Anteriores llamadas a malloc o a otras funciones no modifican el comportamiento de malloc (excepto cuando el ordenador se queda sin memoria).
- Es seguro llamar a malloc en paralelo.
- Es posible optimizar llamadas a malloc si se puede hacer el mismo cálculo sin ellas, como demuestran las optimizaciones de clang en este aspecto o el operator new constexpr en C++20.

<p style='text-align: justify;'>Todas estas propiedades hacen a malloc especial dentro de las funciones que mutan estado global, y justifican que se pueda llamar desde funciones puras sin que dejen de serlo. Pero, más importante aún, el hecho de que podamos identificar y enumerar estas propiedades debería valernos también para encontrar otras funciones que, a pesar de no ser puras, pueden ser llamadas desde funciones puras sin que dejen de serlo. Otras funciones que también son especiales igual que lo es malloc.</p>
