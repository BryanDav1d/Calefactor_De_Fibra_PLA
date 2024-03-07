//Autor
//Bryan Noboa

#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

// Definiciones de pines para los sensores DHT y los actuadores (calefactor y ventiladores)
#define DHTPIN1 15          // Pin de datos del primer sensor DHT
#define DHTPIN2 5           // Pin de datos del segundo sensor DHT
#define DHTTYPE DHT22       // Tipo de sensor DHT utilizado
#define pinCalefactor 12    // Pin de control del calefactor
#define pinVentilador1 14   // Pin de control del primer ventilador
//#define pinVentilador2 25   // Pin de control del segundo ventilador (cambiado)

// Credenciales de la red WiFi y configuración del servidor MQTT
const char* ssid = "BRYAN_NOBOA";                  // Nombre de la red WiFi
const char* password = "123123123"; // Contraseña de la red WiFi
const char* mqtt_server = "192.168.137.124";     // Dirección IP del servidor MQTT

// Temas MQTT para la publicación y suscripción de datos
const char* topic_temp_interior = "TemperaturaInterior";  // Temperatura interior
const char* topic_hum_interior = "HumedadInterior";      // Humedad interior
const char* topic_temp_exterior = "TemperaturaExterior";  // Temperatura exterior
const char* topic_hum_exterior = "HumedadExterior";      // Humedad exterior
const char* topic_temp_deseada = "temperatura_deseada";  // Temperatura deseada
const char* topic_ventilador = "ventilador";             // Control del ventilador

DHT dht1(DHTPIN1, DHTTYPE); // Objeto para el primer sensor DHT
DHT dht2(DHTPIN2, DHTTYPE); // Objeto para el segundo sensor DHT

WiFiClient espClient;       // Cliente WiFi para la conexión
PubSubClient client(espClient); // Cliente MQTT para la comunicación

float temperatura_deseada = 0; // Variable para almacenar la temperatura deseada
float margen = 2.0;            // Margen de temperatura para encender/apagar el calefactor
bool ventiladoresActivos = false; // Variable para indicar si los ventiladores están activos o no

// Función para configurar la conexión WiFi
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando a ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("Conectado a la red WiFi");
  Serial.println("Dirección IP: ");
  Serial.println(WiFi.localIP());
}

// Función para reconectar al servidor MQTT
void reconnect() {
  while (!client.connected()) {
    Serial.print("Intentando conexión MQTT...");
    if (client.connect("ESP32Client")) {
      Serial.println("Conectado");
      client.subscribe(topic_temp_deseada); // Suscripción al tema MQTT para la temperatura deseada
      client.subscribe(topic_ventilador);   // Suscripción al tema MQTT para el control del ventilador
    } else {
      Serial.print("falló, rc=");
      Serial.print(client.state());
      Serial.println(" intentando de nuevo en 5 segundos");
      delay(5000);
    }
  }
}

// Función de retorno de llamada para procesar mensajes MQTT entrantes
void callback(char* topic, byte* payload, unsigned int length) {
  if (strcmp(topic, topic_temp_deseada) == 0) {
    // Si el mensaje recibido es sobre la temperatura deseada, actualizar la variable correspondiente
    char str_payload[length + 1];
    for (int i = 0; i < length; i++) {
      str_payload[i] = (char)payload[i];
    }
    str_payload[length] = '\0';
    temperatura_deseada = atof(str_payload); // Almacena el valor de la temperatura deseada como un flotante
  } else if (strcmp(topic, topic_ventilador) == 0) {
    // Si el mensaje recibido es sobre el control del ventilador
    if (payload[0] == '1') {
      // Si el valor del payload es '1', encender los ventiladores
      ventiladoresActivos = true;
      digitalWrite(pinVentilador1, HIGH); // Enciende el primer ventilador
      //digitalWrite(pinVentilador2, HIGH); // Enciende el segundo ventilador
      Serial.println("Ventiladores encendidos"); // Imprime un mensaje indicando que los ventiladores están encendidos
    } else if (payload[0] == '0') {
      // Si el valor del payload es '0', apagar los ventiladores
      ventiladoresActivos = false;
      digitalWrite(pinVentilador1, LOW); // Apaga el primer ventilador
      //digitalWrite(pinVentilador2, LOW); // Apaga el segundo ventilador
      Serial.println("Ventiladores apagados"); // Imprime un mensaje indicando que los ventiladores están apagados
    }
  }
}

