---
title: Diferencias entre Service Endpoint y Private Link
author: gareda
# date: 2024-XX-XX 10:55:00 +0100
categories: [Azure, Networking]
image: assets/img/posts/azure/service-endpoint-vs-private-link/cover.png
tags: [virtual network, private link,private endpoint]
---

Uno de los problemas de concepto que más me suelo encontrar en Azure es la privatización de los servicios PaaS. Se suele producir por la falta de conocimiento de redes, falta de conocimiento de la plataforma como que en su día Microsoft decidiera dejar nombres muy similares para servicios muy similares en funcionalidad pero con usos diferentes.

## Service Endpoint

Esta característica fue la primera que se implementó con el objetivo de restringir el tráfico entre servicios. Esta se complementa el firewall propio que incorporan los servicios Paas. En la mayoría de servicios con firewall, podemos limitar el acceso mediante una lista blanca de direcciones públicas o mediante el bypass de confianza de los servicios de Azure.

![Service Endpoint drawing](assets/img/posts/azure/service-endpoint-vs-private-link/service-endpoint-drawing.png){: w="800" }

Con la característica de Service Endpoint podemos crear una lista blanca de direccionamiento privado de aquellas redes que se encuentren dentro de nuestro Tenant. Esto permite que los servicios de IaaS (Máquinas virtuales) puedan consumir los servicios PaaS por direccionamiento privado sin que tengan que salir a Internet.

Vamos a ponerlo a prueba con una cuenta de almacenamiento. Lo primero, tendremos que crear una red virtual que seria donde se montarían las máquinas virtuales. En la subred que creemos, deberemos configurar la sección de service endpoint con el servicio de *Microsoft.Storage* para habilitar la característica.

> Si no puedes ver bien la imagen, puede pinchar sobre ella para ampliarla.
{: .prompt-info }

![Service Endpoint subnet](assets/img/posts/azure/service-endpoint-vs-private-link/service-endpoint-subnet.png){: w="800" }

Cuando tengamos la configuración realizada desde el lado de la red, podemos pasar al servicio PaaS. Cada servicio es un mundo, pero a la hora de configurar el firewall deberían ser similares en su gran mayoría. Para el caso de la cuenta de almacenamiento, deberemos habilitar la limitación de acceso por firewall. En esta configuración, podremos establecer que redes que tengan habilitada la función de service endpoint, pueden consumir el servicio PaaS.

![Service Endpoint config](assets/img/posts/azure/service-endpoint-vs-private-link/service-endpoint-config.png){: w="800" }

Con esta configuración, el acceso a la cuenta de almacenamiento esta restringido a los servicios IaaS que esten en nuestra red asignada, el restro del tráfico sera rechazado por el servicio.

Visto esto podemos decir que la característica de **Service Endpoint** es una funcionalidad que nos puede servir para ciertas arquitecturas de red, ya que es fácil de configurar y sin coste adicional. En los entornos más restrictivos es posible que tengamos que hacer uso de los servicios que ofrece la característica de **Private Link**.

## Private Link

Texto temporal

### Private Endpoint

Texto temporal

### Private Link Service

Texto temporal
