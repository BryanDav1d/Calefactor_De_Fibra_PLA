# Calefactor_De_Fibra_PLA
Este código es responsable de administrar un sistema que monitorea y regula la temperatura y humedad mediante un microcontrolador ESP32, sensores de temperatura y humedad DHT, y actuadores como un calefactor y ventiladores. Aquí hay una descripción más detallada de cada parte del código:

Configuración Inicial y Definiciones de Pines:
Importa las bibliotecas necesarias y define los pines utilizados para conectar los sensores DHT y los actuadores.

Variables y Objetos:
Define variables para las credenciales de la red WiFi, la dirección IP del servidor MQTT y los temas MQTT para la publicación y suscripción de datos. También crea objetos para los sensores DHT.

Funciones de Configuración de WiFi y MQTT:
setup_wifi(): Conecta el dispositivo a la red WiFi.
reconnect(): Reintenta la conexión al servidor MQTT si es necesario.

Función de Retorno de Llamada MQTT:
callback(): Se activa cuando se reciben mensajes MQTT. Procesa los mensajes y actualiza las variables correspondientes, como la temperatura deseada y el estado de los ventiladores.

Configuración Inicial del Dispositivo:
Inicializa la comunicación serial, configura la conexión WiFi, establece la configuración del servidor MQTT e inicia los sensores DHT.

Bucle Principal (loop()):
Verifica si el dispositivo está conectado al servidor MQTT y se reconecta si es necesario.
Lee los datos de temperatura y humedad de los sensores DHT y los publica en los temas MQTT correspondientes.
Controla el calefactor en función de la temperatura deseada y el margen definido.
Espera un intervalo de tiempo antes de volver a ejecutar el bucle.
En conclusión, con este código se configura y controla un sistema que monitorea y ajusta la temperatura y humedad utilizando un ESP32, sensores DHT22 y actuadores, y se comunica con un servidor MQTT para intercambiar datos de forma inalámbrica, haciendo lo que se conoce como IOT.
