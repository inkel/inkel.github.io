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

{% highlight ruby lineno %}
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

La función Fibonacci no es tan costosa de computar, pero para valores
de `n` grandes puede tardar bastante tiempo calculando los valores
recursivos de la misma. En el ejemplo lo que hacemos es almacenar en
Memcached los valores de la función para un cierto `n`. La función
`fetch` recibe un parámetro que indica la clave con la cual vamos a
almacenar el valor, y el bloque que le pasamos es el valor que
queremos almacenar. Si la clave existe no se computa el valor sino que
se devuelve directamente desde Memcached. Caso contrario se computa el
valor, se almacena en Memcached y se devuelve el resultado.

Para ejemplificar ejecuté el siguiente <i lang="en">benchmark</i> con
los estos resultados:

{% highlight ruby lineno %}
N = 30
Benchmark.bm do |bm|
  bm.report 'non-cached' do
    0.upto(N) { |n| fib n }
  end
  bm.report 'cached' do
    0.upto(N) { |n| fibmc n }
  end
end

#             user       system     total    real
# non-cached  0.700000   0.010000   0.710000 (0.709932)
# cached      0.020000   0.000000   0.020000 (0.014653)
{% endhighlight %}

¡La versión con cache se ejecuta en menos del 10% del tiempo que sin
cache! No siempre obtendremos estos valores utilizando cache, pero
queda claro en el ejemplo que se pueden disminuir muchísimo los
tiempos y recursos utilizados.

Este tipo de cache se puede denominar también como **cache del lado del
servidor**, dado que si bien el cliente notará una disminución en el
acceso al recurso, la comunicación cliente-servidor se sigue
realizando.

### HTTP Cache

Esta es la forma de cache más desconocida por los programadores web y
tal vez la menos comprendida, y sin embargo un mínimo nivel de dominio
de la misma nos puede permitir conseguir resultados impresionantes en
la experiencia de los usuarios de nuestro sistema.

Este tipo de cache al contrario que la cache de resultados es una
cache del lado del cliente, que no necesariamente representa al
usuario que se conecta a nuestro servidor, sino que puede representar
también a proxies intermedios.

Con HTTP se pueden manejar dos formas de cache del lado del cliente:
caches temporales y caches condicionales.

#### Caches temporales

Inicialmente para este tipo de cache hay que tener acceso a la
configuración del servidor (Apache, Nginx, IIS, etc) para indicarle
que se envíen ciertas cabeceras para diversos recursos. Este tipo de
cache es el más adecuado para implementar con recursos estáticos como
imágenes, hojas de estilo o JavaScript.

Ésta es tal vez la forma mas simple de aplicar cache a recursos, pero
por esa misma simpleza podemos cometer errores. Básicamente el truco
consiste en, al devolver un recurso por primera vez a un cliente,
indicarle también el tiempo de expiración del contenido del
mismo. Luego la próxima vez que el mismo cliente solicite el mismo
recurso lo que hará es fijarse si la fecha actual es menor que la
fecha de expiración: de ser así entonces no realizará la petición al
servidor sino que servirá los resultados de su propia cache interna;
caso contrario se volverá a pedir todo el contenido del recurso. Un
ejemplo de configuración en Apache con mod_cache sería, para todas las
fotos, expirarlas luego de un año de solicitadas:

{% highlight apache %}
ExpiresByType image/jpeg "acces push 1 year"
{% endhighlight %}

Por simple que parezca, esto puede brindar resultados increíbles en
término de performance y ahorro de ancho de banda. El caso de uso
ideal para implementar es en aquellos recursos que no cambian muy
frecuentemente, como pueden ser imágenes de fondo, sprites de
imágenes, hojas de estilos, librerías JavaScript (si servimos jQuery
desde nuestro servidor por ejemplo), etcétera. En estos casos la mejor
solución, y la menos propensa a problemas, es calcular el tiempo de
expiración del recursos, y duplicar esa fecha al comunicársela al
cliente. Luego, si el recurso cambio, modificar su nombre, y de esta
forma forzamos que el cliente obtenga una copia fresca del
contenido. Esto tal vez el razón por la cual no se implementa mucho
este tipo de cache, por el *mantenimiento* que conlleva tener que
cambiar todas las referencias a los recursos actualizados,
mantenimiento que por cierto no es tan grave si se tiene una buena
librería para referenciarlos desde nuestro sistema.

Hace unos años empezó a utilizarse como solución a renombrar los
recursos el agregarle una cadena de números como parte del query
string del recurso, por ejemplo:

`http://example.com/image/inkel.png?12345`

La verdad que esta solución no está ni cerca de ser la solución
correcta dentro de las definiciones del protocolo HTTP, sino que
surgió a que ciertos navegadores (cuyo nombre no queremos repetir) en
vez de hacer cache utilizando solamente el nombre del recurso,
utilizan **todo** el contenido de la URL, lo cual no corresponde. Sin
dudas todos en algún momento u otro implementamos este truco, y hasta
tal vez conseguimos los resultados buscados... hasta que llegó un caso
en el que no y la respuesta que se da cuando nos reportan un issue es
*borrá tu cache*. ¡Horror! Es una respuesta muy poco profesional de
nuestra parte, y que nos deja mal parados a todos, dado que no podemos
pretender que los usuarios ajenos a nuestro equipo reaccionen como
deseamos. ¿Se imaginan a un frontend developer de Twitter respondiendo
a un cambio que no se ve *avisarle a todos los usuarios de Twitter que
tienen que limpiar su cache para que nuestro sitio muestre la imagen
correcta*?
