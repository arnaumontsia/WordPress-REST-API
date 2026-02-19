# WordPress-REST-API
Tutorial per a fer gastar la REST API de WordPress per a crear pàgines amb HTML i fer migracions de contingut.


## 1 - Configuració de la màquina virtual

El primer pas de tots serà crear la màquina virtual per al server, a la qual podem posar les caracteristiques que vulguem mentre respectem les minimes (CPU: 1 GHz, 1 nucli / RAM: 512 MB / Disc: 2.5 GB), aquí es pot veure les capacitats per les quals jo he optat a més de les configuracions:

<img width="607" height="204" alt="image" src="https://github.com/user-attachments/assets/8c8a861d-0005-4b50-974d-5e0c13ccee3b" />
<img width="607" height="204" alt="image" src="https://github.com/user-attachments/assets/9859ba7a-9d09-4420-82af-13ef05734dd6" />
<img width="620" height="343" alt="image" src="https://github.com/user-attachments/assets/fb675870-4a83-41f5-8c42-23ef5f00125f" />


## 2 - Configuració Ubuntu Server i Docker

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


## 3 - Creació d'usuari editor

El seguent pas serà crear un usuari editor, amb el que més endevant realitzarem tota la feina de muntar contingut mitjançant la API:

<img width="469" height="460" alt="image" src="https://github.com/user-attachments/assets/e137d982-197b-4e93-bd09-34f52a705fe7" />

Clicarem a afegir un usuari i plenarem el formulari, la part més important és que posem com a perfil **Editor**:

<img width="auto" height="500" alt="image" src="https://github.com/user-attachments/assets/e40fb779-07c3-41c3-bb08-bb752a17d2c4" />

## 4 - Instal·lació i activació dels plugins

Ara accedirem a la pestanya de Plugins, on buscarem jwt-authentication-for-wp-rest-api, i instal·larem i activarem el plugin de la imatge, aquest ens permetrà autenticar-nos posteriorment per token:

<img width="auto" height="270" alt="image" src="https://github.com/user-attachments/assets/77469c9b-f80e-4065-8cc6-219479a1c76f" />

A més, per evitar errors de CORS, tambe afegiré el seguent plugin:

<img width="auto" height="270" alt="image" src="https://github.com/user-attachments/assets/bcaf1f05-25d9-466e-839f-fba37e5fdc39" />

Aquest crearà una pestanya, a la qual accediré i configuraré per a indicar que es puguin fer POST's de contingut des de qualsevol IP:

<img width="auto" height="600" alt="image" src="https://github.com/user-attachments/assets/51189726-0a3b-4cf1-9284-806f21c8841e" />


## 5 - Configuracions de WP al contenidor de docker

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


## 6 - Obtenció del token

Ara sols quedarà crear les pàgines, però primer haurem de conseguir un token per afegir al script i poder autenticar-se.

Per a fer això, jo accedire al bash del docker on posaré el seguent curl:

```
curl -X POST "https://IP_del_WP/wp-json/jwt-auth/v1/token" \
-H "Content-Type: application/json" \
-d '{"username":"usuari","password":"contrasenya"}'
```

Això ens retornarà un token el qual copiarem per a continuar amb el seguent pas.


## 7 - Creació del script

Per últim, sols ens queda crear el script en JavaScript per poder crear pàgines, jo he usat el següent:
 
```
const JWT_TOKEN = "el_teu_token";
const ESTAT = "draft";

const htmlContent = `
<html>
`;

async function muntarHTML() {
  try {
    console.log("Enviant contingut a WP...");

    const res = await fetch(
      "URL_RESTAPI_pàgines",
      {
        method: "POST",
        headers: {
          "Authorization": `Bearer ${JWT_TOKEN}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          title: "Pàgina muntada amb JS",
          content: htmlContent,
          status: ESTAT
        })
      }
    );

    const data = await res.json();

    if (!res.ok) {
      console.error("Error tornat per WP", data);
      alert("Error al muntar el contingut");
      return;
    }

    console.log("Contingut muntat correctament:", data);
    alert("Contingut muntat correctament");

  } catch (error) {
    console.error("Error de conexió:", error);
    alert("Error de conexió amb WP");
  }
}

muntarHTML();

```

Aquest code JS és una plantilla que ens permet migrar code html a pàgines de WP.

Dalt de tot emmagatzemem en variables el token i el estat de la pàgina (publicat o borrador), tot seguit usant fetch a la REST API fem un POST del cotingut a la pàgina, depenent del resultat ens donarà un code de error o de que surt correctament. En cas de error podem fer troubleshooting des de la consola, on guardem el code de error i el podem interpretar per atacar a les errades.

Aquest JS el vinculo al següent html:
```
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Muntar a WP</title>
</head>
<body>

