# bot-telegram-poli

## Integrantes Subgrupo 2 Integración Continúa Grupo B02 
    - BANOS OSORIO RAHOMIR DE JESUS
    - BETANCUR TORRES JAIDER
    - GARCÍA GARCÍA JHON
    - HERNÁNDEZ TRIANA YULIAN ANTONIO
    - HERRERA RODRIGUEZ WEIMAR JAIR
    - POLANIA LOZANO ANDRES CAMILO

## Variables de docker-compose.yml

1. version
version: '3.8': Especifica la versión de la sintaxis de Docker Compose que se está utilizando.

2. services
Define los servicios que se ejecutarán en contenedores.

    a. db
image: mysql:5.7: Utiliza la imagen de MySQL versión 5.7.
environment: Variables de entorno para configurar la base de datos
MYSQL_ROOT_PASSWORD: rootpassword: Contraseña del usuario root de MySQL.
MYSQL_DATABASE: telegram-bot: Nombre de la base de datos que se creará al iniciar el contenedor.
MYSQL_USER: myuser: Nombre de un usuario adicional que se creará.
MYSQL_PASSWORD: mypassword: Contraseña para el usuario adicional.
ports: Mapeo de puertos
"3306:3306": Expone el puerto 3306 del contenedor al puerto 3306 del host, permitiendo conexiones a MySQL, al momento de clonar el repositorio a la maquina local se debe verificar que el puerto 3306 no este ocupado. 
volumes: Persistencia de datos
db_data:/var/lib/mysql: Monta un volumen llamado db_data en el directorio donde MySQL almacena sus datos, asegurando que los datos persistan incluso si el contenedor se reinicia.

    b. app
build: Instrucciones para construir la imagen del contenedor:
context: .: Especifica el contexto de construcción, que es el directorio actual donde se encuentra el Dockerfile.
environment: Variables de entorno para la aplicación:
DATABASE_HOST: db: Nombre del servicio de la base de datos (referencia al contenedor db).
DATABASE_PORT: 3306: Puerto en el que se está ejecutando MySQL.
DATABASE_NAME: telegram-bot: Nombre de la base de datos a utilizar.
DATABASE_USER: myuser: Nombre del usuario para acceder a la base de datos.
DATABASE_PASSWORD: mypassword: Contraseña del usuario para la base de datos.
depends_on: Indica que el servicio app depende del servicio db, asegurando que el contenedor de la base de datos se inicie antes que la aplicación.
ports: Mapeo de puertos:
"8000:8000": Expone el puerto 8000 del contenedor al puerto 8000 del host.
restart: always: Configura el contenedor para que se reinicie automáticamente si se detiene.

3. volumes
db_data:: Define un volumen llamado db_data que se utiliza para la persistencia de datos de MySQL.

El archivo docker-compose.yml presenta un comportamiento en el que se reinicia continuamente hasta que el codigo se sube correctamente y la base de datos se despliega. 
Esto significa que el sistema no comienza a funcionar completamente hasta que la base de datos está lista.


## Variables y configuraciones en main.py

1. Importaciones
import telebot: Importa la biblioteca para interactuar con la API de Telegram.
from telebot import types: Importa tipos necesarios para crear botones y otras interacciones.
import mysql.connector: Importa la biblioteca para conectarse a bases de datos MySQL.
from dotenv import load_dotenv: Importa la función para cargar variables de entorno desde un archivo .env.
import os: Importa el módulo para interactuar con el sistema operativo.

2. Carga de Variables de Entorno
load_dotenv(): Carga las variables de entorno desde un archivo .env.
TOKEN_BOT = os.getenv('TOKEN'): Obtiene el token del bot de Telegram desde las variables de entorno.

3. Inicialización del Bot
bot = telebot.TeleBot(TOKEN_BOT): Crea una instancia del bot utilizando el token.

4. Conexión a la Base de Datos
db_host = os.getenv("DATABASE_HOST"): Obtiene la dirección del host de la base de datos.
db_port = os.getenv("DATABASE_PORT"): Obtiene el puerto de la base de datos.
db_name = os.getenv("DATABASE_NAME"): Obtiene el nombre de la base de datos.
db_user = os.getenv("DATABASE_USER"): Obtiene el nombre de usuario para la base de datos.
db_password = os.getenv("DATABASE_PASSWORD"): Obtiene la contraseña para el usuario de la base de datos.

