---
layout: post
title: Mitos sobre la cache
description: Donde aclaro varios mitos que los programadores tienen acerca del cache
---

A lo largo de los años que llevo trabajando cometí muchos errores al trabajar con la famosa <em>cache</em>. Al principio fueron fruto de mi falta de conocimiento al respecto, y mi poca experiencia en cuanto a <strong>aprender a aprender lo que no sé</strong>.

## ¿Por qué es importante la cache?

La cache, y el correcto uso de la misma, es importante en el hecho de reducir la carga en nuestros servidores, permitiendo que se atiendan mas peticiones, brindándole al usuario una experiencia mas agradable al no tener que esperar por los resultados. En el caso de la cache de HTTP permite también reducir el consumo de ancho de banda, lo cual lleva a un ahorro monetario y una mayor velocidad de carga de una página.

## Tipos de cache

La principal distinción que podemos hacer es entre las caches de hardware y de software. Las primeras son implementadas en dispositivos como discos rígidos, lectores de CD-ROM/DVD, motherboards, etcétera. Aquí no vamos a hablar de esas caches dado que no tenemos a priori un acceso directo ni sencillo, y ya estaríamos hablando de optimizaciones de bajo nivel.

La cache de software en sí es un procedimiento que se utiliza para reducir el tiempo de computo de un proceso o la cantidad de información enviada a través de una conexión de red. Es un mecanismo que nos permite acelerar nuestras aplicaciones y reducir el consumo de recursos, pero como tal, no viene exento de detalles que si no son tenidos en cuenta pueden producir resultados erróneos o brindar información anticuada. Aquí vamos a hablar principalmente de los dos tipos de cache mas utilizados a la hora de construir páginas web, aunque hay que tener en cuenta que éstas no son las únicas formas y siempre pueden existir variaciones, aunque son lo bastante genéricas para poder implementarlas fácilmente y obtener buenos resultados.

### Cache de resultados

El primer tipo de cache es el mas conocido y probablemente, y es el acto de almacenar el resultado de un proceso complejo y/o lento para no tener que volver a computarlo cada vez que se solicita un recurso. Ejemplos de este tipo de cache son:

 * generación de páginas o fragmentos de las mismas;
 * manipulación de imágenes como recortes y varios tamaños;
 * almacenamiento de resultados de consultas a base de datos.

Si bien

### HTTP Cache

Esta es la forma de cache más desconocida por los programadores web y tal vez la menos comprendida, y sin embargo un mínimo nivel de dominio de la misma nos puede permitir conseguir resultados impresionantes en la experiencia de los usuarios de nuestro sistema.