// Configuración inicial del dispositivo
void setup() {
  Serial.begin(115200); // Inicializa la comunicación serial
  setup_wifi();         // Configura la conexión WiFi
  client.setServer(mqtt_server, 1883); // Configura el servidor MQTT
  client.setCallback(callback); // Establece la función de retorno de llamada MQTT
  dht1.begin(); // Inicializa el primer sensor DHT
  dht2.begin(); // Inicializa el segundo sensor DHT
  pinMode(pinCalefactor, OUTPUT); // Configura el pin del calefactor como salida
  pinMode(pinVentilador1, OUTPUT); // Configura el pin del primer ventilador como salida
  //pinMode(pinVentilador2, OUTPUT); // Configura el pin del segundo ventilador como salida
}

// Función principal de ejecución del bucle
void loop() {
  if (!client.connected()) {
    // Si no está conectado al servidor MQTT, intenta reconectar
    reconnect();
  }
  client.loop(); // Mantén la conexión MQTT activa

  // Lectura de la temperatura y la humedad de los sensores DHT
  float h1 = dht1.readHumidity(); // Lee la humedad interior
  float t1 = dht1.readTemperature(); // Lee la temperatura interior
  if (isnan(h1) || isnan(t1)) {
    // Si hay un error al leer los datos del sensor DHT1, muestra un mensaje de error y espera un segundo
    Serial.println("Error al leer el sensor DHT22 interior!");
    delay(1000);
    return;
  }

  // Publica los datos de temperatura y humedad interior en los temas MQTT correspondientes
  client.publish(topic_temp_interior, String(t1).c_str());
  client.publish(topic_hum_interior, String(h1).c_str());

  // Imprime los datos de temperatura y humedad interior en la consola serial
  Serial.print("Temperatura interior: ");
  Serial.print(t1);
  Serial.print(" °C - Humedad interior: ");
  Serial.print(h1);
  Serial.println(" %");

  // Lectura de la temperatura y la humedad exterior de los sensores DHT
  float h2 = dht2.readHumidity(); // Lee la humedad exterior
  float t2 = dht2.readTemperature(); // Lee la temperatura exterior
  if (isnan(h2) || isnan(t2)) {
    // Si hay un error al leer los datos del sensor DHT2, muestra un mensaje de error y espera un segundo
    Serial.println("Error al leer el sensor DHT22 exterior!");
    delay(1000);
    return;
  }

  // Publica los datos de temperatura y humedad exterior en los temas MQTT correspondientes
  client.publish(topic_temp_exterior, String(t2).c_str());
  client.publish(topic_hum_exterior, String(h2).c_str());

  // Imprime los datos de temperatura y humedad exterior en la consola serial
  Serial.print("Temperatura exterior: ");
  Serial.print(t2);
  Serial.print(" °C - Humedad exterior: ");
  Serial.print(h2);
  Serial.println(" %");

  // Control del calefactor en función de la temperatura deseada y el margen definido
  if (t1 < temperatura_deseada + margen) {
    digitalWrite(pinCalefactor, LOW); // Apaga el calefactor
    Serial.println("Calefactor encendido");
  } else if (t1 > temperatura_deseada - margen) {
    digitalWrite(pinCalefactor, HIGH); // Enciende el calefactor
    Serial.println("Calefactor apagado");
  }
  
  delay(2000); // Espera 2 segundos antes de volver a ejecutar el bucle
}