---
layout: post
title: 'La composición procedural'
date: 2021-4-29
tags:
  informática
  programación
  c++
---
## Introducción

<p style='text-align: justify;'>La programación funcional ha puesto sobre la mesa la importancia de la composición, es decir, la construcción de programas grandes a partir de programas más pequeños. Siendo la función el átomo de la programación funcional, ésta ha puesto especial énfasis en la composición de funciones, definida como g∘f(x) = g(f(x)). Al sistema de reglas que rigen la composición de funciones lo llamamos "sistema de tipos". La mónada, bloque de construcción esencial de la programación funcional, no es más que una redefinición de la composición de funciones para poder componer flechas de Kleisli. En cierta medida, el auge de la teoría de categorías en ciertos círculos de programadores estos últimos años se debe principalmente al hecho de que la teoría de categorías es, en esencia, una abstracción del estudio de la composición de funciones arriba definida.</p>

<p style='text-align: justify;'>Por otro lado, el átomo de la programación orientada a objetos es la clase. La orientación a objetos ha encarado generalmente la composición mediante las conexiones entre objetos. Un programa orientado a objetos es, conceptualmente, un grafo de objetos donde cada uno esconde su estado y expone una interfaz. Estos objetos se comunican a través de mensajes definidos por sus interfaces. La forma de construir programas más complejos es generalmente añadiendo más objetos y más conexiones al grafo.</p>

<p style='text-align: justify;'>Sin embargo, el átomo de la programación procedural es el procedimiento o subrutina. Una subrutina es un conjunto de instrucciones que se ejecutan en orden y exponen una interfaz. Aunque a menudo reciben informalmente el nombre de funciones, las funciones son en realidad un subconjunto de las subrutinas. Por motivos de rigor en el tema tratado, este texto hará una distinción entre funciones y subrutinas. Una función es una asociación matemática entre conjuntos, definida generalmente por una expresión que mapea los parámetros a su resultado. Esta expresión está generalmente compuesta por otras subexpresiones, cuya composición está reglada por el sistema de tipos. Una subrutina está formada por una secuencia de statements. La relación entre los statements es temporal: cada statement se ejecuta después de aquellos que le preceden en orden. Cada statement lee y muta el estado del programa. Los statements se comunican mutando aquellas partes del estado del programa que futuros statements van a leer. La diferencia esencial entre funciones y subrutinas es la capacidad de las subrutinas para mutar el estado del programa más allá del valor devuelto. La composición en un programa procedural se da mediante la sucesión de statements que invocan otras subrutinas. En este ejemplo de <a href="https://www.youtube.com/watch?v=W2tWOdzgXHA">la charla C++ seasoning de Sean Parent</a>, podemos ver cómo construye el algoritmo <code>gather</code>, que dado una secuencia bidireccional, un punto en la secuencia y un predicado sobre los elementos mueve aquellos elementos para los que el predicado es verdad a ese punto, como la composición de dos llamadas a <code>stable_partition</code>.</p>

```cpp
template <BidirectionalIterator It,
          UnaryPredicate Pred>
auto gather(It first, It last, It point, Pred p) -> pair<It, It>
{
    return { stable_partition(first, point, not(p)),
             stable_partition(point, last, p) };
}
```

