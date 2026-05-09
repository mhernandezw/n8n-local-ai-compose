# n8n-local-ai-compose

# Despliegue de n8n con IA Local (Ollama + Open-WebUI)

Este repositorio documenta el proceso de despliegue y configuración de un servidor de **n8n** (basado en su documentación oficial), al que hemos añadido **Ollama** y **Open-WebUI** para poder ejecutar y gestionar modelos de Inteligencia Artificial de forma totalmente local y privada.

---

## 1. Preparación del Entorno

En primer lugar, crearemos la carpeta principal del proyecto y configuraremos las variables de entorno:

1. Crea una carpeta llamada `n8n-compose` y accede a ella.
2. Dentro de esta carpeta, crea un archivo llamado `.env` con el siguiente contenido. Asegúrate de modificar los valores según tu dominio y correo electrónico:

```env
# DOMAIN_NAME and SUBDOMAIN together determine where n8n will be reachable from
# The top level domain to serve from
DOMAIN_NAME=example.com

# The subdomain to serve from
SUBDOMAIN=n8n

# The above example serve n8n at: [https://n8n.example.com](https://n8n.example.com)

# Optional timezone to set which gets used by Cron and other scheduling nodes
# New York is the default value if not set
GENERIC_TIMEZONE=Europe/Madrid

# The email address to use for the TLS/SSL certificate creation
SSL_EMAIL=user@example.com
```
### Carpetas compartidas: Dentro de la carpeta n8n-compose, crea un directorio llamado local-files. Esta carpeta servirá para compartir ficheros entre la instancia de n8n y el sistema anfitrión de forma sencilla.

## 2. Configuración de DNS
Para que n8n sea accesible a través del subdominio indicado en el archivo .env (ej. n8n.example.com), necesitas resolver el nombre de dominio:

Si tienes un servidor DNS / Dominio propio: Crea un registro tipo A apuntando tu subdominio (ej. n8n) a la dirección IP pública o local del servidor donde estás desplegando los contenedores.

Si no tienes servidor DNS: Tendrás que modificar el archivo hosts de tu sistema operativo cliente (Windows, Linux o macOS), creando una entrada que asocie la IP del servidor con el dominio completo (ej. 192.168.1.100 n8n.example.com).


## 3. Archivo Docker Compose
Dentro de la carpeta n8n-compose, crea (o descarga) el archivo compose.yaml

### Explicación de los Contenedores
Traefik y n8n: El contenedor de n8n es el núcleo de nuestra automatización. Para exponerlo de forma segura, utilizamos traefik, un proxy inverso que gestiona automáticamente el enrutamiento y la generación de certificados TLS/SSL (HTTPS) mediante Let's Encrypt. Además, mapeamos los directorios locales (local-files) como volúmenes para interactuar con archivos desde los flujos de n8n.

Ollama: Es el motor que permite ejecutar modelos de lenguaje de gran tamaño (LLMs) localmente. Hemos mapeado el puerto 11434 y configurado un volumen persistente (ollama_data) para no perder los modelos descargados al reiniciar.

Aceleración por GPU: Si dispones de una tarjeta gráfica NVIDIA y tienes instalado el NVIDIA Container Toolkit, puedes descomentar las líneas bajo deploy para que la IA se ejecute de forma mucho más rápida. Existen más formas de configurar los recursos según la documentación oficial de Docker sobre despliegues: https://docs.docker.com/reference/compose-file/deploy/

Este es el listado de hardware compatible con Ollama para usar nuestra GPU: https://docs.ollama.com/gpu
https://developer.nvidia.com/cuda/gpus

Esta es la guia de instalación del kit de herramientas de Nvidia para utilizar la GPU en contenedores Docker o Kubernetes: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

Open-WebUI: Proporciona una interfaz gráfica estilo ChatGPT para interactuar con los modelos descargados en Ollama. El contenedor expone el puerto 3000 (que internamente redirige al 8080 del contenedor) y utiliza la variable de entorno OLLAMA_BASE_URL=http://ollama:11434 para conectarse directamente al contenedor de Ollama de forma automática.

