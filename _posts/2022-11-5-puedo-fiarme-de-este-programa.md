---
layout: post
title: '¿Puedo confiar en este programa?'
date: 2022-11-5
tags:
  informática
---
<p style='text-align: justify;'>Este es un texto que deberías poder enseñar a tus padres para explicarles unos mínimos de cómo no ser estafado en internet.</p>

# Capítulo 1: Un correo del banco

<p style='text-align: justify;'>Imaginemos la siguiente situación: he recibido un correo electrónico de mi banco diciendo que he hecho una transferencia de tanto dinero, con un enlace en el que me dicen que haga clic para cancelar esa transferencia. A mí esto me huele un poco raro. Yo no recuerdo haber hecho esta transferencia. Mejor que la cancele por si acaso, ¿no? O a lo mejor es mentira. ¿Puedo fiarme de este mensaje?</p>

![Correo falso](https://raw.githubusercontent.com/asielorz/blog/master/images/correo-falso.png)

<p style='text-align: justify;'>En 1983, Ken Thompson recibió el premio Turing (algo así como el premio Nobel de informática). Todo receptor del premio Turing tiene que dar una conferencia al recibir el premio, y la de Thompson trató sobre el tema de la confianza en la informática. Thompson se preguntaba si se puede confiar en un programa arbitrario. Es decir, si dado un programa, podemos inspeccionarlo para asegurarnos de que es seguro. A continuación, procedió a mostrar cómo saltarse la pantalla de pedir contraseña de un popular sistema operativo y acceder directamente a la sesión de cualquier usuario en ese ordenador. Las manipulaciones que hizo, aunque simples, eran prácticamente indetectables. Cualquiera que usara el sistema operativo trucado sería totalmente inconsciente de que un actor malicioso podía acceder a sus archivos sin necesidad de contraseña. Thompson terminó la charla concluyendo que es imposible confiar en un programa arbitrario, y que la informática necesita un modelo de confianza distinto. Puede que no podamos confiar en los programas, pero podemos confiar en las personas. Eso significa que, aunque uno no pueda confiar en un programa, puede confiar en la persona que le ha enviado ese programa, y saber que el programa es seguro de ejecutar porque quien nos lo ha enviado lo dice y confiamos en él. Es similar a cómo funcionamos con la comida. No vamos por ahí probando si cada plato que comemos está envenenando, ni vamos todo el día cargados con medidores de toxicidad y antídotos. La mayor parte del tiempo sabemos que la persona que nos está dando de comer no quiere envenenarnos.</p>

<p style='text-align: justify;'>Bueno, volvamos al correo del banco e intentemos ver cómo podemos aplicar las conclusiones de Thompson. Hemos dicho que es imposible distinguir un correo verdadero de ING de uno falso que esté lo bastante bien hecho. De forma que mirando al correo en sí no vamos a sacar nada. Ahora bien, ¿quién nos ha mandado este correo? ¿Podemos confiar en él? El correo dice ser de ING, de forma que sería de esperar que quien sea que ha mandado el mensaje lo ha hecho desde una cuenta que termine en algo tipo @ing.com o @ing.es. Ahora bien, miremos de nuevo quién ha mandado este mensaje.</p>

![Correo falso destacando que el remitente es paquito45@gmail.com](https://raw.githubusercontent.com/asielorz/blog/master/images/correo-falso-remitente-destacado.png)

<p style='text-align: justify;'>Vaya, vaya. ¡Pero si es paquito45@gmail.com! Debe de ser el becario nueve de ING. O a lo mejor es un estafador. No sé vosotros, pero yo no confío en paquito45@gmail.com para que me lleve mis gestiones bancarias. Algo me huele mal aquí. Todo apunta a que alguien está enviando correos haciéndose pasar por ING desde una cuenta que claramente no es de ING, por si acaso algún incauto pica y termina haciendo clic en el enlace para intentar cancelar la transferencia y termina entregando sus datos bancarios a una página falsa.</p>

# Capítulo 2: Phishing

<p style='text-align: justify;'>El phishing es una forma de estafa en la que una página web se hace pasar por otra para engañar al usuario a introducir datos sensibles como contraseñas pensando que está utilizando la página verdadera. Un tipo muy común de phishing es una página que es idéntica a la página de inicio de sesión verdadera de algún servicio web, pero que en lugar de mandar los datos a ese servicio web para iniciar sesión de verdad se los manda al autor de la página falsa. Por ejemplo, supongamos que no nos hemos dado cuenta y hemos hecho clic en el enlace del correo falso de ING de arriba para cancelar la transferencia, y nos ha traído a esta página.</p>

![Página de inicio de sesión de ING falsa](https://raw.githubusercontent.com/asielorz/blog/master/images/inicio-sesion-ing-falso.png)

<p style='text-align: justify;'>Se parece mucho a la página de inicio de sesión verdadera de ING, ¿no? Cómo que la he copiado, vamos. Mirando a la página es imposible distinguirla de la verdadera y darnos cuenta de que es de mentira. De nuevo, no podemos confiar en el programa en sí, porque no se puede. Tenemos que ver si confiamos en la persona que nos ha enviado el programa. La página de inicio de sesión de ING es <a href="https://ing.ingdirect.es/app-login/">https://ing.ingdirect.es/app-login/</a>. Puede que el enlace exacto cambie en el futuro, pero sería de esperar que siempre terminara en algo como ing.es, ing.com, ingdirect.es… Algo así. Veamos qué tenemos aquí:</p>

![Página de inicio de sesión de ING falsa, destacando que la url es paquito45.com](https://raw.githubusercontent.com/asielorz/blog/master/images/inicio-sesion-ing-falso-url-destacada.png)

<p style='text-align: justify;'>¡Pero bueno, si es nuestro amigo paquito45 otra vez, que como es un chico muy organizado parece que se ha hecho una página web para todas sus estafas! La única persona (jurídica en este caso) con derecho a pedirnos nuestra contraseña de ING, es ING. No deberíamos dar nuestra contraseña de ING a nadie más, da igual cuánto se curre la página de inicio de sesión. Siempre que una página nos pide una contraseña, deberíamos mirar qué página es y qué contraseña nos está pidiendo, y si tiene sentido que esta página nos pida esta contraseña. Si quien sea que nos está pidiendo la contraseña no tiene derecho a saber esa información, nunca deberíamos dársela.</p>

# Capítulo 3: película.exe

<p style='text-align: justify;'>A veces, hay que descargar un programa de internet. Puede que hayamos comprado una licencia de Photoshop y queramos descargar el instalador, o que haya salido una nueva versión de Microsoft Office y queramos actualizar, o que hayamos comprado un juego en una tienda de juegos online como Steam o Epic y queramos descargarlo para jugarlo. Descargar un programa es mucho más peligroso que descargar una foto o una película, porque un programa puede hacer lo que quiera. Puede borrar todos nuestros archivos, o mandárselos por internet a alguien malvado, o quedarse instalado de fondo registrando todas las teclas que pulsamos en busca de contraseñas. Un programa malvado de este tipo es lo que solemos llamar un virus informático.</p>

<p style='text-align: justify;'>Como decía Thompson en su conferencia de 1983, es imposible analizando simplemente un programa, saber si ese programa es un virus o no. Esto significa que el trabajo de un antivirus es, por definición, imposible. Por supuesto, los antivirus detectan muchos virus, y con los años se han vuelto cada vez más sofisticados y precisos, pero es ingenuo pensar que un antivirus va a ser capaz de detectar correctamente todos los virus, porque es imposible.</p>

<p style='text-align: justify;'>Por eso, de nuevo volviendo a la conclusión de Thompson, la cuestión no es si se puede confiar en el programa que hemos descargado, sino si se puede confiar en la persona que nos ha mandado ese programa. Por ejemplo, si hemos descargado un programa, por ejemplo una nueva versión de Office o un instalador de Windows, desde la página oficial de Microsoft, no hace pasarle un antivirus para saber que va a ser seguro. Igualmente, si descargamos el instalador de Photoshop desde la página oficial de Adobe, sabemos que va a ser auténtico. Las tiendas de juegos acostumbran a ser de fiar y no alojar virus en su catálogo de juegos a la venta, de forma que si compramos un juego en Steam, Epic, itch.io, GoG o cualquier otra tienda conocida, todo apunta a que el programa descargado va a ser el que esperamos que sea.</p>

<p style='text-align: justify;'>El problema viene cuando el lugar desde el que hemos descargado el programa no es tan de fiar. Servicios de intercambio de archivos descentralizados como BitTorrent o Ares, páginas web de dudosa autenticidad, la cuenta de Mega de vete a saber tú quién, tutoriales de YouTube poco fiables, foros donde algún internauta anónimo asegura que su enlace sirve para descargar tal programa de pago totalmente gratis… Por supuesto, la piratería existe, y algunos de estos enlaces son lo que dicen que son y funcionan de verdad, pero la confianza aquí es mucho más dudosa que en una página oficial. Por lo general no es buena idea descargar programas de internet y ejecutarlos sin estar totalmente seguro de que son auténticos, y siempre que se hace se asume un riesgo.</p>

![Tutorial para piratear Photoshop subido por la cuenta "paquito45 tutoriales"](https://raw.githubusercontent.com/asielorz/blog/master/images/tutorial-estafa.png)

<p style='text-align: justify;'>Parece que nuestro amigo paquito45 ha decidido diversificar su negocio de estafas y ha empezado a hacer tutoriales en YouTube explicando cómo piratear programas de pago. No estoy diciendo que seguir los pasos de este tutorial vaya a terminar instalando un virus en el ordenador, pero por lo que sea a lo mejor no es la idea más sensata del mundo.</p>

# Conclusiones

<p style='text-align: justify;'>Al final, todo esto se resume en que el mejor antivirus es saber dónde haces clic. No es una cuestión de ser capaz de analizar un correo, una página web o un programa y saber si es de fiar o no, porque es muy difícil y en muchos casos imposible. Navegar por internet con un mínimo de seguridad es tan simple como mirar quién nos está mandando algo, y saber decidir si ese alguien es una autoridad suficiente para estar mandándonos eso y por lo tanto nos podemos fiar, o si por el contrario no lo es y posiblemente sea un fraude. Ante la duda, un fraude potencial es un fraude. Lo único totalmente seguro es fiarse solamente de aquello que fijo que es auténtico.</p>
