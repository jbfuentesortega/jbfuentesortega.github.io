---
layout: post
title: protected es mentira
date: 2021-11-19
tags:
  informática
  programación
---
<p style='text-align: justify;'>Es común, en lenguajes de programación que permiten programar en un estilo orientado a objetos, que se pueda restringir el acceso a los miembros de una clase. Los especificadores de acceso más comunes suelen ser <code>public</code>, que permite que el miembro sea accedido desde cualquier lugar; <code>private</code>, que sólo permite que el miembro sea accedido por funciones miembro de la clase, y <code>protected</code>, que permite que el miembro sea accedido por las funciones de esta clase y aquellas que deriven de esta clase, transitivamente. Estos tres especificadores de acceso suelen ser los más comunes y los encontramos en lenguajes como C++, Java o C#. Mientras que <code>public</code> y <code>private</code> tienen significados sólidos, bien definidos y correctamente ejecutados por el lenguaje de programación, el significado de <code>protected</code> es más pobre, hasta el punto de que podríamos considerarlo equivalente a <code>public</code>.</p>

## Cómo saltarse `protected`, el tutorial

<p style='text-align: justify;'>Supongamos que hay una clase que tiene un miembro <code>protected</code> al que necesitamos acceder. Por ejemplo, el estándar manda que <code>std::stack</code> tenga su contenedor interno en una variable miembro <code>protected</code> llamada <code>c</code>. Ahora, supongamos que tenemos una stack que contiene un vector, y en la que queremos hacer que el vector reserve cierta cantidad de elementos de antemano para evitar crecer el bloque mientras se usa la stack. En principio, acceder al vector interno desde fuera es imposible, ya que éste es <code>protected</code>. Sin embargo, nada nos impide escribir algo así</p>

```cpp
template <typename T, typename C>
auto underlying_container(std::stack<T, C> & stack) -> C &
{
    struct Oops : public std::stack<T, C>
    {
        auto get_underlying_container() -> C & { return c; }
    };
    return static_cast<Oops &>(stack).get_underlying_container();
}
```

<p style='text-align: justify;'>Veamos un poco qué está sucediendo en esta función. Se está declarando, dentro de la función, una nueva clase que deriva de aquella en la que se quiere acceder a un miembro protegido y añade una función que permite acceder a ese miembro. Como la función <code>get_underlying_container</code> es un miembro de una clase que hereda de <code>std::stack</code>, tiene permitido acceder a <code>c</code>. Es importante tener en cuenta también que este código no es peligroso ni puede incurrir en undefined behavior accidentalmente ni va a dejar de funcionar misteriosamente en el futuro. No se está quitando <code>const</code> a nada, ni reinterpretando pointers como tipos no relacionados. <code>std::stack</code> y el tipo al que hemos decidido bautizar como <code>Oops</code> tienen exactamente la misma disposición en binario, ya que <code>Oops</code> no añade ninguna variable, lo que significa que acceder a <code>c</code> desde <code>std::stack</code> y desde <code>Oops</code> es exactamente la misma operación. Otra ventaja de esta técnica es que la existencia de <code>Oops</code> es local a la función <code>underlying_container</code>. No necesitamos cambiar todos los sitios en los que usamos <code>std::stack</code> por un <code>Oops</code>. Podemos seguir usando <code>std::stack</code> y aun así acceder a sus miembros protegidos con funciones como ésta.</p>

<p style='text-align: justify;'>Además, es importante tener en cuenta que esto no es una particularidad de <code>std::stack</code>. Puede usarse esta técnica para acceder a cualquier variable protegida de cualquier clase de forma segura y no intrusiva. Una vez llegado a este punto, cualquier variable protegida es en realidad pública en la práctica, sólo que con un poco de tedio extra. Una vez que tenemos <code>underlying_container</code>, podemos hacer algo como:</p>

```cpp
std::stack<int, std::vector<int>> stack;
underlying_container(stack).reserve(64);
```

## De todas formas, ¿qué significa `protected`?

<p style='text-align: justify;'>Antes hemos mencionado que los significados de <code>public</code> y <code>private</code> están bien definidos. Para variables, una variable pública es una variable que no tiene precondiciones, o, dicho de otra forma, que no participa en las invariantes de la clase. Es seguro hacer cualquier operación segura sobre una variable pública sin invalidar el estado de la clase en la que vive. Por el contrario, una variable privada es una variable cuyo valor está restringido por las invariantes de la clase. Por ejemplo, una clase de vector suele contener tres variables: el pointer a la memoria, el tamaño y la capacidad. Sin embargo, estas tres variables no tienen valores arbitrarios y no relacionados entre sí. El pointer apunta a un array que tiene memoria como para tantos elementos como capacidad tenga el vector y en el que hay tantos elementos construidos como tamaño tenga el vector. El valor de las tres variables está estrechamente relacionado, y modificar cualquiera de ellas al azar, sin valorar su significado y cómo afecta al resto, es un error. Por eso, estas variables son privadas, para que sólo puedan ser manipuladas por un puñado de operaciones que sabemos que son seguras y no van a invalidar el estado de la clase.</p>

<p style='text-align: justify;'>Para funciones, el significado es parecido. Una función pública suele representar una operación segura. Una operación que puede invocarse en una clase cuyas invariantes estén satisfechas y que garantiza que al terminar las invariantes de la clase seguirán cumpliéndose. Por el contrario, una función privada suele ser un detalle de implementación de las funciones públicas y suele tener permitido operar sobre estados inválidos de la clase ya que nunca va a ser llamada fuera de contexto. Siempre se va a llamar desde otra función miembro de la clase que entiende su significado y se va a encargar de que al final el estado de la clase vuelva a ser válido.</p>

<p style='text-align: justify;'>Pero, ¿qué es un miembro protegido? Por un lado, que el acceso a un miembro protegido pudiera poner en peligro las invariantes de la clase sería algo inseguro, ya que puede ser accedido por código de terceros. Recordemos que, a menos que la clase sea final (en cuyo caso <code>protected</code> significa exactamente lo mismo que <code>private</code>), cualquiera puede extender una clase y acceder a un miembro protegido, por lo que debería ser tratado por el implementador de la clase, a efectos prácticos, como si fuera un miembro público más de la clase. Un miembro protegido es parte de la interfaz de la clase con la que va a interactúar código de terceros, por lo que a menos que queramos cargar sobre éstos el peso de entender y mantener las invariantes de una clase que no han escrito, un miembro protegido tiene que tener las mismas garantías de seguridad que uno público. Por otro lado, si es un miembro en el que es seguro operar y sobre el que no pesa ninguna invariante de clase, ¿por qué no es público? ¿Por qué hacer que el acceso sea tan incómodo y tedioso?</p>

## Conclusión

<p style='text-align: justify;'>Un miembro protegido es, a efectos prácticos, público, ya que puede ser accedido libremente por código de terceros. Por lo tanto, a la hora de diseñar una clase, y decidir el especificador de acceso de un miembro, un miembro protegido debería tener las mismas garantías de seguridad que uno público. Y, una vez llegado a este punto, si operar sobre él es seguro, cabría preguntarse por qué es ese miembro protegido y no público. Ver un <code>protected</code> en el código suele ser señal de que una clase ha tenido poca reflexión sobre cuál es su interfaz y cómo debería ser usada, y en general está cerca de ser un antipatrón a evitar en la medida de lo posible. En el mejor de los casos, un miembro protegido es un miembro público al que acceder es tedioso. En el peor de los casos, es un bug latente esperando a saltar por los aires.</p>
