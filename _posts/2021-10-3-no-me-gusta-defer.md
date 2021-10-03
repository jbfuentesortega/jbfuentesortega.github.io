---
layout: post
title: 'No me gusta defer'
date: 2021-10-3
tags:
  informática
  programación
---
<p style='text-align: justify;'>Suele haber una gran influencia mutua en las decisiones de diseño de los lenguajes de programación de bajo nivel, o "de sistemas", como se les llama ahora. Cuando uno desarrolla una nueva idea suele ser explorada y adoptada por varios durante los próximos años y es frecuente encontrar elementos similares o idénticos en varios de los lenguajes. Esto se debe principalmente a que la limitación de garantizar un gran control sobre la velocidad de ejecución del programa acota muchísimo el espacio de diseño, lo que hace que en última instancia todos estos lenguajes estén explorando el mismo subconjunto del espacio de diseño. La seguridad de los recursos, es decir, el evitar fugas de memoria u otros recursos, siempre ha sido una preocupación muy grande para estos lenguajes de programación, sobre todo debido a que en C, el abuelo de todos, garantizar la seguridad de los recursos es un proceso manual y en el que es fácil meter la pata. C++ resuelve el problema mediante los destructores. Un tipo encapsula un recurso y gracias a su destructor, que se ejecuta cuando el tipo es destruido, garantiza que el recurso siempre va a ser devuelto cuando deje de ser necesario. Esta idea ha sido muy influyente y es la adoptada por D y Rust. Por otro lado, en los últimos años ha surgido una forma alternativa de garantizar la seguridad de recursos: <code>defer</code>. Ideada originalmente por Go¹, <code>defer</code> es la forma preferente en Zig de gestionar recursos² y se está trabajando en añadirla a C³. Hay además implementaciones muy interesantes en C disponibles hoy usando el preprocesador⁸. En su último paso por CppCast, JeanHeyd Meneide, editor del documento del estándar de ISO de C y colaborador del comité estándar de C++, argumentó a favor de introducir <code>defer</code> en C e incluso de su potencial en C++ como complemento de los destructores⁴, explicando cómo facilitaría programar en el estilo transaccional descrito por Alexandrescu en la cppcon de 2015⁵.</p>

<p style='text-align: justify;'>¿En qué consiste <code>defer</code>? La idea es simple. <code>defer</code> permite posponer la ejecución de código al final del bloque en el que es declarado.</p>

```c
{
    defer puts(“Esto se ejecuta al final.”);
    puts(“Esto se ejecuta primero.”);
    puts(“Esto se ejecuta segundo.”);
}
```

<p style='text-align: justify;'>El código anterior escribiría a la consola lo siguiente:</p>

```
Esto se ejecuta primero.
Esto se ejecuta segundo.
Esto se ejecuta al final.
```

<p style='text-align: justify;'>Además, <code>defer</code> garantiza que el código pospuesto se llamará independientemente de la forma en la que el flujo de control del programa salga del bloque, lo que facilita mucho garantizar la limpieza de recursos incluso en el caso en el que una función tiene varios puntos de retorno. En cierto sentido, defer es una forma de declarar destructores ad hoc en el cuerpo de una función. Su uso más común es posponer la devolución de un recurso acto seguido de adquirirlo, de forma que una fuga se vuelve imposible.</p>

```c
{
    FILE * file = fopen(filename, “rb”);
    defer if (file != NULL) fclose(file);
	
    // Código que procesa el archivo
}
```

<p style='text-align: justify;'>También es útil para desbloquear mecanismos de sincronización, destruir archivos temporales o incluso escribir strings paralelas como abir y cerrar paréntesis, corchetes, llaves o comillas al serializar. Otro caso en el que es relevante es en bibliotecas como ImGui, donde muchas llamadas para crear widgets de UI tienen que venir acompañadas de una llamada a la función que las cierra. Es el caso de <code>ImGui::Begin/End</code>, <code>TreePush/TreePop</code>, <code>BeginPopup/EndPopup</code> o <code>BeginCombo/EndCombo</code>, entre otras.</p>

<p style='text-align: justify;'>Los más avispados del lugar se habrán dado cuenta de que este texto se titula "No me gusta <code>defer</code>", así que alguna queja tendré que tener con todo esto. Bueno, pues vamos allá. Mi mayor objeción para con <code>defer</code> es su falta de estructura. <code>defer</code> podría ser entendido, como hemos dicho antes, como una forma de declarar destructores ad hoc en medio de funciones. La principal ventaja de los destructores es que es imposible olvidarse de llamarlos. Cuando uno abre un archivo en C++, Rust o D, es imposible que se olvide de cerrarlo. Habría que hacer un esfuerzo activo retorciendo las reglas del lenguaje para conseguir una fuga del archivo. Sin embargo, es perfectamente posible que un programador escribiendo <code>fopen</code> se olvide de escribir <code>defer fclose</code> en la siguiente línea. Desde luego, es mucho más fácil comprobar si el archivo se cierra correctamente si para ello hay que mirar la siguiente línea a donde se abre en lugar del final de la función, y <code>defer</code> hace también mucho más fácil garantizar que el archivo se cierra siempre aunque la función tenga un flujo de control complejo, pero no garantiza que el programador vaya a acordarse de escribir el código de limpieza. En un entorno en el que <code>defer</code> es la forma preferente de limpiar recursos, es necesario que el programador memorice que cada vez que pide un recurso a un sistema tiene que acordarse de escribir el código de limpieza pertinente. El código de limpieza se vuelve más fácil de escribir, desde luego, pero el problema fundamental no ha sido resuelto. Mientras que en un sistema con destructores el código que uno escribe por defecto es correcto, en un sistema que depende de <code>defer</code> para liberar los recursos es posible olvidarse de hacerlo. Es más, el código escrito por defecto es incorrecto y uno tiene que activamente acordarse de hacerlo bien. La diferencia es una carga cognitiva grande que se impone al programador, y que resulta en una mayor probabilidad de introducir bugs.</p>

