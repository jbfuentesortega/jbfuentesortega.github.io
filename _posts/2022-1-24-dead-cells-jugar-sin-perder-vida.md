---
layout: post
title: 'El algoritmo de salto en la cabeza de Super Mario'
date: 2022-1-20
tags:
  videojuego
  informática
---
<p style='text-align: justify;'>En este texto quiero explicar uno de mis algoritmos de implementación de gameplay favoritos, por su simplicidad y elegancia resolviendo un problema bastante difícil. El problema es el siguiente, en un juego de plataformas como Super Mario en el que al saltar sobre un enemigo éste muere, cuando sucede una colisión entre el jugador y un enemigo, ¿cómo decide el juego si lo que está sucediendo es que el jugador ha saltado sobre el enemigo o por el contrario ha chocado contra él y es el jugador el que tiene que morir? El nombre que le doy al algoritmo es inventado por mí. No sé si fueron los programadores de Nintendo los inventores de este algoritmo, y es posible que no todos los juegos de Mario la usen. Sin embargo, <i>Super Mario Land</i> para la Game Boy es el juego más antiguo del que tengo constancia que usa este algoritmo. Otro ejemplo de juego que lo usa es uno de los juegos de móvil de Nitrome, <i>Leap Day</i>.</p>

<p style='text-align: justify;'>El problema tiene más miga de la que parece. Cuando dos objetos colisionan en un videojuego 2D lo único que sabemos en principio es que sus cajas de colisión están superpuestas durante este fotograma. Esto no nos permite distinguir entre los dos casos mencionados arriba: ¿es el jugador el que ha aterrizado sobre el enemigo, o es el enemigo el que ha chocado contra el jugador? ¿Quién de los dos tiene que recibir el daño?</p>

<p style='text-align: justify;'>Una aproximación posible al problema podría ser comparando las posiciones relativas de los objetos. Si el jugador está encima del enemigo, es que ha saltado sobre él. Si no, no. Esto puede parecer suficiente, pero tiene falsos positivos en casos muy comunes que lo hacen inservible en la práctica. Supongamos el siguiente caso, en el que el rojo es el enemigo, el azul oscuro es el jugador ahora y el azul claro es el movimiento del jugador.</p>

![Bash](https://raw.githubusercontent.com/asielorz/blog/master/images/algoritmo-salto-cabeza-mario-1.png)

<p style='text-align: justify;'>El jugador y el enemigo están colisionando, y el jugador está encima, sin embargo el jugador no ha aterrizado sobre el enemigo, sino que ha saltado y mientras saltaba se ha chocado de cara con él. En esta situación el jugador debería recibir daño, no el enemigo. Cualquier aproximación al problema que use dos cajas, ya sea para los pies del jugador o para la cabeza del enemigo, no es más que una versión más sofisticada de esta solución y va a estar sujeta a problemas muy parecidos.</p>

<p style='text-align: justify;'>¿Cuál es el algoritmo de Mario entonces? ¿Y si, en vez de comparar las posiciones, comparáramos las velocidades? Si la velocidad en el eje Y del jugador es menor que la del enemigo, eso puede significar dos cosas. O bien el jugador está cayendo a más velocidad que el enemigo, y por lo tanto ha aterrizado sobre él, o bien el jugador está ascendiendo a menos velocidad que el enemigo, y por lo tanto el enemigo ha golpeado al jugador por debajo. En ambos casos la solución es la misma: el enemigo pierde. En el caso contrario, si la velocidad en el eje Y del jugador es mayor o igual, el jugador ha chocado contra el enemigo y el enemigo gana.</p>

<p style='text-align: justify;'>Me gusta mucho esta solución por lo simple que es y lo buenos resultados que da. Además, si no se quieres guardar velocidades para el jugador o los enemigos porque el movimiento del juego no está basado en una simulación física, se pueden usar velocidades de Verlet guardando la posición del objeto el fotograma anterior y restándolas para obtener una aproximación de la velocidad.</p>

<p style='text-align: justify;'>Un posible falso positivo de este algoritmo es la situación en la que el jugador está cayendo y golpea a un enemigo estático (en el eje Y) con una esquina superior. Algo parecido al caso inverso del anterior.</p>

![Bash](https://raw.githubusercontent.com/asielorz/blog/master/images/algoritmo-salto-cabeza-mario-1.png)

<p style='text-align: justify;'>Intuitivamente uno pensaría que en este tipo de casos el jugador debería perder. Sin embargo, tanto en <i>Super Mario Land</i> como en <i>Leap Day</i> el jugador gana en este caso. Se puede resolver fácilmente combinándolo con la técnica anterior y usando tanto la posición como la velocidad para decidir si el jugador ha aterrizado sobre el enemigo. Es decir, el jugador ha aterrizado sobre un enemigo si su posición en el eje Y es superior y su velocidad en el eje Y es inferior. También puede dejarse sin resolver, como demuestran los dos juegos puestos de ejemplo. Acertar saltos que explotan este tipo de truco requiere habilidad y es satisfactorio para el jugador, y puede permitir una colocación de enemigos en el diseño de niveles que no sería posible sin.</p>

<p style='text-align: justify;'>En general, es un algoritmo muy fácil y muy efectivo que apenas requiere información extra para funcionar y que resuelve correctamente la mayoría de situaciones. Las situaciones donde falla son fáciles de detectar y corregir, e incluso dejar los fallos puede ser interesante como mecánica desde el punto de vista del diseño. Si vais a desarrollar un juego de plataformas en el que se puede saltar sobre la cabeza de los enemigos, planteaos usar este algoritmo.</p>