Para levantar todos los servicios, ejecuta en la terminal desde la carpeta n8n-compose:
docker compose up -d


## 4. Configuración de IA Local (Open-WebUI y Ollama)
Acceso a Open-WebUI
Para acceder a la interfaz de Open-WebUI, deberás utilizar la misma URL/IP de tu servidor n8n, pero apuntando al puerto 3000.

Ejemplo de acceso: Si tu n8n está en https://n8n.example.com (o usando tu IP local http://192.168.1.100), accederás a Open-WebUI entrando a http://n8n.example.com:3000 (o http://192.168.1.100:3000).

Registro y Descarga de Modelos
La primera vez que accedas a Open-WebUI, se te pedirá crear una cuenta. El primer usuario que se registre se convertirá automáticamente en el administrador de la aplicación.

Aunque el contenedor ya está preparado para apuntar a Ollama (http://ollama:11434), es recomendable ir a los Ajustes (Settings) > Conexiones para verificar que el estado de conexión con Ollama es correcto.

Antes de poder usar la IA en n8n, necesitas descargar al menos un modelo local.

Desde el panel de administración o la sección de configuración de modelos de Open-WebUI, busca la opción para descargar un modelo (Pull a model). Introduce el nombre del modelo que desees, por ejemplo gemma4:e2b (u otros como llama3, phi3), y haz clic en descargar.

Para consultar el listado de modelos disponibles consultar la siguiente URL: https://ollama.com/library 


<img width="958" height="404" alt="image" src="https://github.com/user-attachments/assets/0dc52f87-fd71-4e50-b33e-b5d4161fc556" />



## 5. Integración de Ollama en n8n
Una vez que tengas tu modelo descargado y funcionando, integrarlo en tus flujos de automatización es muy sencillo:

Entra a tu instancia de n8n.

Ve a la sección de Credentials (Credenciales) y añade una nueva credencial buscando Ollama API.

En el campo de configuración de la URL, debes indicar la ruta interna del contenedor de Docker: http://ollama:11434.

No es necesario indicar ningún Token, ya que la ejecución es en local y no requiere autenticación en este entorno. Guarda la credencial.

<img width="1198" height="748" alt="image" src="https://github.com/user-attachments/assets/efa6869e-dcbf-4f5c-b2fd-2398b4ba6ef0" />


Al crear un flujo (Workflow), podrás añadir un nodo de AI Agent. En la configuración del agente, selecciona Ollama Chat Model como tu modelo de IA. El nodo se conectará a la API y te permitirá seleccionar directamente en un desplegable los modelos locales que hayas descargado previamente (ej. gemma4:e2b).

<img width="767" height="395" alt="image" src="https://github.com/user-attachments/assets/9d98413d-0104-4ded-b107-8eac252f8d72" />

<img width="1241" height="422" alt="image" src="https://github.com/user-attachments/assets/a090913b-c118-4b30-9f0f-14863d665c86" />



## 6. Gestión de Memoria y Chats
Es importante entender cómo manejan el historial de conversaciones ambas plataformas:

En Open-WebUI: Todo tu historial de chat se guarda automáticamente y de forma persistente en su base de datos local, por lo que podrás retomar tus conversaciones cuando quieras.

En n8n: Si configuras tu nodo AI Agent utilizando una memoria de tipo "Window Buffer Memory" o "Simple Memory", el contexto de la conversación se almacenará temporalmente en la RAM. Esto significa que cuando el flujo termine de ejecutarse, el chat se borrará y no se guardará el historial.

Persistencia en n8n: Si deseas que el agente de n8n recuerde interacciones pasadas en diferentes ejecuciones, tendrás que utilizar nodos de memoria persistente. Algunos ejemplos disponibles en n8n son:

Postgres Chat Memory (requiere una base de datos PostgreSQL)

Redis Chat Memory

Zep Chat Memory

Motorhead o Xata
