---
layout: post
title: `operator ?` y el mal código autogenerado
date: 2020-06-23
tags:
  programación
---
<p style='text-align: justify;'>Contaba Kevlin Henney en una charla que revisando código de otra persona se encontró con una clase en la que para cada variable privada de la clase había un par de funciones get y set que leían y escribían la variable respectivamente, a pesar de que esto no tenía ningún sentido porque esas variables tenían que mantener una serie de invariantes que se podían romper si se les escribía valores arbitrarios. Al preguntarle al autor del código en cuestión por qué lo había hecho, este le contestaba que su editor de texto tenía una opción muy cómoda de autogenerar estas funciones, así que era algo que él ya hacía por defecto cada vez que añadía una variable a una clase. Henney aprovecha esta anécdota para reflexionar sobre la generación procedural de mal código y la forma en la que las herramientas que usamos dan forma a nuestra forma de pensar, no siempre para nuestro bien.</p>

<p style='text-align: justify;'>Una reflexión similar me surge al hilo del operator ? de los tipos opcionales de rust y el operator ?. de los pointers de switft, que vienen a ser azúcar sintáctico para decir si el objeto es válido haz esto, si no lo es no hagas nada. Y las demostraciones siempre contienen largas cadenas de accesos donde cada uno puede fallar y escribirlo en términos de if-else como se ha hecho toda la vida sería muy tedioso.</p>

`level?.terrain?.decals?.add_decal(position, texture);`

```cpp
if (level)
{
	auto terrain = level->terrain;
	if (terrain)
	{
		auto decals = terrain->decals;
		if (decals)
		{
			decals->add_decal(position, texture);
		}
	}
}
```

<p style='text-align: justify;'>El problema está en que tal vez los árboles nos están impidiendo ver el bosque. Aunque es cierto que el operator ? es una solución genial para este problema que son las cadenas largas de accesos a través de pointers en estructuras, la verdadera pregunta es si realmente queremos tener estas cadenas largas de accesos en nuestro programa. Este tipo de código, además de ser fatal para la memoria caché y por lo tanto para la velocidad del programa, es sintomático de un diseño demasiado orientado a objetos, sobre el que es muy difícil razonar, es costoso de mantener y además es difícil de poner a prueba con pruebas autocontenidas y simples.</p>

<p style='text-align: justify;'>A la hora de hacer herramientas que hacen ciertas tareas más fáciles de hacer, deberíamos tener en cuenta ante todo si hacer esas tareas es deseable en primer lugar. El mal código autogenerado lleva inevitablemente a más mal código, ya que ese se convierte en el camino de menor resistencia. Si un mal diseño de software destaca con patrones tediosos y en los que resulta evidente que ahí hay algo mal, será más fácil que alguien intente arreglarlo que si los síntomas más superficiales se esconden detrás de sintaxis conveniente y procesos automatizados.</p>
