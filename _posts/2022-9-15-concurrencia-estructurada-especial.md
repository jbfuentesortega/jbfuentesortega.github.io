---
layout: post
title: 'La concurrencia estructurada también es especial'
date: 2022-9-15
tags:
  informática
  programación
---
<p style='text-align: justify;'>Decíamos en su día que <a href="https://asielorz.github.io/malloc-es-especial/">malloc es especial</a>. Con ello queríamos decir que, a pesar de causar efectos secundarios en el sistema operativo, en la práctica podemos tratar malloc como una función pura, o al menos considerar que una función que llama a malloc no por ello deja de ser pura. El texto terminaba preguntándose qué más funciones son especiales. Hoy tenemos un nuevo ejemplo: la concurrencia estructurada también es especial.</p>

<p style='text-align: justify;'>Pequeño resumen de qué es la concurrencia estructurada. Es el paradigma de programación concurrente en el que toda ejecución asíncrona lanzada por un bloque de código debe terminar dentro de ese bloque de código. Entendemos como “bloque de código” el concepto de bloque de la programación estructurada, es decir, el código delimitado por llaves {} en C. La programación estructurada se diferencia de un paradigma en el que la ejecución asíncrona se lanza sin control de cuándo va a terminar, con frecuencia con una función de continuación o callback al que llamar cuando termine su trabajo. En vez de eso, se fuerza al hilo de ejecución que lanza esta tarea asíncrona a esperar a que termine antes de salir del bloque que ha lanzado la tarea. Una explicación muy buena de qué es la concurrencia estructurada, desde la motivación a sus principios, puede leerse en el artículo <a href="https://vorpus.org/blog/notes-on-structured-concurrency-or-go-statement-considered-harmful/">Go statement considered harmful</a> de Nathaniel Smith.</p>

<p style='text-align: justify;'>La concurrencia estructurada, como cualquier otro tipo de programación concurrente, requiere de efectos secundarios. Lanzar trabajo en paralelo requiere de crear nuevos hilos de ejecución o de registrar tareas en un sistema de tareas centralizado. Además, las funciones de continuación que estas tareas paralelas ejecutan al terminar requieren también de provocar efectos secundarios para poder transmitir su resultado y el mensaje de que han terminado al hilo principal. Sin embargo, la concurrencia estructurada es en la práctica invisible desde fuera de la función, desde un punto de vista semántico. Una función que use concurrencia estructurada para acelerar su trabajo va a ser, a todos los efectos observables, idéntica a una función que hiciera el mismo trabajo de forma secuencial, sólo que más rápida.</p>

<p style='text-align: justify;'>Por ello, podemos considerar la concurrencia estructurada especial de la misma forma que malloc. Una función por lo demás pura que use concurrencia estructurada para acelerar su trabajo es, a efectos prácticos, pura. Este conocimiento es muy valioso ya que significa una importante optimización para el código funcional sin tener que ceder en ser funcional.</p>