5. Conexión Inicial a la Base de Datos
db = mysql.connector.connect(...): Establece la conexión a la base de datos utilizando las variables anteriores.
cursor = db.cursor(): Crea un cursor para ejecutar consultas SQL.
sql_query = "CREATE DATABASE IF NOT EXISTS {}".format(db_name): Prepara una consulta SQL para crear la base de datos si no existe.
cursor.execute(sql_query): Ejecuta la consulta para crear la base de datos.

6. Creación de Tabla
cursor.execute(...): Ejecuta una consulta SQL para crear una tabla llamada bot-telegram con varias columnas:
id: Identificador único autoincremental.
user_id: ID del usuario de Telegram.
nombre_usuario: Nombre de usuario de Telegram.
nombre_colaborador: Nombre del colaborador.
genero_colaborador: Género del colaborador.
correo_colaborador: Correo electrónico del colaborador.
imagen_url: URL de la imagen del colaborador.

7. Registro de Colaboradores
datos_colaborador = {}: Diccionario para almacenar temporalmente los datos de los colaboradores durante el registro.

8. Manejadores de Mensajes
@bot.message_handler(commands=['registrocolaborador']): Manejador para iniciar el registro de colaboradores.
@bot.message_handler(func=lambda message: datos_colaborador.get(message.chat.id, {}).get('state') == 'nombre_colaborador'): Manejador para recibir el nombre del colaborador.
@bot.callback_query_handler(func=lambda datos_genero_colaborador: datos_genero_colaborador.data.startswith("genero_")): Manejador para recibir el género del colaborador.
@bot.message_handler(func=lambda message: datos_colaborador.get(message.chat.id, {}).get('state') == 'correo_colaborador'): Manejador para recibir el correo electrónico.
@bot.message_handler(func=lambda message: datos_colaborador.get(message.chat.id, {}).get('state') == 'ubicacion_colaborador'): Manejador para recibir la ubicación.
@bot.message_handler(content_types=['location'], func=lambda message: datos_colaborador.get(message.chat.id, {}).get('state') == 'ubicacion_gps'): Manejador para recibir la ubicación GPS.
@bot.message_handler(content_types=['photo'], func=lambda message: datos_colaborador.get(message.chat.id, {}).get('state') == 'Foto_colaborador'): Manejador para recibir la foto del colaborador.

9. Almacenamiento de Datos en la Base de Datos
cursor.execute(...): Inserta los datos del colaborador en la tabla bot-telegram después de completar el registro.

10. Comandos del Bot
bot.set_my_commands([...]): Configura los comandos que el bot puede reconocer.
Este archivo configura un bot de Telegram que permite registrar colaboradores y almacenar su información en una base de datos MySQL.

## Instrucciones:

1. Requisitos Previos
Docker: Asegúrate de tener Docker y Docker Compose instalados en tu sistema. Puedes seguir las instrucciones de instalación en Docker y Docker Compose.
Git: Necesitarás Git para clonar el repositorio.

2. Clonar el Repositorio
Desde la terminal ejecute el siguiente comando:
git clone https://github.com/YulianHernandez/bot-telegram-poli.git
cd bot-telegram-poli

3. Configurar el Archivo .env
Crea un archivo llamado .env en la raíz del proyecto y añade las siguientes variables, reemplazando los valores según corresponda:

TOKEN=tu_token_de_bot_de_telegram
DATABASE_HOST=db
DATABASE_PORT=3306
DATABASE_NAME=telegram-bot
DATABASE_USER=myuser
DATABASE_PASSWORD=mypassword

4. Dockerfile 
Usa una imagen base de Python
FROM python:3.9

Establece el directorio de trabajo
WORKDIR /app

Copia los archivos necesarios
COPY . .

Instala las dependencias
RUN pip install -r requirements.txt

Comando para ejecutar el bot
CMD ["python", "main.py"]

5. Archivo docker-compose.yml
Asegurese que al momento de clonar el repositorio a la maquina local se debe verificar que el puerto 3306 no este ocupado. 

6. Construir y Ejecutar los Contenedores
Con todo configurado, ejecuta el siguiente comando para construir y ejecutar los contenedores:

docker-compose up --build
Esto descargará las imágenes necesarias, construirá el contenedor de la aplicación y levantará ambos contenedores (la base de datos y la aplicación).

7. Verificar el Funcionamiento
Base de Datos: Asegurese que el contenedor de MySQL esté funcionando correctamente. Puede verificar los registros de la base de datos con el comando:
docker-compose logs db

