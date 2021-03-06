---
title: Trabajar con el registro de npm
intro: 'Puedes configurar npm para publicar paquetes en {% data variables.product.prodname_registry %} y para usar los paquetes almacenados en {% data variables.product.prodname_registry %} como dependencias en un proyecto npm.'
product: '{% data reusables.gated-features.packages %}'
redirect_from:
  - /articles/configuring-npm-for-use-with-github-package-registry
  - /github/managing-packages-with-github-package-registry/configuring-npm-for-use-with-github-package-registry
  - /github/managing-packages-with-github-packages/configuring-npm-for-use-with-github-packages
  - /packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages
  - /packages/guides/configuring-npm-for-use-with-github-packages
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
shortTitle: Registro de npm
---

{% data reusables.package_registry.packages-ghes-release-stage %}
{% data reusables.package_registry.packages-ghae-release-stage %}

**Nota:** Cuando instalas o publicas una imagen de docker, {% data variables.product.prodname_registry %} no es compatible con capas externas, tales como imágenes de Windows.

## Límites para las versiónes de npm publicadas

Si estableces más de 1,000 versiones de paquetes de npm en el {% data variables.product.prodname_registry %}, podrías notar que ocurren problemas de rendimiento y de tiempos excedidos durante el uso.

En el futuro, para mejorar el rendimiento del servicio, no podrás publicar más de 1,000 versiones de un paquete en {% data variables.product.prodname_dotcom %}. Cualquier versión que se publique antes de llegar a este límite aún será legible.

