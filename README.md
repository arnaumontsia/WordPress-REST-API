# WordPress-REST-API
Tutorial per a fer gastar la REST API de WordPress per a crear pàgines amb HTML i fer migracions de contingut.

## Configuració de la màquina virtual

El primer pas de tots serà crear la màquina virtual, a la qual podem posar les caracteristiques que vulguem mentre respectem les minimes (CPU: 1 GHz, 1 nucli / RAM: 512 MB / Disc: 2.5 GB), aquí es pot veure les capacitats per les quals jo he optat a més de les configuracions:

<img width="607" height="204" alt="image" src="https://github.com/user-attachments/assets/8c8a861d-0005-4b50-974d-5e0c13ccee3b" />
<img width="607" height="204" alt="image" src="https://github.com/user-attachments/assets/9859ba7a-9d09-4420-82af-13ef05734dd6" />
<img width="620" height="343" alt="image" src="https://github.com/user-attachments/assets/fb675870-4a83-41f5-8c42-23ef5f00125f" />

## Configuració Ubuntu Server i Docker

Tot seguit fem la instal·lació normal de la màquina utilitzant el sistema Ubuntu Server 25.04, i el primer que farem una vegada està instal·lada és accedir a el arxiu de configuració de xarxa i assignar una IP estàtica:

<img width="731" height="349" alt="image" src="https://github.com/user-attachments/assets/5bd83f2c-0857-428b-a195-6c8b32395bc7" />

Amb una IP estàtica podrem instal·lar docker i tot seguit, crear el docker compose per a allotjar els serveis necesaris per al WP, aquí podem vorer el code que he gastat per a això (s'haura de substituir la paraula contrasenya amb les contrasenyes que vulguis posar):

```
services:
  db:
    image: mysql:8.0
    container_name: wordpress_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: contrasenya
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: contrasenya
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: contrasenya
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html

volumes:
  db_data:
  wp_data:

```

Tot seguit, usare docker compose up per aixecar el servei, el qual s'instal·larà i tot seguit estarà actiu per poder accedir al WP:

<img width="1843" height="959" alt="image" src="https://github.com/user-attachments/assets/2393faba-3600-4118-a244-085d8e56f4de" />

## Creació d'usuari editor

El seguent pas serà crear un usuari editor, amb el que més endevant realitzarem tota la feina de muntar contingut mitjançant la API:

<img width="469" height="460" alt="image" src="https://github.com/user-attachments/assets/e137d982-197b-4e93-bd09-34f52a705fe7" />

Clicarem a afegir un usuari i plenarem el formulari, la part més important és que posem com a perfil **Editor**:

<img width="auto" height="500" alt="image" src="https://github.com/user-attachments/assets/e40fb779-07c3-41c3-bb08-bb752a17d2c4" />

## Instal·lació i activació dels plugins

Ara accedirem a la pestanya de Plugins, on buscarem jwt-authentication-for-wp-rest-api, i instal·larem i activarem el plugin de la imatge:

<img width="auto" height="270" alt="image" src="https://github.com/user-attachments/assets/77469c9b-f80e-4065-8cc6-219479a1c76f" />

A més, per evitar errors de CORS, tambe afegiré el seguent plugin:

<img width="auto" height="270" alt="image" src="https://github.com/user-attachments/assets/bcaf1f05-25d9-466e-839f-fba37e5fdc39" />

Aquest crearà una pestanya, a la qual accediré i configuraré per a indicar que es puguin fer POST's de contingut des de qualsevol IP:

<img width="auto" height="600" alt="image" src="https://github.com/user-attachments/assets/51189726-0a3b-4cf1-9284-806f21c8841e" />



## Configuracions de WP al contenidor de docker

Una vegada instal·lat i activat el plugin, accedirem a el servidor on obrirem la linea de comandes de aquest i a més, farem un update i instal·larem nano per editar arxius:

<img width="802" height="50" alt="image" src="https://github.com/user-attachments/assets/fad28f65-d1cd-4ed4-b813-5029fc427b6b" />

Un cop instal·lat, editarem l'arxiu /var/www/html/.htaccess, al qual afegirem les seguents linies al principi del arxiu:

```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
</IfModule>
```

Després, editarem l'arxiu /var/www/html/wp-config.php, al qual afegirem les seguents linies abans de la linea que es veu a la imatge:

<img width="591" height="23" alt="image" src="https://github.com/user-attachments/assets/3e968540-f1d4-40d3-9b80-66e7917da30d" />


```
define('JWT_AUTH_SECRET_KEY', 'clau');
define('JWT_AUTH_CORS_ENABLE', true);
```