Bot de Telegram: Verifique que el bot esté funcionando correctamente y que esté escuchando los comandos. Puede enviar un mensaje al bot en Telegram y ver los registros de la aplicación con:
docker-compose logs app

8. Interacción con el Bot
Envía el comando /registrocolaborador al bot en Telegram para iniciar el proceso de registro de colaboradores. El bot debería guiarte a través de las preguntas y almacenar la información en la base de datos.

9. Detener los Contenedores
Para detener los contenedores, puedes usar:

docker-compose down

10. Notas Adicionales
Asegurese que el token del bot de Telegram sea válido y que el bot esté habilitado.
Revise la documentación de cada biblioteca utilizada en main.py para entender mejor su funcionamiento.
Puede modificar el código según sus necesidades y volver a construir los contenedores si realizas cambios.
Siguiendo estos pasos, deberia poder desarrollar y ejecutar el programa en contenedores de Docker sin problemas.

## Integración Continua con Jenkins

Este proyecto utiliza Jenkins para la integración continua. A continuación se presentan los pasos para configurar Jenkins y un pipeline para este proyecto.

### Requisitos Previos

- **Jenkins**: Asegúrate de tener Jenkins instalado y en funcionamiento. Puedes seguir las instrucciones de instalación en [Jenkins Documentation](https://www.jenkins.io/doc/).
- **Plugins**: Instala los siguientes plugins en Jenkins:
  - GitHub Integration Plugin
  - Pipeline
  - Git

### Configuración de Jenkins

1. **Crear un Nuevo Pipeline**:
   - En la página principal de Jenkins, haz clic en **"Nuevo elemento"**.
   - Escribe un nombre para tu proyecto (por ejemplo, `bot-telegram-poli-pipeline`) y selecciona **"Pipeline"**.
   - Haz clic en **"Aceptar"**.

2. **Configurar el Pipeline**:
   - En la configuración del proyecto, busca la sección **"Pipeline"**.
   - Cambia el **"Definition"** a **"Pipeline script"**.
   - Agrega el siguiente código en el campo de script:

     ```groovy
     pipeline {
         agent any
         stages {
             stage('Checkout') {
                 steps {
                     // Clonar el repositorio
                     git 'https://github.com/YulianHernandez/bot-telegram-poli.git'
                 }
             }
             stage('Build') {
                 steps {
                     echo 'Construyendo...'
                     // Aquí puedes agregar comandos para construir tu bot si es necesario
                 }
             }
             stage('Test') {
                 steps {
                     echo 'Ejecutando pruebas...'
                     // Aquí puedes agregar comandos para ejecutar pruebas si tienes
                 }
             }
             stage('Deploy') {
                 steps {
                     echo 'Desplegando...'
                     // Aquí puedes agregar el comando para desplegar tu bot
                 }
             }
         }
     }
     ```

3. **Configurar el Desencadenador de GitHub (Opcional)**:
   - Ve a **"Configuración"** en tu proyecto de Jenkins.
   - En **"Construcción desencadenada por"**, selecciona **"GitHub hook trigger for GITScm polling"**.

4. **Guardar y Ejecutar el Pipeline**:
   - Haz clic en **"Guardar"**.
   - En la página del proyecto, haz clic en **"Construir ahora"** para ejecutar el pipeline.

### Integración con GitHub

Para que Jenkins se active automáticamente con cambios en el repositorio, debes configurar un webhook en GitHub:

1. Ve al repositorio en GitHub.
2. Haz clic en **"Configuración"** > **"Webhooks"** > **"Agregar webhook"**.
3. En **"Payload URL"**, ingresa `http://<tu-ip>:8080/github-webhook/`.
4. Selecciona **"application/json"** como tipo de contenido.
5. En **"Which events would you like to trigger this webhook?"**, selecciona **"Just the push event."**
6. Haz clic en **"Agregar webhook"**.

### Verificación

- Después de configurar Jenkins y el webhook, cada vez que realices un push al repositorio, Jenkins ejecutará el pipeline automáticamente.
- Puedes verificar los registros de construcción en Jenkins para asegurarte de que todo funcione correctamente.

### Notas Adicionales

- Asegúrate de que el token del bot de Telegram sea válido y que el bot esté habilitado.
- Revisa la documentación de cada biblioteca utilizada en `main.py` para entender mejor su funcionamiento.
- Puedes modificar el código según tus necesidades y volver a construir los contenedores si realizas cambios.

