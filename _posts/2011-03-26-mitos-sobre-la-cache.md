---
layout: post
title: Mitos sobre la cache
description: Donde aclaro varios mitos que los programadores tienen acerca del cache
---

A lo largo de los años que llevo trabajando cometí muchos errores al
trabajar con la famosa <em>cache</em>. Al principio fueron fruto de mi
falta de conocimiento al respecto, y mi poca experiencia en cuanto a
<strong>aprender a aprender lo que no sé</strong>.

## ¿Por qué es importante la cache?

La cache, y el correcto uso de la misma, es importante en el hecho de
reducir la carga en nuestros servidores, permitiendo que se atiendan
mas peticiones, brindándole al usuario una experiencia mas agradable
al no tener que esperar por los resultados. En el caso de la cache de
HTTP permite también reducir el consumo de ancho de banda, lo cual
lleva a un ahorro monetario y una mayor velocidad de carga de una
página.

## Tipos de cache

La principal distinción que podemos hacer es entre las caches de
hardware y de software. Las primeras son implementadas en dispositivos
como discos rígidos, lectores de CD-ROM/DVD, motherboards,
etcétera. Aquí no vamos a hablar de esas caches dado que no tenemos a
priori un acceso directo ni sencillo, y ya estaríamos hablando de
optimizaciones de bajo nivel.

La cache de software en sí es un procedimiento que se utiliza para
reducir el tiempo de computo de un proceso o la cantidad de
información enviada a través de una conexión de red. Es un mecanismo
que nos permite acelerar nuestras aplicaciones y reducir el consumo de
recursos, pero como tal, no viene exento de detalles que si no son
tenidos en cuenta pueden producir resultados erróneos o brindar
información anticuada. Aquí vamos a hablar principalmente de los dos
tipos de cache mas utilizados a la hora de construir páginas web,
aunque hay que tener en cuenta que éstas no son las únicas formas y
siempre pueden existir variaciones, aunque son lo bastante genéricas
para poder implementarlas fácilmente y obtener buenos resultados.

### Cache de resultados

El primer tipo de cache es el mas conocido y probablemente, y es el
acto de almacenar el resultado de un proceso complejo y/o lento para
no tener que volver a computarlo cada vez que se solicita un
recurso. Ejemplos de este tipo de cache son:

 * generación de páginas o fragmentos de las mismas;
 * manipulación de imágenes como recortes y varios tamaños;
 * almacenamiento de resultados de consultas a base de datos.

Este es el tipo más sencillo de cache para utilizar, y a su vez muy
incomprendido y propenso a errores. A la hora de la implementación no
tiene sentido reinventar la rueda y hacerlo mal, sino que conviene
basarse en una solución ya probada, y para esto recomiendo utilizar
Memcached, sin dudas el lider absoluto en este tipo de soluciones.

Desarrollado inicialmente por LiveJournal para reducir la carga de
generar los contenidos de sus páginas, Memcached se ha convertido en
un estándar de facto a la hora de cachear resultados de procesos. De
modo muy básico se puede describir como un diccionario de pares
clave/valor. Existen librerías para casi todos los lenguajes y es
bastante sencillo de utilizar, al igual de estar implementados en
varios frameworks para web, como Ruby on Rails, CakePHP y Django.

#### Utilizando Memcached

{% highlight ruby %}
require 'memcached'

def fib n
  if n == 0 or n == 1
    n
  else
   fib(n - 1) + fib(n - 2)
  end
end

mc = Memcached.new '127.0.0.1', '11211'
def fibmc n
  if n == 0 or n == 1
    n
  else
    mc.fetch "fib:#{n}" do
      fibmc(n - 1) + fibmc(n - 2)
    end
  end
end
{% endhighlight %}

La función Fibonacci no es tan costosa de computar, pero para valores de `n` grandes puede tardar bastante tiempo calculando los valores recursivos de la misma. En el ejemplo lo que hacemos es almacenar en Memcached los valores de la función para un cierto `n`. La función `fetch` recibe un parámetro que indica la clave con la cual vamos a almacenar el valor, y el bloque que le pasamos es el valor que queremos almacenar. Si la clave existe no se computa el valor sino que se devuelve directamente desde Memcached. Caso contrario se computa el valor, se almacena en Memcached y se devuelve el resultado.

Veamos unas comparaciones.

### HTTP Cache

Esta es la forma de cache más desconocida por los programadores web y
tal vez la menos comprendida, y sin embargo un mínimo nivel de dominio
de la misma nos puede permitir conseguir resultados impresionantes en
la experiencia de los usuarios de nuestro sistema.
