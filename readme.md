# Deploy con ngUniversal y functions
Se describe los pasos para hacer deploy de una app de Angular en con Firebase

## Requisitos
1. Instalar global Firebase Tools
2. Tener un proyecto de FIREBASE creado
3. Iniciar sesión en el proyecto de Firebase con FIREBASE CLI


## Instalación
1. Instala `ng add @nguniversal/express-engine --clientProject <<NOMBRE DEL PROYECTO>>`
2. Pegar los siguientes archivos al proyecto:

```
../
src/
    tsconfig.server.json
    main.server.ts
    app/
        app.server.module.ts
webpack.server.config.js
server.ts
```

## Preparar el proyecto para el deploy

### Configuración del proyecto

1. En ``src/app/app.module.ts`` incluir la configuración al **BrowserModule**
```ts
@NgModule({
    imports: [
        BrowserModule.withServerTransition({ appId: '<<NOMBRE DEL PORYECTO>>' }),
    ]
})

```

2. Copiar la configuración del `src/environments/environment.ts` al `src/environments/environment.prod.ts`

3. Incluir en el `/package.json` los siguientes scripts con el cambio en el nombre del proyecto en el comando `build:client-and-server-bundles`

```json
{
    "scripts": {
        "compile:server": "webpack --config webpack.server.config.js --progress --colors",
        "serve:ssr": "node dist/server.js",
        "build:ssr": "npm run build:client-and-server-bundles && npm run compile:server",
        "build:client-and-server-bundles": "ng build --prod <<NOMBRE DEL PROYECTO ANGULAR>> && ng run <<NOMBRE DEL PROYECTO ANGULAR>>:server:production"
    },
}
```

4. Incluir en `/angular.json` una nueva arquitectura:

```json
{
    "projects": {
        "NOMBRE DEL PROYECTO ANGULAR":{
            "architect": {
                "server": {
                    "builder": "@angular-devkit/build-angular:server",
                    "options": {
                        "outputPath": "dist/server",
                        "main": "src/main.server.ts",
                        "tsConfig": "src/tsconfig.server.json"
                    },
                    "configurations": {
                        "production": {
                            "fileReplacements": [{
                                "replace": "src/environments/environment.ts",
                                "with": "src/environments/environment.prod.ts"
                            }]
                        }
                    }
                }
            }
        }
    }
}
```

5. Instalar las siguientes dependencias:
`npm install ws xhr2 bufferutil utf-8-validate  -D`


6. En la consola correr el comando para crear el directorio de producción `npm run build:ssr`
7. Si se desea probar el proyecto prod, ejecutar `npm run serve:ssr`


### Configuración de firebase

1. `firebase init`
2. selecciona *hosting* y *functions*

3. Crear la siguiente functión en el archivo `functions/index.ts`

```ts
import * as functions from 'firebase-functions';
const universal = require(`${process.cwd()}/dist/server`).app;

export const <<NOMBRE DE LA FUNCIÓN>> = functions.https.onRequest(universal);
```

4. elige la carpeta a desplegar en el hosting en el `/firebase.json`
```json
{
  "hosting": {
    "public": "<<CARPETA DEL PROYECTO BUILD>>",
     // ...
    "rewrites": [
      {
        "source": "**",
        "function": "<<NOMBRE DE LA FUNCIÓN>>"
      }
    ]
  }
}

```

5. Ingresa a la carpeta */functions* e ingresa el comando `npm i fs-extra`
6. Copiar archivo llamado **cp-angular.js** en la carpeta functions:
7. En el `functions/package.json` configurar el comando:

```json
{
    "scripts": {
        "build": "node cp-angular && tsc"
    }
}
```




### Hacer deploy
1. Correr en la consola `npm run build`
2. Salir de la carpeta y si se desea probar, correr `firebase serve`
3. Hacer el deploy con `firebase deploy`


## Actualizar deploy
1. Correr el comando `npm run build:ssr`
2. Ingresar a la carpeta de functions `cd functions`
3. Correr en la consola `npm run build`
4. Salir de la carpeta y si se desea probar, correr `firebase serve`
5. Hacer el deploy con `firebase deploy`



## Documentación

https://fireship.io/lessons/angular-universal-firebase/