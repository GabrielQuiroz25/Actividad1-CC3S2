# Actividad 1 - CC3S2
## Preguntas de Reflexión

**Pregunta 1:** ¿Por qué surgió la necesidad de DevOps en el desarrollo de software?

Respuesta: *La necesidad de DevOps en el desarrollo de software surgió debido al notable avance tecnológico que influenció en un cambio de estrategias de las organizaciones y empresas que necesitan lanzar productos rápidamente y adaptarse a los cambios del mercado.*

**Pregunta 2:** Explica cómo la falta de comunicación y coordinación entre los equipos de desarrollo y operaciones en el pasado ha llevado a la creación de DevOps.

Respuesta: *La falta de comunicación y coordinación entre los equipos de desarrollo y operaciones daba como resultado lanzamientos de software ineficaces ya que el código que el equipo de desarrollo podía haber entregado al equipo de operaciones no resultaba funcional para el entorno de producción. Esto generaba fricciones y retrasos, lo que llevó a la creación de DevOps para mejorar la colaboración de ambos equipos y asegurar lanzamientos más rápidos y eficientes.*

**Pregunta 3:** Describe cómo el principio de mejora continua afecta tanto a los aspectos técnicos como culturales de una organización.

Respuesta: *El principio de mejora continua afecta positivamente en una organización ya que ofrece muchas ventajas tanto en los aspectos técnicos como culturales.
Desde el punto de vista técnico, permite cualidades como la confiabilidad, adaptabilidad y eficiencia de los procesos de entrega de software. Y culturalmente se fomenta un entorno colaborativo, asegurando responsabilidad, alentando una cultura de aprendizaje, ayudando a desmantelar los silos organizacionales.*

**Pregunta 4:** ¿Qué significa que DevOps no se trata solo de herramientas, individuos o procesos?

Respuesta: *Significa que DevOps es un cambio cultural en un organización. Es un enfoque diseñado para eliminar la fricción entre los silos dentro de una organización. Implica que haya un cambio en la mentalidad, los hábitos y esencialmente la cultura de quienes la integran.*


**Pregunta 5:** Según el texto, ¿cómo contribuyen los equipos autónomos y multifuncionales a una implementación exitosa de DevOps?

Respuesta: *Los equipos autónomos y multifuncionales agilizan la toma de decisiones, lo que resulta en respuestas rápidas a los cambios, fomentando así una cultura de colaboración y responsabilidad. De esta manera DevOps se implementa con éxito.*

----

Aplicaremos los conceptos de DevOps en un entorno práctico, configurando un pipeline básico de CI/CD para un proyecto de software y experimentando con la automatización de procesos en un entorno local utilizando Docker.

**Proyecto:** Utilizaremos una aplicación web sencilla utilizando **`Node.js`**. Esta aplicación tendrá una **API 
REST** básica con un único endpoint que devuelve un mensaje de *"Hello, World!"*.


## Instrucciones:

### 1. Configuración del entorno

1. Inicializamos el proyecto de Node.js dentro de la carpeta `/devops-practice`. Para ello ejecutamos los siguientes comandos en nuestra terminal:

    ```
    mkdir devops-practice
    cd devops-practice
    npm init -y   
    ```
2. Instalamos las dependencias necesarias:
    ```
    npm install express jest supertest
    ```
3. Creamos la estructura para nuestro proyecto:
    ```
    mkdir src tests
    touch src/app.js tests/app.test.js 
    ```
    La estructura resultante es la siguiente:

    ```
    ├── 📁 node_modules
    ├── package.json
    ├── package-lock.json
    ├── 📁 src
    │   └── app.js
    └── 📁 tests
        └── app.test.js
    ```

4. Implementamos la **API REST** en `src/app.js` :

    ```javascript
    const express = require('express');
    const app = express();
    
    app.get('/', (req, res) => {
        res.send('Hello, World!');
    });
    
    module.exports = app;
    
    
    if (require.main === module) {
        const port = process.env.PORT || 0; 
        app.listen(port, () => {
            console.log(`Server running on port ${port}`);
        });
    }
    
    ```

5. Escribimos un test básico en `tests/app.test.js` :

    ```javascript
    const request = require('supertest');
    const app = require('../src/app');
    
    describe('GET /', () => {
        let server;
    
        beforeAll(() => {
            server = app.listen(0); // Usar 0 permite al sistema asignar un puerto libre automáticamente
        });
    
        afterAll(() => {
            server.close(); // Cierra el servidor después de que las pruebas hayan terminado
        });
    
        it('should return Hello, World!', async () => {
            const res = await request(app).get('/');
            expect(res.statusCode).toEqual(200);
            expect(res.text).toBe('Hello, World!');
        });
    });
    ```
    
6. Configuramos el script de test en `package.json` :

    ```json
    {
      "name": "devops-practice",
      "version": "1.0.0",
      "main": "index.js",
      "scripts": {
        "test": "jest"
      },
      "keywords": [],
      "author": "",
      "license": "ISC",
      "description": "",
      "dependencies": {
        "express": "^4.19.2",
        "jest": "^29.7.0"
      },
      "devDependencies":{
        "jest":"^27.0.0",
        "supertest":"^6.1.3"
      }
      
    }
    ```
    
