---
title: Upgrade MediaWiki from 1.36.x to 1.39.x
date: '2024-01-14 20:00:00:00 +0300'
categories:
  - Linux
tags:
  - MediaWiki
render_with_liquid: false
published: true
---

![MediaWiki](/assets/img/common/MediaWiki-2020-logo.png)

## SITUACION

Tengo MediaWiki versión 1.36.x instalado en un Debian 12. Voy a dejar actualizado MediaWiki a la última versión LTS 1.39.x para evitar en un futuro tener que hacer varios saltos al actualizar.

En el momento de redactar esta entrada van por la versión 1.41.x estable, pero veo más conveniente ir a la versión LTS.

## REQUISITOS

Para realizar el upgrade me he basado en la [documentación oficial de MediaWiki](https://www.mediawiki.org/wiki/Manual:Upgrading)

La documentación dice que desde la versión 1.36 solo se puede hacer un upgrade directo si venimos como máximo de dos versiones LTS anteriores. En este caso puedo hacer upgrade directo.

[Version_lifecicle](https://www.mediawiki.org/wiki/Version_lifecycle)



## PASOS

**La parte de backup la omito, ya que dispong de un backup de la máquina virtual completa.**

El contenido actual de MediaWiki está en /var/www/html

Hago una copia en /opt y elimino el contenido de la ruta actual.

```plaintext
mkdir /opt/mediawiki-actual

rsync -av /var/www/html/ /opt/mediawiki-actual

rm -rf /var/www/html/*
```

Descargo el contenido de la nueva version y lo extráigo.

```plaintext
wget https://releases.wikimedia.org/mediawiki/1.39/mediawiki-1.39.6.tar.gz

tar -xvzf mediawiki-1.39.6.tar.gz

rsync -av mediawiki-1.39.6/ html

chown -R www-data:www-data html
```
Copio mi anterior fichero LocalSettings.php, las imágenes y el favicon a la nueva versión.

Al venir de la versión 1.36 no tengo que adaptar el fichero LocalSeting.php

No uso skis personalizadas ni tenía extensiones instladas previamente.

```plaintext
rsync -av /opt/mediawiki-actual/LocalSettings.php /var/www/html

rsync -av /opt/mediawiki-actual/images/ /var/www/html/images

rsync -av /opt/mediawiki-actual/resources/assets/ /var/www/html/resources/assets
```

Por último procedo a hacer el update como indica la documentación para la versión 1.39.x

```plaintext
php maintenance/update.php
```

## COMPROBACIONES

Realizo las comprobaciones que indica la documentación oficial:

Ver y editar páginas

Subir un fichero 

Visitar la página especial Version para comprobar que aparece la versión 1.39.x

Por último, elimino la copia en /opt de la instalación anterior

```plaintext
rm -rf /opt/mediawiki-actual
```