<button id="muntar" onclick="muntarHTML()">Muntar a WP</button>

<script src="wprestapi.js"></script>
</body>
</html>
```

Aquest ens permet clicar per a tornar a executar el code JS per assegurar-nos de la muntada.

Cal tenir en compte que sols es pot executar des de un server web, en el meu cas el mètode més ràpid és usar xampp, que es pot instal·lar a Windows i és molt simple.

<img width="auto" height="400" alt="image" src="https://github.com/user-attachments/assets/ed7924c6-41dd-44d5-b5e2-d4f6f64c5a64" />

## 8 - Format del html

El format del html ha de ser específic ja que WP no accepta qualsevol cosa. Sols deus incluir al script el contingut de dintre del body perquè tot el demés es rebutjat per WP.

Un altre punt a tindre en compte son els estils, la manera més fàcil de afegir-los és utilitzant <script>, i afegit dins tot el css, però per a que WP el agafe sencer, ha de estar a la mateixa linea.

Per concluir, aquí hi ha un exemple de html vàlid:

```
<style>html{box-sizing:border-box;scroll-behavior:smooth;font-size:100%}body{font-family:Arial,Helvetica,sans-serif;margin:0;padding:0;background-color:#f4f4f4;color:#333;line-height:1.6;display:flex;justify-content:center;align-items:center;height:100%}header{background-color:#004a99;color:#fff;padding:1.5625rem 2.5rem;text-align:center}nav{background-color:#4CAF50;padding:.9375rem;text-align:center;position:sticky;top:0;z-index:100}main{max-width:1000px;margin:1.875rem auto;padding:0 1.25rem}section{background-color:#fff;padding:1.5625rem 1.875rem;margin-bottom:1.5625rem;border-radius:.5rem;box-shadow:0 2px 5px rgba(0,0,0,.05)}footer{background-color:#333;color:#f4f4f4;text-align:center;padding:1.5625rem;margin-top:2.5rem}h1{color:#004a99;font-size:2.8rem;margin-top:0}h2{color:#6ea8f0;font-size:2.2rem;border-bottom:3px solid #eee;padding-bottom:.5rem;margin-top:2.5rem}h3{color:#555;font-size:1.6rem}p{margin-bottom:.9375rem}nav a{color:#fff;text-decoration:none;font-weight:700;padding:.9375rem 1.25rem;transition:background-color .3s;display:inline-block}nav a:hover{background-color:#45a049}main a{color:#004a99;text-decoration:none;font-weight:700}main a:hover{text-decoration:underline}ol,ul{margin-left:1.25rem;padding-left:.9375rem}li{margin-bottom:.625rem}blockquote{border-left:5px solid #4CAF50;margin:1.25rem 0;padding:.625rem 1.25rem;background-color:#f9f9f9;font-style:italic;color:#555}figure{margin:1.25rem 0;text-align:center}figcaption{font-style:italic;color:#777;margin-top:.3125rem;font-size:.9rem}.flex{display:flex}.center{text-align:center;justify-content:center}body{background-color:#fff}.boton{display:block;width:450px;padding:30px 40px;font-size:1.8rem;font-weight:700;color:#fff;text-decoration:none;text-align:center;border-radius:10px;transition:transform .2s ease,box-shadow .2s ease;box-shadow:0 5px 12px rgba(0,0,0,.15);margin:.5rem}a,a:active,a:hover,a:visited{text-decoration:none;color:inherit}.boton:hover{transform:translateY(-4px);box-shadow:0 8px 20px rgba(0,0,0,.2)}.digi{background-color:#004a99}.soste{background-color:#4CAF50}.intra{background-color:#607D8B}.benvinguda{text-align:center;justify-content:center;margin:auto;box-shadow:none;border:none}</style>
<div>
    <section class="benvinguda">
        <article>
            <h1>Benvingut a Montsià 30</h1>
        </article>
        <article class="flex center">
            <a href="digitalitzacio/pag1_carrusel/digi1.html"><button class="digi boton">Pàgina
                    Digitalització</button></a>
            <a href="recursos/proximament.html"><button class="soste boton">Pàgina Sostenibilitat</button></a>
        </article>
        <article class="flex center">
            <a href="intranet/login.html"><button class="intra boton">Intranet</button></a>
        </article>
    </section>
</div>
```

## 9 - Comprovació final

Per últim, després d'executar el JS, accedirem al WP, a la pestanya de Pàgines, on trovarem la pàgina que hem muntat.

<img width="1180" height="283" alt="image" src="https://github.com/user-attachments/assets/8f46208f-151e-43f8-81d7-b85232f3b750" />

I la podrem visualitzar i comprovar que queda com voliem.

<img width="auto" height="900" alt="image" src="https://github.com/user-attachments/assets/447fec6c-e60a-48d9-943c-3345049b721e" />