## Complementar los destructores: guardias de scope estructurados

<p style='text-align: justify;'>Creo que los destructores son la mejor forma de gestionar recursos. Desde luego son la forma más intuitiva y más segura. Los destructores se comportan como uno esperaría que lo hicieran, y lo hacen siempre sin tener que acordarse de ellos, lo que permite liberar una gran carga cognitiva. Sin embargo, gestionar recursos no es la única razón por la que uno podría querer ejecutar código al final de un scope. Antes hemos mencionado desbloquear mecanismos de sincronización o asegurarse de que llamadas paralelas de empezar y terminar una acción se ejecutan de forma correspondiente. Y es en estas situaciones en las que los destructores son más ruidosos y poco intuitivos de lo deseable.</p>

```cpp
{
     auto guard = std::scope_guard(my_mutex);
	 
    // Código sincronizado por “my_mutex”
}
```

<p style='text-align: justify;'>Hay que declarar una variable para el guarda, que es inútil ya que no tiene ningún miembro público ni ninguna función además del destructor. Además, es fácil confundirse y crear una variable temporal que libere el mutex al instante. Este bug es tan común que tiene su propia entrada en las core guidelines de C++⁷. De todas formas, <code>defer</code> no es mucho mejor.</p>

```c
{
    mtx_lock(&my_mutex);
    defer mtx_unlock(&my_mutex);
	
    // Código sincronizado por “my_mutex”
}
```

<p style='text-align: justify;'>Sigue siendo posible olvidarse de llamar a <code>defer</code>, hay que escribir dos líneas y el nombre del mutex dos veces… Es bastante mejorable. Sinceramente, me gusta el <code>lock</code> de C#⁶.</p>

```c#
lock (my_mutex)
{
    // Código sincronizado por “my_mutex”
}
```

<p style='text-align: justify;'>Me parece insuperable en densidad de información y en facilidad de uso. No hay información redundante ni ruido, es conciso y fácil de leer y es imposible de usar mal. Lo que no me gusta del lock de C# es que es un caso particular introducido en la sintaxis del lenguaje. Mientras que con destructores o <code>defer</code> podemos usar el mismo mecanismo para todo tipo de gestión de recursos, código de limpieza y llamadas paralelas, C# ha decidido tratar a los mutexes de forma especial, e introducir sintaxis específica para ello. Esto hace que aunque trabajar con mutexes en C# es muchísimo más cómodo que en C++ o Zig, acordarse de cerrar un archivo o usar una biblioteca como ImGui no lo es, porque el mecanismo que usa para bloquear mutexes no es genérico.</p>

<p style='text-align: justify;'>Sin embargo, una generalización del <code>lock</code> de C# podría ser la solución definitiva al problema. Sería muy interesante un mecanismo que permitiera implementar estructuras arbitrarias de la forma</p>

```
nombre(parámetros)
{
    código
}
```

<p style='text-align: justify;'>que permitiera personalizar bajo qué condiciones se ejecuta ese código, y qué sucede antes y después. Este mecanismo sería estrictamente mejor que <code>defer</code>, ya que le daría estructura, que es lo que le falta, además de ser un gran complemento para los destructores, haciendo bien precisamente las cosas que a los destructores se les dan mal. Un punto de partida interesante podría ser el diseño de <code>for_expansion</code> y macros que cogen código como parámetro en Jai⁹. De nuevo habría que generalizarlo para permitir su uso en casos arbitrarios en lugar de solamente en el bucle for, pero es un comienzo muy interesante.</p>

## Referencias:

1- Defer en Go <a href="https://golangbot.com/defer/">https://golangbot.com/defer/</a>

2- Propuesta para introducir defer en C <a href="http://robertseacord.com/wp/wp-content/uploads/2020/09/Defer-Mechanism-for-C-1.pdf">http://robertseacord.com/wp/wp-content/uploads/2020/09/Defer-Mechanism-for-C-1.pdf</a>

3- Defer en Zig <a href="https://ziglang.org/learn/overview/#manual-memory-management">https://ziglang.org/learn/overview/#manual-memory-management</a>

4- Entrevista a JeanHeyd Meneide in CppCast <a href="https://cppcast.com/jeanheyd-defer/">https://cppcast.com/jeanheyd-defer/</a>

5- Declarative control flow, Andrei Alexandrescu, CppCon 2015 <a href="https://www.youtube.com/watch?v=WjTrfoiB0MQ&pp=ugMICgJlcxABGAE%3D">https://www.youtube.com/watch?v=WjTrfoiB0MQ&pp=ugMICgJlcxABGAE%3D</a>

6- Lock statement en C# <a href="https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/lock-statement">https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/lock-statement</a>

7- Core Guidelines de C++ sobre no dar nombre a lock_guard  <a href="https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-name">https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-name</a>

8- Implementación de defer en C con el preprocesador <a href="https://godbolt.org/z/7P151zvjr">https://godbolt.org/z/7P151zvjr</a>

9- Explicación de macros, #code y for_expansion en Jai <a href="https://www.youtube.com/watch?v=QX46eLqq1ps">https://www.youtube.com/watch?v=QX46eLqq1ps</a>