Si llegas a este límite, considera borrar las versiones del paquete o contacta a soporte para recibir ayuda. Cuando se aplique este límite, actualizaremos nuestra documentación con una forma de dar soluciones para él. Para obtener más información, consulta la sección "{% ifversion fpt or ghes > 3.0 or ghec %}[Borrar y restablecer un paquete](/packages/learn-github-packages/deleting-and-restoring-a-package){% elsif ghes < 3.1 or ghae %}[Borrar un paquete](/packages/learn-github-packages/deleting-a-package){% endif %}" o [Contactar a Soporte](/packages/learn-github-packages/about-github-packages#contacting-support)".

## Autenticarte en {% data variables.product.prodname_registry %}

{% data reusables.package_registry.authenticate-packages %}

{% data reusables.package_registry.authenticate-packages-github-token %}

### Autenticarte con un token de acceso personal

{% data reusables.package_registry.required-scopes %}

Puedes autenticarte en {% data variables.product.prodname_registry %} con npm al editar tu archivo *~/.npmrc* por usuario para incluir tu token de acceso personal o al iniciar sesión en npm en la línea de comando por medio tu nombre de usuario y token de acceso personal.

Para autenticarte agregando tu token de acceso personal a tu archivo *~/.npmrc*, edita el archivo *~/.npmrc* para que tu proyecto incluya la siguiente línea, reemplazando a{% ifversion ghes or ghae %}*HOSTNAME* con el nombre de host de {% data variables.product.product_location %} y a {% endif %}*TOKEN* con tu token de acceso personal. Crea un nuevo archivo *~/.npmrc* si no existe uno.

{% ifversion ghes %}
Para obtener más información acerca de cómo crear un paquete, consulta la [documentación maven.apache.org](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html).
{% endif %}

```shell
//{% ifversion fpt or ghec %}npm.pkg.github.com{% else %}npm.<em>HOSTNAME</em>/{% endif %}/:_authToken=<em>TOKEN</em>
```

{% ifversion ghes %}
Por ejemplo, los proyectos *OctodogApp* y *OctocatApp* publicarán en el mismo repositorio:

```shell
$ npm login --registry=https://npm.pkg.github.com
> Username: <em>USERNAME</em>
> Password: <em>TOKEN</em>
> Email: <em>PUBLIC-EMAIL-ADDRESS</em>
```
{% endif %}

Para autenticarte al iniciar sesión en npm, usa el comando `npm login`, reemplaza *USERNAME* por tu nombre de usuario de {% data variables.product.prodname_dotcom %}, *TOKEN* por tu token de acceso personal y *PUBLIC-EMAIL-ADDRESS* por tu dirección de correo electrónico.

Si el {% data variables.product.prodname_registry %} no es tu registro de paquetes predeterminado para utilizar npm y quieres utilizar el comando `npm audit`, te recomendamos que utilices el marcador `--scope` con el propietario del paquete cuando te autentiques en {% data variables.product.prodname_registry %}.

{% ifversion ghes %}
Para obtener más información acerca de cómo crear un paquete, consulta la [documentación maven.apache.org](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html).
{% endif %}

```shell
$ npm login --scope=@<em>OWNER</em> --registry=https://{% ifversion fpt or ghec %}npm.pkg.github.com{% else %}npm.<em>HOSTNAME</em>/{% endif %}

> Username: <em>USERNAME</em>
> Password: <em>TOKEN</em>
> Email: <em>PUBLIC-EMAIL-ADDRESS</em>
```

{% ifversion ghes %}
Por ejemplo, los proyectos *OctodogApp* y *OctocatApp* publicarán en el mismo repositorio:

```shell
$ npm login --scope=@<em>OWNER</em> --registry=https://<em>HOSTNAME</em>/_registry/npm/
> Username: <em>USERNAME</em>
> Password: <em>TOKEN</em>
> Email: <em>PUBLIC-EMAIL-ADDRESS</em>
```
{% endif %}

## Publicar un paquete

{% note %}

**Nota:** Los nombres y alcances de los paquetes deben escribirs exclusivamente en minúscula.

{% endnote %}

De forma predeterminada, {% data variables.product.prodname_registry %} publica un paquete en el repositorio de {% data variables.product.prodname_dotcom %} que especifiques en el campo nombre del archivo *package.json*. Por ejemplo, si publicas un paquete denominado `@my-org/test` en el repositorio de `my-org/test` {% data variables.product.prodname_dotcom %}. Puedes agregar un resumen para la página de descripción del paquete al incluir un archivo *README.md* en el directorio de tu paquete. Para obtener más información, consulta "[Trabajar con package.json](https://docs.npmjs.com/getting-started/using-a-package.json)" y "[Cómo crear módulos Node.js](https://docs.npmjs.com/getting-started/creating-node-modules)" en la documentación de npm.

Puedes publicar varios paquetes en el mismo repositorio de {% data variables.product.prodname_dotcom %} al incluir un campo `URL` en el archivo *package.json*. Para obtener más información, consulta "[Publicar varios paquetes en el mismo repositorio](#publishing-multiple-packages-to-the-same-repository)".

Puedes configurar la asignación de alcance de tu proyecto por medio de un archivo *.npmrc* local en el proyecto o mediante la opción `publishConfig` en *package.json*. {% data variables.product.prodname_registry %} solo admite paquetes npm con alcance definido. Los paquetes con alcance definido tienen nombres con el formato de `@owner/name`. Además, siempre comienzan con un símbolo `@`. Es posible que tengas que actualizar el nombre en tu *package.json* para usar el nombre de alcance definido. Por ejemplo, `"name": "@codertocat/hello-world-npm"`.

{% data reusables.package_registry.viewing-packages %}

### Publicar un paquete por medio de un archivo *.npmrc* local

Puedes usar un archivo *.npmrc* para configurar la asignación del alcance de tu proyecto. En el archivo *.npmrc*, usa la URL y el propietario de la cuenta de {% data variables.product.prodname_registry %} para que {% data variables.product.prodname_registry %} sepa dónde enrutar las solicitudes del paquete. Usar un archivo *.npmrc* impide que otros programadores publiquen accidentalmente el paquete en npmjs.org en lugar de {% data variables.product.prodname_registry %}.

{% data reusables.package_registry.authenticate-step %}
{% data reusables.package_registry.create-npmrc-owner-step %}
{% data reusables.package_registry.add-npmrc-to-repo-step %}
1. Verifica el nombre de tu paquete en el *package.json* de tu proyecto. El campo `name (nombre)` debe contener el alcance y el nombre del paquete. Por ejemplo, si tu paquete se denomina "test" (prueba) y vas a publicar en la organización "My-org" de {% data variables.product.prodname_dotcom %}, el campo `name (nombre)` de tu *package.json* debería ser `@my-org/test`.
{% data reusables.package_registry.verify_repository_field %}
{% data reusables.package_registry.publish_package %}

### Publicar un paquete por medio de `publishConfig` en el archivo *package.json*

Puedes usar el elemento `publishConfig` en el archivo *package.json* para especificar el registro en el que deseas que se publique el paquete. Para obtener más información, consulta "[publishConfig](https://docs.npmjs.com/files/package.json#publishconfig)" en la documentación de npm.

1. Edita el archivo *package.json* de tu paquete e incluye una entrada de `publishConfig`.
  {% ifversion ghes %}
  Para obtener más información acerca de cómo crear un paquete, consulta la [documentación maven.apache.org](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html).
  {% endif %}
  ```shell
  "publishConfig": {
    "registry":"https://{% ifversion fpt or ghec %}npm.pkg.github.com{% else %}npm.<em>HOSTNAME</em>/{% endif %}"
  },
  ```
  {% ifversion ghes %}
  Por ejemplo, los proyectos *OctodogApp* y *OctocatApp* publicarán en el mismo repositorio:
   ```shell
   "publishConfig": {
     "registry":"https://<em>HOSTNAME</em>/_registry/npm/"
   },
  ```
  {% endif %}
{% data reusables.package_registry.verify_repository_field %}
{% data reusables.package_registry.publish_package %}

## Publicar varios paquetes en el mismo repositorio

Para publicar varios paquetes en el mismo repositorio, puedes incluir la URL del repositorio de {% data variables.product.prodname_dotcom %} en el campo `repository (repositorio)` del archivo *package.json* para cada paquete.

Para asegurarte de que la URL del repositorio sea correcta, reemplaza REPOSITORY por el nombre del repositorio que contiene el paquete que deseas publicar y OWNER por el nombre de la cuenta de usuario o de organización en {% data variables.product.prodname_dotcom %} que posee el repositorio.

{% data variables.product.prodname_registry %} coincidirá con el repositorio en base a la URL, en lugar de basarse en el nombre del paquete.

```shell
"repository":"https://{% ifversion fpt or ghec %}github.com{% else %}<em>HOSTNAME</em>{% endif %}/<em>OWNER</em>/<em>REPOSITORY</em>",
```

## Instalar un paquete

Puedes instalar paquetes desde {% data variables.product.prodname_registry %} al agregar los paquetes como dependencias en el archivo *package.json* para tu proyecto. Para obtener más información sobre el uso de un *package.json* en tu proyecto, consulta "[Trabajar con package.json](https://docs.npmjs.com/getting-started/using-a-package.json)" en la documentación de npm.

Por defecto, puedes agregar paquetes de una organización. Para obtener más información, consulta la sección "[Instalar paquetes de otras organizaciones](#installing-packages-from-other-organizations)".

También necesitas agregar el archivo *.npmrc* a tu proyecto para que todas las solicitudes para instalar paquetes se {% ifversion ghae %}enruten a{% else %}pasen por{% endif %} el {% data variables.product.prodname_registry %}. {% ifversion fpt or ghes or ghec %}Cuando enrutas todas las solicitudes de paquetes a través del {% data variables.product.prodname_registry %}, puedes utilizar tanto los paquetes dentro como fuera del alcance de *npmjs.org*. Para obtener más información, consulta la sección de "[npm-scope](https://docs.npmjs.com/misc/scope)" en la documentación de npm.{% endif %}

{% ifversion ghae %}
Predeterminadamente, solo puedes utilizar paquetes de npm hospedados en tu empresa y no podrás utilizar paquetes fuera del alcance. Para obtener más información sobre el alcance de los paquetes, consulta la sección de "[npm-scope](https://docs.npmjs.com/misc/scope)" en la documentación de npm. En caso de que se requiera, se puede habilitar el {% data variables.product.prodname_dotcom %} en un proxy de nivel superior como npmjs.org. Una vez que se habilite un proxy de nivel superior, si un paquete que se haya solicitado no se encuentra en tu empresa, el {% data variables.product.prodname_registry %} hará una solicitud de proxy a npmjs.org.
{% endif %}

{% data reusables.package_registry.authenticate-step %}
{% data reusables.package_registry.create-npmrc-owner-step %}
{% data reusables.package_registry.add-npmrc-to-repo-step %}
4. Configura *package.json* en tu proyecto para usar el paquete que estás instalando. Para agregar las dependencias de tu paquete al archivo *package.json* para {% data variables.product.prodname_registry %}, especifica el nombre del paquete de alcance completo, como `@my-org/server`. Para paquetes de *npmjs.com*, especifica el nombre completo, como `@babel/core` o `@lodash`. Por ejemplo, el archivo *package.json* a continuación utiliza el paquete `@octo-org/octo-app` como una dependencia.

  ```json
  {
    "name": "@my-org/server",
    "version": "1.0.0",
    "description": "Server app that uses the @octo-org/octo-app package",
    "main": "index.js",
    "author": "",
    "license": "MIT",
    "dependencies": {
      "@octo-org/octo-app": "1.0.0"
    }
  }
  ```
5. Instala el paquete.

  ```shell
  $ npm install
  ```

### Instalar paquetes de otras organizaciones

Por defecto, solo puedes usar paquetes de {% data variables.product.prodname_registry %} de una organización. Si te gustaría enrutar las solicitudes de paquetes a organizaciones y usuarios múltiples, puedes agregar líneas adicionales a tu archivo de *.npmrc*, reemplazando a {% ifversion ghes or ghae %}*HOSTNAME* con el nombre de host de {% data variables.product.product_location %} y a {% endif %}*OWNER* con el nombre de la cuenta de usuario o de organización a la que pertenece el repositorio que contiene tu proyecto.

{% ifversion ghes %}
Para obtener más información acerca de cómo crear un paquete, consulta la [documentación maven.apache.org](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html).
{% endif %}

```shell
@<em>OWNER</em>:registry=https://{% ifversion fpt or ghec %}npm.pkg.github.com{% else %}npm.<em>HOSTNAME</em>{% endif %}
@<em>OWNER</em>:registry=https://{% ifversion fpt or ghec %}npm.pkg.github.com{% else %}npm.<em>HOSTNAME</em>{% endif %}
```

{% ifversion ghes %}
Por ejemplo, los proyectos *OctodogApp* y *OctocatApp* publicarán en el mismo repositorio:

```shell
@<em>OWNER</em>:registry=https://<em>HOSTNAME</em>/_registry/npm
@<em>OWNER</em>:registry=https://<em>HOSTNAME</em>/_registry/npm
```
{% endif %}

{% ifversion ghes %}
## Utilizar el registro oficial de NPM

El {% data variables.product.prodname_registry %} te permite acceder al registro oficial de NPM en `registry.npmjs.com`, si tu administrador de {% data variables.product.prodname_ghe_server %} habilitó esta característica. Para obtener más información, consulta la sección [Conectarse al registro oficial de NPM](/admin/packages/configuring-packages-support-for-your-enterprise#connecting-to-the-official-npm-registry).
{% endif %}

## Leer más

- "{% ifversion fpt or ghes > 3.0 or ghec %}[Deleting and restoring a package](/packages/learn-github-packages/deleting-and-restoring-a-package){% elsif ghes < 3.1 or ghae %}[Deleting a package](/packages/learn-github-packages/deleting-a-package){% endif %}"
