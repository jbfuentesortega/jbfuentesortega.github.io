---
layout: post
title: 'Desarrollar intuición para el functor aplicativo'
date: 2022-10-9
tags:
  informática
  programación
---
<p style='text-align: justify;'>Suele pasar con las estructuras algebraicas que, al ser muy abstractas, es difícil entender a partir de la definición qué idea representan. Al programar, no tener una intuición sobre las propiedades de una estructura algebraica puede hacer que no diseñemos nuestros tipos de datos para satisfacer sus axiomas, aun pudiendo, y no obtengamos el beneficio de todos los teoremas y algoritmos desarrollados para esta estructura. El functor aplicativo es una estructura de datos especialmente abstrusa y sorprendentemente útil una vez entendida. Este texto intentará ofrecer una intuición para entender cuál es el propósito detrás de esta estructura algebraica. No prometo que uno saldrá de haber leído este texto sabiendo lo que es un functor aplicativo, al fin y al cabo desarrollar intuición requiere tiempo y práctica y es difícil atajar, pero se hará lo que se pueda. Al menos, se intentará abstenerse de metáforas descabelladas y referencias a la gastronomía mexicana.</p>

<p style='text-align: justify;'>El functor aplicativo fue introducido, hasta donde tengo conocimiento, en un paper de 2008 titulado <a href="http://www.staff.city.ac.uk/~ross/papers/Applicative.pdf "><i>Applicative programming with effects</i></a>. El paper es bastante confuso y es difícil extraer de él un propósito para lo que se está explicando. Sobre todo, es difícil entender, a partir de la definición, cuál es el problema que resuelve el functor aplicativo, y el paper no termina de dejar clara esta conexión. El functor aplicativo es una estructura de datos entre el functor y la mónada. Es decir, todos los functores aplicativos son functores, y todas las mónadas son functores aplicativos, pero no al revés.</p>

<p style='text-align: justify;'>Antes de comenzar con el functor aplicativo, repasemos el functor. Un functor es una estructura algebraica con una sola operación, llamada comúnmente map o fmap, y con la forma, dado un functor f:</p>

```Haskell
fmap :: (a -> b) -> f a -> f b
```

<p style='text-align: justify;'>Es decir, si tenemos un functor de As, y una función capaz de convertir As en Bs, podemos usar fmap para convertir el functor de As en un functor de Bs. Es valiosa aquí la intuición de Bartosz Milewski para entender los functores. Podemos pensar en un functor como su fuera un contenedor de As, que contiene en su interior varios valores de tipo A con alguna estructura. La función fmap transforma los valores contenidos en el functor, pero preserva la estructura. El ejemplo más común de functor suele ser la lista. Una lista contiene cero o más elementos de un mismo tipo en un cierto orden. Al llamar fmap sobre una lista con una función, se transforma cada elemento de la lista usando esa función y se obtiene como resultado una lista nueva, potencialmente de un tipo distinto, que contiene el resultado de transformar cada elemento de la primera lista con la función.</p>

<p style='text-align: justify;'>Fmap tiene sin embargo una limitación importante. Sólo funciona con funciones que cogen un solo parámetro. Sin embargo, se podría imaginar una situación en la que tenemos varios functores (varias listas, por ejemplo) y una función que coge varios parámetros, y que queramos llamar a esta función con estos functores. Fmap no nos va a ayudar en este caso. Y este es precisamente el propósito del functor aplicativo. Extender fmap a cualquier número de parámetros, de forma que dada una función que coge N parámetros y dados N functores donde el tipo del contenido de cada functor se corresponde con el tipo del parámetro, poder llamar a esa función con esos functores.</p>

<p style='text-align: justify;'>Para lograr eso, sobre la operación del functor, el functor aplicativo añade dos operaciones:</p>

```Haskell
pure : a -> f a
(<*>) : f (a -> b) -> f a -> f b
```

<p style='text-align: justify;'>Pure es muy simple. Dado un elemento, construye un functor que contenga únicamente ese elemento. Muchos functores tienen esta operación. Por ejemplo, construir una lista de un solo elemento, un Maybe o Result lleno, o una función constante que siempre devuelve lo mismo independientemente del parámetro.</p>

<p style='text-align: justify;'>La segunda tiene más enjundia. Se parece mucho a fmap, pero hay una diferencia esencial. En lugar de coger una función como primer parámetro, la función viene también dentro de un functor. Como curiosidad, podemos definir fmap trivialmente en términos de estas dos funciones, demostrando así que todo functor aplicativo es también, por definición, un functor.</p>

```Haskell
fmap t a = pure t <*> a
```

