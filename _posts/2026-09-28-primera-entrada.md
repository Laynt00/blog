---
layout: post
title: "Creación Del Blog y Configuración de Cloudflare"
date: 2026-09-28
tags: [cloudflare, jekill, github, blogs]
---

Si estás leyendo esto significa que todo funciona correctamente.

Pero antes de nada, ¿qué es esto?

Bien. Mi idea es tener este blog como una especie de **bitácora** donde anotar los diferentes proyectos que voy realizando y así poder hacer un seguimiento de los mismos.

Teniendo un poco de contexto, veamos cómo estoy montando todo este chiringuito.

---
## ¿Por qué GitHub Pages + Jekyll?

Para este blog he decidido usar **GitHub Pages + Jekyll**.

Esto es, básicamente, un servicio de alojamiento estático desde repositorios que ofrece GitHub, donde Jekyll es una forma sencilla de hacer que se vea más como un blog y no como una página de los años 90.

![Pagina de los 90](/assets/img/1-WEB90.png)

Esto lo consigue convirtiendo archivos Markdown, a través de un motor de plantillas, en un sitio web HTML estático.

De esta forma me ahorro tener que configurar de nuevo otro servidor, como explicaré en posteriores posts.

---

## Crear el repositorio y activar GitHub Pages

Primero tenemos que crear un repositorio y activar GitHub Pages.

> **IMPORTANTE:** Antes de activar GitHub Pages el repositorio debe contener al menos un archivo, preferiblemente un `index.html` en la raíz. De lo contrario, GitHub Pages no se activará.

Una vez creado el archivo y hecho el primer commit, ya podemos activarlo.

Entramos en `Settings > Pages` y configuramos:

![Configuración GitHub Pages](/assets/img/1-github.png)

Guardamos los cambios y esperamos unos minutos para que GitHub empiece a construir el sitio.

Cuando termine, muestra un mensaje indicando que está publicado, pudiendo ya acceder a nuestro sitio web mediante la URL por defecto.

---

## Activar Jekyll

Ahora debemos activar Jekyll.

Podemos hacerlo de dos formas, en mi caso he optado por la más sencilla, ya que no necesito más al menos por ahora.

Tenemos que añadir un archivo `_config.yml` en la raíz del repositorio con el siguiente contenido:

![Configuración YML Jekyll](/assets/img/1-archivojek.png)

`jekyll-theme-minimal` es el tema oficial compatible con GitHub Pages, de estilo limpio y adecuado para mostrar proyectos personales.

Posteriormente crearemos una carpeta `_posts` y dentro introduciremos los artículos, teniendo en cuenta que cada artículo es un archivo con nombre:
(`año-mes-dia-titulo.md`) 
Y que el encabezado de cada artículo debe incluir los siguientes metadatos para que Jekyll lo reconozca:

![Metadatos artículo Jekyll](/assets/img/1-metadatos.png)

Si queremos añadir imágenes, crearemos una carpeta en la raíz del repositorio llamada `assets` y una subcarpeta llamada `img`, donde subiremos las imágenes que incrustaremos con Markdown de la siguiente forma:

![Formato de imágenes en Jekyll](/assets/img/1-imagenes.png)

Finalmente necesitamos eliminar el index.html y crear un index.md, en este caso mostrando las ultimas entradas
![index en Jekyll](/assets/img/1-index.png)

---

## Dominio propio y Cloudflare

Si uno solo desea un dominio por defecto como `laynt00.github.io/blog`, con esto ya estaría.

Pero en mi caso quiero un dominio propio, por lo que tras elegir `laynt.dev` como dominio, lo compré a un proveedor de los mismos.

Por defecto, este proveedor asigna sus propios servidores DNS al dominio, algo que está bien si no tienes necesidades especiales.

Sin embargo, en mi caso ya tenía pensado usar **Cloudflare** para gestionar la resolución DNS, por cuatro razones principales:

**Se, gu, ri, dad.**

No tiene más. En el caso de este blog no habría problemas, ya que el servidor sobre el que se aloja pertenece a GitHub.

Pero el servidor principal al que apunta `laynt.dev` es una **Raspberry Pi con Docker** en mi red doméstica, y no hace buen día como para abrir puertos ni exponer mi IP pública. Al menos por ahora, que no tengo la experiencia suficiente como para hacerlo de forma segura al 100%.

Usando el servicio **Cloudflare Tunnel**, mi Raspberry Pi establece una conexión saliente y cifrada a la red de Cloudflare.