![Gather](https://raw.githubusercontent.com/asielorz/blog/master/images/gather.png)

## Tesis

<p style='text-align: justify;'>Muchos de los lenguajes de programación en uso hoy en día son procedurales. La influencia de C en el diseño de gran parte de los lenguajes de programación posteriores es innegable. Aunque estos lenguajes tratan ahora de incorporar elementos provenientes de la programación funcional, su uso es muchas veces incómodo y problemático. Si el objetivo del estilo o paradigma elegido es maximizar la capacidad de expresión, los aros por los que tiene que pasar un lenguaje principalmente procedural u orientado a objetos para escribir código en un estilo funcional son muchas veces suficientes para superar a las ventajas. Y esto se debe principalmente a que la composición de funciones requiere de grandes cantidades de azúcar sintáctico para poder ser aceptablemente ergonómica. De hecho, gran parte de Haskell consiste en azúcar sintáctico para facilitar la composición, mediante estructuras sintácticas que no existen en otros lenguajes, como las funciones de Curry o la notación "do".</p>

<p style='text-align: justify;'>Por su parte, la estructura orientada a objetos es problemática por ser difícil de paralelizar y optimizar, además de que la gran cantidad de estados en los que el programa puede estar y la opacidad de las interfaces a la hora de acceder al verdadero estado del programa pueden hacer que sea difícil razonar sobre el código y fácil introducir bugs.</p>

<p style='text-align: justify;'>Los lenguajes procedurales están preparados de forma natural para la composición procedural. Cuando uno escribe una función en C, Java o Python, lo que está escribiendo fundamentalmente es una secuencia de statements separados por punto y coma (salto de línea en el caso de Python). Las estructuras de control como <code>if</code>, <code>for</code> o <code>return</code> son asimismo statements. Por eso, para construir un programa de forma composicional en un lenguaje procedural, hay que encontrar las formas de explotar la composición procedural. Además, los lenguajes procedurales deberían ser extendidos de forma que los nuevos mecanismos introducidos exploten la composición procedural, suplan sus carencias o le encuentren nuevas aplicaciones.</p>

## &lt;algorithm&gt;

<p style='text-align: justify;'>Un gran ejemplo de código escrito para explotar la composición procedural es la biblioteca <code>&lt;algorithm&gt;</code>, escrita originalmente por Alexander Stepanov y Meng Lee como parte de la biblioteca estándar de C++. La biblioteca está compuesta por algoritmos pequeños y reutilizables a partir de los cuales se puede escribir algoritmos más grandes. Incluso se dan algunos casos en los que algoritmos de la biblioteca están escritos en términos de otros. Notablemente, <code>sort</code> está escrito en términos de <code>partition</code> y <code>stable_sort</code> de <code>merge</code>. El algoritmo gather presentado arriba es otro buen ejemplo de usar algoritmos de la biblioteca para escribir algoritmos nuevos. Es también destacable la implementación de Arthur O’Dwyer de un algoritmo para obtener la suma de los cuadrados de los dos números más grandes de una secuencia usando <code>partial_sort_copy</code> e <code>inner_product</code>.</p>

```cpp
auto stl_solution(const auto& v)
{
    int top[2] = {};
    partial_sort_copy(begin(v), end(v), begin(top), end(top), greater());
    return inner_product(begin(top), end(top), begin(top), 0);
}
```

<p style='text-align: justify;'>Este algoritmo es más fácil de leer que su contraparte escrita a mano, al seguir línea a línea la especificación del problema. Primero, consigue los dos elementos más grandes de la secuencia. Luego, calcula la suma de sus cuadrados. Además, es más fácil de extender o modificar, por ejemplo, para calcular la suma de los primeros tres, los primeros diez, o los dos más pequeños. <a href="https://quuxplusone.github.io/blog/2020/12/14/no-raw-loops-you-say/">El texto en el que Arthur O’Dwyer habla de este algoritmo</a> es una fantástica apología de la biblioteca <code>&lt;algorithm&gt;</code> y muy recomendable de leer.</p>

<p style='text-align: justify;'>Pero lo más importante es que <code>&lt;algorithm&gt;</code> no está pensada solamente para ser usada. Está pensada para ser imitada y extendida, ya que predica un estilo que resulta en un código eficiente, expresivo, correcto y fácil de componer y reusar. El lema "no raw loops" de Sean Parent, en gran parte influido por esta biblioteca y por la figura de Stepanov, también empuja en esta dirección.</p>

## Corrutinas

<p style='text-align: justify;'>El código asíncrono es difícil de escribir de forma correcta y eficiente, sobre todo en un estilo procedural. Supongamos una función que tarda mucho en devolver su resultado, por ejemplo una petición a una base de datos. Podríamos escribir código que interactuara con esta función más o menos así:</p>

```cpp
// Advertencia. Este código es imaginario y no está
// usando ninguna biblioteca existente, o al menos
// no de forma intencionada. Su objetivo es ser un
// ejemplo de una idea, no ser código de producción.
{
    auto data = request_data_from_database();
    auto transformed_data = program_logic(data);
    auto database_response = send_to_database(transformed_data);
    return database_response.is_ok();
}
```

<p style='text-align: justify;'>Esta implementación es muy legible. Podemos seguir la secuencia de pasos que conlleva esta operación y entenderla sin problemas. Sin embargo, es enormemente ineficiente, ya que en cada llamada a la base de datos se está parando el hilo de ejecución hasta que el resultado esté disponible. Esto es ineficiente porque el programa podría estar usando ese tiempo para llevar a cabo otros cálculos mientras espera a la respuesta de la red, en lugar de dormir. Una forma más inteligente en la que podemos escribir esta función es usando funciones de continuación para las funciones asíncronas, de forma que cuando se lanza una función que va a hacer esperar al programa, en lugar de esperar se provee al programa de la continuación que llevar a cabo una vez esté la respuesta. Mientras tanto el programa puede seguir haciendo otras cosas.</p>

```cpp
{
    future<Data> data = request_from_database();
    return data.then([](Data data)
    {
        auto transformed_data = program_logic(data);
        future<Response> database_response = send_to_database(transformed_data);
        return database_response.then([](Response response)
        {
            return response.is_ok();
        });
    });
}
```

<p style='text-align: justify;'>Y de repente hemos perdido gran parte de nuestra legibilidad. La función procedural, simple y legible se convierte en un monstruo en el que el exceso de control explícito para poder controlar la ejecución asíncrona de manera eficiente dificulta la legibilidad. Las corrutinas presentan una solución a este problema, permitiendo escribir código asíncrono de forma procedural. Sin embargo, al tener que esperar, en lugar de congelarse la corrutina devuelve el control al punto de invocación y permite ser reanudada más tarde, manteniendo la eficiencia.</p>

```cpp
{
    auto data = co_await request_from_database();
    auto transformed_data = program_logic(data);
    auto database_response = co_await send_to_database(transformed_data);
    return database_response.is_ok();
}
```

<p style='text-align: justify;'>De nuevo se recupera la legibilidad del primer ejemplo, esta vez con la eficiencia del segundo. El operador <code>co_await</code>, en lugar de congelar el programa, suspende la ejecución de la corrutina y devuelve un objeto invocable que permite reanudarla más adelante, permitiendo al programa realizar otras tareas mientras espera a la base de datos. Esto nos permite escribir rutinas asíncronas en un estilo procedural, más natural para algunos lenguajes, manteniendo sin embargo la eficiencia de las funciones de continuación. Las corrutinas son un gran ejemplo de cómo se puede extender un lenguaje para permitir usar la composición procedural en más situaciones, permitiendo resolver más tipos de problemas de la forma que es más natural para el lenguaje.</p>

## Corrutinas y tipos de error monádicos

<p style='text-align: justify;'>La programación procedural no tiene una forma canónica de reportar errores. O más bien tiene tres, cada uno con sus pros y sus contras. Hay subrutinas que cogen por referencia la memoria a la que escribir el resultado que van a generar y devuelven un código de error indicando si la subrutina ha finalizado con éxito o si algo ha fallado. Esta técnica implica intercalar la lógica del programa con la gestión de los errores y termina introduciendo un exceso de condicionales que dificultan la legibilidad. Además, tiene el problema de que los errores pueden ignorarse, lo cual suele ser poco deseable y conducir a errores. Las excepciones, al contrario, introducen saltos implícitos en el control del programa que hace difícil razonar sobre ellas, y suelen ser más ineficientes ya que por lo general dependen de memoria dinámica. Por último, los tipos de error monádicos, como <code>optional&lt;T&gt;</code> o <code>expected&lt;T, E&gt;</code>, suelen tener interfaces con las que es incómodo trabajar, ya que la composición de mónadas no se lleva del todo bien con el estilo procedural.</p>

<p style='text-align: justify;'>Las corrutinas de C++20 son tan flexibles que con un poco de imaginación permiten operar sobre los tipos de error monádicos con la sintaxis de corrutinas. La principal ventaja de esto es que se puede definir que <code>co_await</code> signifique extraer el resultado si hay alguno o terminar la subrutina y devolver el error si no. Esto permite escribir código en C++ que opera sobre tipos de error monádicos que es muy similar en forma a la notación "do" de Haskell.</p>

```cpp
// Sin corrutinas
optional<Level> load_level(string path)
{
    optional<string> file_content = load_all_file(path);
    if (!file_content)
        return nullopt;
		
    optional<json::value> level_json = json::parse(*file_content);
    if (!level_json)
        return nullopt;
	
    return build_level(*level_json);
}

// Con corrutinas
optional<Level> load_level(string path)
{
    string file_content = co_await load_all_file(path);
    json::value level_json = co_await json::parse(file_content);
    return build_level(level_json);
}
```

<p style='text-align: justify;'>De esta forma, conseguimos lo mejor de los dos mundos. Tenemos la eficiencia de propagar los errores por la stack, sin memoria dinámica. Sin embargo, el control es explícito. A diferencia de con las excepciones, se puede ver exactamente cuáles son los puntos en los que el programa puede fallar, sin que la sintaxis sea ruidosa. Además, los errores que una subrutina puede generar son anotados en el prototipo, facilitando la legibilidad e interoperabilidad del código. Por último, la propuesta de pattern matching que se espera aprobar para C++23 hará el interactuar con los errores en los puntos en los que se quiera hacer más cómodo.</p>

<p style='text-align: justify;'>De nuevo nos encontramos ante la misma idea. Las corrutinas aplicadas a los tipos de error muestran una forma de reportar y propagar errores en los programas que es eficiente, difícil de usar mal y natural para el estilo procedural. Es llamativo que ésta sea la única forma en la que se propagan errores en Rust, usando una macro para conseguir el efecto que <code>co_await</code> consigue en C++.</p>

## Contratos

<p style='text-align: justify;'>Los contratos son una respuesta a un conflicto entre corrección y eficiencia en la programación procedural. La programación funcional impone que todas las funciones sean totales. Esto hace que sea extremadamente raro para un programa funcional abortar o tener comportamiento no definido, ya que el sistema de tipos obliga a que en todo momento se contemplen todos los posibles estados del programa. Sin embargo, es ineficiente, ya que obliga a introducir comprobaciones que a veces son innecesarias, en situaciones en las que es demostrable que una función parcial nunca va a ser llamada fuera de su dominio. Un ejemplo simple es el acceso por índice a una secuencia. Es necesario que el índice sea mayor o igual a 0 y menor que el tamaño de la secuencia. Sin embargo, si iteramos la secuencia con el muy común patrón de <code>for (int i = 0; i &lt; n; ++i)</code>, es evidente que el índice siempre va a estar dentro de los límites del tamaño, por lo que la comprobación cada vez es redundante e ineficiente. Además, muchas veces evaluar una función parcial fuera de su dominio no es un estado contemplable del programa, sino un bug. Lo deseable en ese caso es notificar al programador y modificar el programa para que esa situación no se dé, y no permitir que el programa sea capaz de recuperarse.</p>

<p style='text-align: justify;'>Los contratos permiten extender la declaración de una subrutina para incluir, además de los tipos de los parámetros, las precondiciones que deben cumplir. Al compilar el programa en modo <i>debug</i>, estas precondiciones se convertirán en comprobaciones que abortarán el programa al fallar. En modo <i>release</i>, las comprobaciones desaparecen y se convierten en pistas para el optimizador de qué estados no van a suceder nunca dentro del programa. Los contratos facilitan la composición procedural acotando las interfaces de las subrutinas sin introducir caminos de error que dificulten la lectura y comprensión del código. También facilitan la búsqueda de bugs, sobre todo cuando están acompañados por una buena cobertura de tests automatizados. Además, permiten generar el código óptimo, ya que en la versión final esas comprobaciones desaparecen. Es más, se convierten en un mecanismo para transmitir al optimizador condiciones que no son expresables en el sistema de tipos, lo que le permite, al contar con más contexto, optimizar mejor.</p>

```cpp
// Ejemplo de contrato en una función de acceso por índice
T & vector<T>::operator [] (size_t i) [[assert: i < size()]];
```

## Extensiones a constexpr y el TS de reflection

<p style='text-align: justify;'>La implementación original de <code>constexpr</code> en C++11 sólo permitía funciones puras formadas por una sola expresión. Básicamente restringía la evaluación en el compilador únicamente al reino de la programación funcional. Sin embargo, era tedioso de usar, ya que expresar todos los problemas en una sola expresión en C++ es complicado, lo que llevó a que no se usara mucho. Las extensiones a constexpr en C++14, 17 y 20 han ido incorporando elementos procedurales, permitiendo tener más de un statement, mecanismos de control como <code>if</code> o <code>for</code>, mutación del estado local de la función o de los parámetros e incluso memoria dinámica. Ahora incluso se puede usar <code>&lt;algorithm&gt;</code> en el compilador. Conforme el subconjunto del programa evaluable en el compilador crece y tiende a converger con el lenguaje normal, usar <code>constexpr</code> se vuelve más fácil y más gente lo está usando para hacer más cosas con él. También se vuelve factible evaluar código más complejo en el compilador, porque es más fácil de escribir y mantener.</p>

<p style='text-align: justify;'>De la misma forma, hasta ahora la metaprogramación ha sido puramente funcional. Si a esto se le suma una sintaxis de templates bastante horrible, el resultado es que algo tan simple como ordenar una lista de tipos es un problema muy difícil. Sin embargo, en el TS de reflection propuesto para C++23 los metaobjetos que representan a los elementos del programa son valores que se pueden meter en contenedores y manipular con algoritmos como si fueran números enteros. Esto, junto con las extensiones a <code>constexpr</code> que permiten usar <code>&lt;algorithm&gt;</code> en el compilador, permite metaprogramar en un estilo procedural, en el que se puede reutilizar subrutinas escritas para transformar otros tipos de datos también para manipular tipos. La posibilidad de escribir en este estilo, mucho más natural para C++ y más cercano para el programador medio, tendrá el efecto de reducir la barrera de entrada a la metaprogramación, permitirá escribir metaprogramas más complejos y que en general más problemas sean resueltos de esta manera cuando tenga sentido, con soluciones que son legibles, mantenibles y escalables.</p>

## Conclusión

<p style='text-align: justify;'>Los lenguajes procedurales están especialmente capacitados para la composición procedural. La composición procedural no es mejor ni peor que la funcional. Ambas necesitan un gran apoyo del lenguaje de programación para ser adecuadamente usables y sin él se sienten torpes e incómodas de usar, de forma que cada lenguaje debería centrarse en explotar el estilo que más le convenga dadas las herramientas que ya tiene. Resulta que en el mundo hay varios lenguajes de programación muy usados que son procedurales, y que no van a cambiar de paradigma porque eso requeriría un cambio tan radical que estaríamos hablando de un lenguaje distinto al final. Para los programadores que trabajan en un lenguaje procedural, es importante tener una buena comprensión del paradigma y de sus puntos fuertes para poder aprovecharlos al máximo. Es importante también conocer los algoritmos y aprender a escribir a partir de los existentes nuevos algoritmos que sean tan reutilizables como los de Stepanov.</p>

<p style='text-align: justify;'>Desde el punto de vista del desarrollo de lenguajes de programación, la forma de extender estos lenguajes pasa por reconocer el hecho de que son procedurales, e intentar explotarlo al máximo. Diseñar las nuevas extensiones al lenguaje de forma que puedan componerse bien en un estilo procedural resulta en elementos que permiten expresar más en menos texto e interoperar mejor que el resto de lenguaje. Como hemos visto, el desarrollo de C++ en la última década tiene un importante componente procedural, y el trabajo existente en reflection, contratos y pattern matching continúa en esa dirección. Rust va más allá y ha decidido no incluir virtual, la característica esencial de la orientación a objetos. Parece que apuesta de lleno por lo procedural. Habrá que ver adonde llega. Parece que el futuro de la programación de bajo nivel será procedural. Tocan tiempos interesantes.</p>