Hasta este punto podemos ver que las pruebas se ejecutan correctamente:
[![test1.png](https://i.postimg.cc/Zn6DP270/test1.png)](https://postimg.cc/XB784H00)

### 2. Pipeline CI/CD

#### Parte 1: Configuración de  integración continua (CI) con GitHub Actions

- Para ello crearemos un archivo de configuración para GitHub Actions:

1. Creamos la estructura para GitHub Actions: 
    ```
    mkdir -p .github/workflows
    touch .github/workflows/ci.yml
    ```

2.  Definimos el flujo de trabajo en .github/workflows/ci.yml:

    ```yml
    name: CI Pipeline
    
    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main
    
    jobs:
      build:
        runs-on: ubuntu-latest
    
        steps:
        - name: Checkout code
          uses: actions/checkout@v2
    
        - name: Set up Node.js
          uses: actions/setup-node@v2
          with:
            node-version: '14'
    
        - name: Install dependencies
          run: npm install
          working-directory: devops-practice
    
    
        - name: Run tests
          run: npm test

    ```

3.  Subimos el código a GitHub:
- Para esto crearemos un nuevo repositorio en GitHub y haremos '*push*'  al código:

    ```
     git init
     git add .
     git commit -m "Initial commit"
     git branch -M main
     git remote add origin <your-repo-url>
     git push -u origin main
    ```

#### Parte 2: Configuración de entrega continua (CD) con Docker

- Crearemos un archivo Docker para contenerizar la aplicación:

1. Creamos un archivo Dockerfile:

    ```Dockerfile
    # Usa la imagen oficial de Node.js
    FROM node:14
    
    # Establece el directorio de trabajo en el contenedor
    WORKDIR /app
    
    # Copia los archivos package.json y package-lock.json
    COPY package*.json ./
    
    # Instala las dependencias
    RUN npm install
    
    # Copia el resto de los archivos de la aplicación
    COPY . .
    
    # Expone el puerto en el que la aplicación correrá
    EXPOSE 3000
    
    # Comando para iniciar la aplicación
    CMD ["node", "src/app.js"]
    ```

2. Construímos la imagen de Docker: 

    ```
    docker build -t devops-practice .

    ```
    
    [![docker1.png](https://i.postimg.cc/9XwhhXQV/docker1.png)](https://postimg.cc/grp5yWhB)


3. Corremos el contenedor localmente:

    ```
    docker run -d -p 3000:3000 devops-practice

    ```
    
- Automatizamos el despliegue con GitHub Actions:

    1. Actualizamos el archivo .github/workflows/ci.yml para construir y desplegar la imagen de Docker:
    
        ```
        name: CI/CD Pipeline
        
        on:
          push:
            branches:
              - main
          pull_request:
            branches:
              - main
      
        jobs:
          build:
            runs-on: ubuntu-latest
        
            steps:
            - name: Checkout code
              uses: actions/checkout@v2
        
            - name: Set up Node.js
              uses: actions/setup-node@v2
              with:
                node-version: '14'
         
            - name: Install dependencies
              run: npm install
         
            - name: Run tests
              run: npm test
         
            - name: Build Docker image
              run: docker build -t devops-practice .
         
            - name: Run Docker container
             run: docker run -d -p 3000:3000 devops-practice
    
        ```
        
     2. Verificamos que la aplicación se despliegue correctamente localmente usando Docker:        
         - Abre un navegador web y accede a http://localhost:3000 para verificar que la aplicación esté 
    funcionando.
    [![docker2.png](https://i.postimg.cc/G28hVkfn/docker2.png)](https://postimg.cc/r0Tkttcf)
    


### 3. Automatización

 - Automatizamos la configuración y gestión del entorno local usando Docker Compose:

1. Creamos un archivo docker-compose.yml:

    ```yml
    version: '3.8'
    
    services:
      app:
        build: .
        ports:
          - "3000:3000"  
        environment:
          - NODE_ENV=production
    ```
2. Corremos la aplicación usando Docker Compose:

    ```
    docker compose up --build -d

    ```
    [![docker3.png](https://i.postimg.cc/bNCYyVjf/docker3.png)](https://postimg.cc/RNtxGRFg)


### 4. Documentación y evaluación

- Se ha documentado el proceso seguido, desde la configuración del entorno hasta la creación del pipeline 
CI/CD.


- **Evaluación de la experiencia**: Reflexiona sobre los beneficios de tener un pipeline automatizado y cómo esto 
reduce la fricción entre los equipos de desarrollo y operaciones.

Tener un pipeline automatizado resulta ser muy beneficioso para la organización ya que la entrega del software se vuelve mucho más rápida y frecuente puesto que se eliminan procesos manuales y repetitivos. 
Además, al automatizar las pruebas e implementaciones se reducen los errores que pueden ser cometidos al hacerlo manualmente, lo que resulta en una mayor consistencia en la calidad del software ya que el código se prueba automáticamente con cada cambio.

Respecto a la fricción entre los equipos de desarrollo y operaciones se verían reducidas ya que el código es desarrollado y probado en un entorno similar o igual al entorno de producción por lo que se evitarían conflictos a la hora de enviar el código.