Básicamente, cuando alguien visite mi dominio, el tráfico llega a Cloudflare y este lo reenvía de forma segura a través de ese "túnel" hasta mi servidor, de forma que mi IP queda oculta para el visitante.

---

## Configuración de Cloudflare

La configuración de Cloudflare es la siguiente.

Una vez añadido el dominio (`laynt.dev`), te asigna un par de nameservers personalizados:

![Nameservers de Cloudflare](/assets/img/1-nameservers.png)

Estos son los servidores que posteriormente debemos configurar en el proveedor de dominio.

Estos cambios pueden tardar en propagarse por todo el mundo unos minutos o varias horas.

Para verificarlo, en mi caso usé una herramienta que incluye mi propio proveedor de dominio. En cualquier caso, siempre podemos usar el comando:

`dig NS laynt.dev` para comprobarlo.

Una vez completada la configuración tenemos que tener en cuenta ahora todos los registros DNS del dominio se van a gestionar desde Cloudflare y no desde nuestro proveedor.

## Crear el registro CNAME

El siguiente paso es crear un registro DNS que va a permitir que blog.laynt.dev apunte correctamente a Github Pages. De este paso depende que github pueda verificar el domido y emitir un certificado SSL, si esto no sucediera cuando se intente acceder al blog podrían saltar avisos al usuario de seguridad ya que el sitio web no estaría confirmado.

Para el blog necesito un registro de tipo **CNAME**, que es el que se usa para apuntar un subdominio a otro dominio. Quiero que (`blog.laynt.dev`) apunte a (`laynt00.github.io`), que es la dirección que GitHub Pages asigna a mi repositorio.
![cname dominio](/assets/img/1-CNAME.png)
Hay un pequeño detalle que me costó entender al principio y quiero explicar bien porque es la causa de los problemas que tuve que solucionar. 
- **Proxied (nube naranja):** el tráfico pasa por los servidores de Cloudflare antes de llegar a su destino. Esto aporta ventajas como ocultar la IP real, protección DDoS y caché.
- **DNS only (nube gris):** el tráfico va directamente al destino, sin pasar por Cloudflare.
Cuando estás configurando un dominio personalizado en GitHub Pages, debes usar DNS only.

La razón es que GitHub necesita verificar que el dominio realmente le pertenece, y para ello hace una petición HTTP a una ruta concreta del dominio.

Si Cloudflare está en modo Proxied, esa petición la responde Cloudflare, no GitHub, y la verificación falla.

Por eso, durante la configuración inicial del dominio personalizado, hay que dejar el registro en DNS only.

Una vez que GitHub ha verificado el dominio y ha emitido el certificado SSL, ya se puede activar el proxy de Cloudflare si se quiere, aunque para un blog estático no es imprescindible.

Para comprobar que todo va como la seda de nuevo usamos el comando dig
`dig NS laynt.dev CNAME`
Si la respuesta muestra laynt00.github.io, significa que el registro se ha creado correctamente y que Github ya puede verlo.

Por último solo nos queda configurar el dominio personalizado en Github Pages. Para ello nos vamos al repositorio, Settings > Pages y dentro del apartado Custom domain escrribimos nuestro dominio:
![dominio personalizado github pages](/assets/img/1-domainGithub.png)
Guardamos y Github verificará el dominio haciendo una comprobación de DNS. Una vez Github verifica el dominio, comienza automáticamente el proceso de emisión del certificado SSL.
Durante el tiempo que tarde, la opción Enforce HTTPS aparece deshabilitada. Cuando el certificado esté emitido, la opción Enforce HTTPS se activa automáticamente.
En ese momento, marcamos la casilla y guardamos.

A partir de ahí, todo el tráfico a (`blog.laynt.dev`) se sirve por HTTPS, y los navegadores muestran el candado de conexión segura.
Para comprobar que el sitio está correctamente publicado, puedo hacer varias pruebas:
- Acceder a (`https://blog.laynt.dev`) desde el navegador y comprobar que carga el contenido.

- Verificar el certificado SSL haciendo clic en el candado de la barra de direcciones.

 -Comprobar que la URL por defecto de GitHub Pages (https://laynt00.github.io/blog) también funciona.

## Conclusión

La activación de GitHub Pages y la configuración del dominio personalizado es el paso que convierte un repositorio en un sitio web accesible desde tu propio dominio.

Es un proceso sencillo, pero requiere atención a los detalles, especialmente en lo que respecta al DNS y al certificado SSL.

Nos vemos en el siguiente post, un abrazo y haced el mundo más bonito.

