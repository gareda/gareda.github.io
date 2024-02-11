---
title: Diferencias entre Service Endpoint y Private Link
categories: [Azure, Networking]
tags: [virtual network, private link, private endpoint]
date: 2024-02-11 12:10:00 +0100
img_path: assets/img/posts/azure/service-endpoint-vs-private-link
image: cover.png
author: gareda
---

Uno de los problemas de concepto que más me suelo encontrar en Azure es la privatización de los servicios PaaS. Se suele producir por la falta de conocimiento de redes, falta de conocimiento de la plataforma como que en su día Microsoft decidiera dejar nombres muy similares para servicios muy similares en funcionalidad pero con usos diferentes.

## Service Endpoint

Esta característica fue la primera que se implementó con el objetivo de restringir el tráfico entre servicios. Esta se complementa el firewall propio que incorporan los servicios Paas. En la mayoría de servicios con firewall, podemos limitar el acceso mediante una lista blanca de direcciones públicas o mediante el bypass de confianza de los servicios de Azure.

![Service Endpoint drawing](service-endpoint-drawing.png){: w="800" }

Con la característica de Service Endpoint podemos crear una lista blanca de redes privadas que se encuentren dentro de nuestro Tenant. Esto permite que los servicios de IaaS (Máquinas virtuales) puedan consumir los servicios PaaS por direccionamiento privado sin que tengan que salir a Internet y sin que el servicio PaaS tenga direccionamiento privado.

Vamos a ponerlo a prueba con una cuenta de almacenamiento. Lo primero, tendremos que crear una red virtual que seria donde se montarían las máquinas virtuales. En la subred que creemos, deberemos configurar la sección de service endpoint con el servicio de *Microsoft.Storage* para habilitar la característica.

> Si no puedes ver bien la imagen, puede pinchar sobre ella para ampliarla.
{: .prompt-info }

![Service Endpoint subnet](service-endpoint-subnet.png){: w="800" }

Cuando tengamos la configuración realizada desde el lado de la red, podemos pasar al servicio PaaS. Cada servicio es un mundo, pero a la hora de configurar el firewall deberían ser similares en su gran mayoría. Para el caso de la cuenta de almacenamiento, deberemos habilitar la limitación de acceso por firewall. En esta configuración, podremos establecer que redes que tengan habilitada la función de service endpoint, pueden consumir el servicio PaaS.

![Service Endpoint config](service-endpoint-config.png){: w="800" }

Con esta configuración, el acceso a la cuenta de almacenamiento está restringido a los servicios IaaS que estén en nuestra red asignada, el resto del tráfico será rechazado por el servicio.

Visto esto podemos decir que la característica de **Service Endpoint** es una funcionalidad que nos puede servir para ciertas arquitecturas de red, ya que es fácil de configurar y sin coste adicional. En los entornos más restrictivos es posible que tengamos que hacer uso de los servicios que ofrece la característica de **Private Link**.

## Private Link

Azure Private Link nacio para cubrir las necesidades de aquellos equipos que necesitaban tener más control sobre el tráfico de red de los servicios que se despliegan en Azure y que nose pueden permitir que esten expuestos directamente a internet. De esta caracteristica se componen de dos servicios, **Private Endpoint** y **Private Link Service**.

### Private Endpoint

Los servicios PaaS cuando son desplegados en Azure, por defecto la plataforma les establece un acceso por FQDN que expone a internet para que pueda ser consumido por cualquier usuario. Ciertamente, aquellos servicios como las *cuentas de almacenamiento*, aunque estén expuestos a internet, es necesario tener credenciales de acceso para poder consumirlas, pero hay servicios como las Web Apps, donde el acceso es libre para cualquier usuario.

Para proteger estos servicios, lo ideal es poder gestionar el tráfico que permite acceder a los mismos para evitar una exposición no deseada. Para poder gestionar este tráfico nació el servicio de **Private Endpoint** que permite montar una tarjeta de red con direccionamiento privado en los servicios PaaS y restringir por completo cualquier acceso público al servicio.

![Private Endpoint drawing](private-endpoint-drawing.png){: w="800" }

Pongamos a prueba esta configuración. Para ellos deberemos tener una red virtual que contenga subred. A continuación en la sección de **Networking** lo primero que configuraremos es opción de deshabilitar el trafico público y a partir de ese momento el servicio no podra ser consumido desde Internet.