<p style='text-align: justify;'>En un principio, es difícil ver cómo llegamos desde aquí a llamar a una función que coge por ejemplo tres parámetros dados tres functores. Esta es una función binaria igual que fmap, que de hecho se parece mucho a fmap. Y este es el clic. Vamos a llamar al operador <code>&lt;*&gt;</code> varias veces, no sólo una, en la misma expresión. Ésa es la razón por la que el operador pide que la función venga dentro de un functor. De esa forma, el resultado de una llamada a <code>&lt;*&gt;</code> puede servir como parámetro a otra llamada a <code>&lt;*&gt;</code>.</p>

<p style='text-align: justify;'>El otro elemento clave a tener en cuenta es el curry de las funciones. En Haskell y muchos otros lenguajes funcionales no existen las funciones que cogen más de un parámetro. Sólo hay funciones unarias. Cuando hablamos de una función que coge tres parámetros en Haskell, en realidad estamos hablando de una función unaria que devuelve otra función unaria que devuelve otra función unaria que devuelve el resultado final. La asociatividad, al escribir <code>f a b c</code>, es <code>(((f a) b) c)</code>. Hablamos de funciones que cogen varios parámetros por conveniencia y porque tiene más sentido en muchos contextos pensar en ellas así, pero es importante entender que por debajo todo son funciones unarias. ¿Dónde entra en juego esto con los functores aplicativos? Supongamos que tenemos una función <code>t : a -> b -> d</code>, y dos objetos, <code>fa</code> y <code>fb</code> de tipos <code>f a</code> y <code>f b</code> respectivamente. Queremos llamar a <code>t</code> con los contenidos de <code>fa</code> y <code>fb</code>.</p>

<p style='text-align: justify;'>Intentemos entender qué sucede en la siguiente expresión:</p>

```Haskell
pure t <*> fa <*> fb
```

<p style='text-align: justify;'>Comencemos por <code>pure t</code>. Simplemente estamos construyendo un functor que contiene solamente la función t. Este es un paso necesario para poder llamar por primera vez a <code>&lt;*&gt;</code>, pero no tiene especial significado ya que <code>pure t</code> es exactamente lo mismo que t para todos los functores.</p>

<p style='text-align: justify;'>Analicemos ahora la expresión <code>pure t &lt;*&gt; fa</code></p>

<p style='text-align: justify;'>En este caso estamos invocando <code>&lt;*&gt;</code> con dos parámetros de tipo <code>f (a -> b -> c)</code> y <code>f a</code> respectivamente. Aunque por lo general pensamos en una función de tipo <code>(a -> b -> c)</code> como una función que coge dos parámetros, de tipo <code>a</code> y <code>b</code>, y devuelve un valor de tipo <code>c</code>. Sin embargo, si recordamos el curry, podemos pensar también en ella como una función que coge un solo parámetro, de tipo <code>a</code>, y devuelve una nueva función de tipo <code>(b -> c)</code>. Al verlo así nos damos cuenta de que esta llamada a <code>&lt;*&gt;</code> va a llamar a <code>t</code> con un valor de tipo <code>a</code> para obtener una nueva función <code>(b -> c)</code> y a guardar el resultado en un functor, devolviendo así un resultado de tipo <code>f (b -> c)</code>.</p>

<p style='text-align: justify;'>De forma que en la segunda llamada a <code>&lt;*&gt;</code> el primer parámetro, que es el resultado de la expresión anterior, tiene el tipo <code>f (b -> c)</code>, mientras que el segundo parámetro tiene tipo <code>f b</code>. Siguiendo la definición podemos ver trivialmente que el resultado de esta expresión tiene tipo <code>f c</code>, que es lo que intentábamos obtener.</p>

<p style='text-align: justify;'>Esto no funciona sólo con dos parámetros. Es extensible a cualquier cantidad de parámetros. Podemos escribir una función que concatene tantas llamadas a <code>&lt;*&gt;</code> como haga falta para invocar la función con todos sus parámetros.</p>

<p style='text-align: justify;'>Por supuesto, al igual que <code>fmap</code> tiene una implementación distinta para cada tipo, lo mismo sucede con <code>&lt;*&gt;</code>. El ejemplo más simple es Maybe. Fmap de Maybe es simple. Si el Maybe contiene un valor, transforma ese valor usando la función y devuelve un nuevo Maybe con el resultado de esa transformación. Si no, devuelve un Maybe vacío. La intuición con un functor aplicativo sería la misma, pero para una función con más parámetros. Si todos los Maybes tienen un valor, podemos llamar a la función y obtener un Maybe lleno con el resultado. Si al menos uno de los parámetros está vacío, el resultado será un Maybe vacío.</p>

<p style='text-align: justify;'>Recapitulando, en este texto hemos aprendido que el objetivo del functor aplicativo es extender fmap a funciones que cogen más de un parámetro. Para ello, usamos un operador un poco raro, <code>&lt;*&gt;</code>, que nos permite ir aplicando la función parcialmente parámetro a parámetro hasta que al final obtenemos el resultado.</p>