![Private Endpoint PaaS service](private-endpoint-paas-service.png){: w="800" }

Ahora para poder consumirlo, deberemos pasar a pestaña de **Private endpoint connections** y darle a la opción de **+ Private endpoint** para proceder a crear el servicio. Tendremos que tener en cuenta:

- El nombre y el grupo de recursos donde lo queremos meter. También podremos definir el nombre del la tarjeta de red que acompaña al servicio.
- El tipo de *sub-servicio* al que queremos añadir el private endpoint. Esto ocurrirá en aquellos servicios que cuenten con más de una característica a exponer, como puede ser para las cuentas de almacenamiento, el servicio de *blobs* o de *file shares*.
- La red donde se va a desplegar la tarjeta de red.
- Y las zonas DNS que permitirán realizar la resolución de los servicios PaaS en red interna. Más información [aquí](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns-integration)

![Private Endpoint config](private-endpoint-config.png){: w="800" }

Una vez que tenemos configurados, solo necesitaremos el acceso desde una VPN o desde una máquina virtual que se encuentre dentro de la red para poder consumir el servicio.

Este servicio es un poco más complejo que los service endpoints, pero nos permite tener un control de los flujos de conexión ya sea mediante un Azure Firewall, usando Grupos de seguridad de red (NSGs) u otros sistemas con funciones similares.

Hay que tener en cuenta que la falta de experiencia realizando implantaciones de red o de servicios de DNS puede generar de problemas acceso cuando nos están bien configurados. A diferencia el otro, este si es un servicio de pago que supondrá un coste adicional por cada private endpoint que se despliegue.

### Private Link Service

En ocasiones, cuando se quiere ofrecer acceso algún tipo de servicio a un cliente como podría ser una BBDD y no queremos que esté expuesta a internet, la solución más inmediata que se nos ocurriría seria montar una VPN entre nuestra red y la red del cliente, pero podemos encontrar una alternativa más sencilla si ambos nos encontramos en Azure.

El Private Link Service se aprovecha de la tecnología de private endpoint para generar un túnel directo entre la red del cliente y la nuestra, permitiendo compartir solo al servicio sin exponer el resto de servicios que podríamos tener en la misma red.

![Private Link Service drawing](private-link-service-drawing.png){: w="800" }

Para poner a prueba esta configuración, vamos a necesitar un **Load Balancer** estándar, que junto a una configuración de backend que le establezcamos, podremos exponer un servicio. Cuando tengamos listo el load balancer, procedemos a crear el private link service y deberemos establecerle una red propia donde se desplegá y definirle que tipo de auto aprobación deseamos, todos los que no cumplan con ese requisito, tendrán que ser aprobados manualmente.

![Private Link Service config 1](private-link-service-config-1.png){: w="800" }

Una vez creado el private link service, si entramos dentro del servicio, en el overview podremos encontrarnos con el *alias*, que es el enlace que tendremos que compartir con el cliente para que tome como referencia al montar el private endpoint. Una vez que este creado, dependiendo de la configuración que hayamos establecido previamente, se autoaprobara la conexión del service endpoint o habrá que realizarla a mano.

![Private Link Service config 2](private-link-service-config-2.png){: w="800" }

Como podemos ver, este tipo de configuración es más sencilla que tener que configurar una VPN y seguramente las latencias serán inferiores, ya que la comunicación no saldrá de la red interna de Microsoft dentro de sus CPDs. Este tipo de configuraciones serán las ideales para ciertas situaciones, pero son raras de ver porque la mayoría de los equipos de redes la desconocen.

## Conclusiones

Como hemos podido, cada tecnología tiene un enfoque diferente y unos costes diferentes, que dependiendo de cada situación, sea más recomendable utilizar una u otra o ambas. A mí se me han dado situaciones en cliente donde he implementado tanto el service endpoint como el private endpoint para cumplir objetivos diferentes aunque sean tecnologías muy similares con funcionalidad muy similar.

Podemos resumir que la tecnología de service endpoint fue la primera en implantarse en Azure y que es las mejor para el bolsillo de los clientes, ya que no tiene un coste adicional, pero para las situaciones donde se debe controlar todo el flujo del tráfico, sobre todo en topologías Hub&Spoke, la tecnología de private link prima sobre la otra aunque suponga un coste adicional.
