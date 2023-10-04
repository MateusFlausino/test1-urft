#include <wl_definitions.h>
#include "ESP8266WiFi.h"
#include <PubSubClient.h>

//Parametros de conexão
const char* ssid = "GRVA"; // REDE
const char* password = "grva-feelt0205"; // SENHA


// MQTT Broker
const char *mqtt_broker = "07722a33c9d749018ea6cea380e5cb1f.s2.eu.hivemq.cloud";  //Host do broket
const char *topic = "tps_test";            //Topico a ser subscrito e publicado
const char *mqtt_username = "";         //Usuario
const char *mqtt_password = "";         //Senha
const int mqtt_port = 8883;             //Porta

//Variáveis
bool mqttStatus = 0;

//Objetos
WiFiClient espClient;
PubSubClient client(espClient);

//Prototipos
bool connectMQTT();
void callback(char *topic, byte * payload, unsigned int length);

void setup(void)
{
  Serial.begin(9600);
  Serial2.begin(9600, SERIAL_8N1);
  // Conectar
  WiFi.begin(ssid, password);

  //Aguardando conexão
  Serial.println();
  Serial.print("Conectando");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  //Envia IP através da UART
  Serial.println(WiFi.localIP());

  mqttStatus =  connectMQTT();

}

void loop() {
 static long long pooling  = 0;
  if ( mqttStatus){
    
    client.loop();    

    if (millis() > pooling +1000){
      pooling = millis();
      client.publish(topic, "{teste123,113007042022}");
    }
       
  }
}



bool connectMQTT() {
  byte tentativa = 0;
  client.setServer(mqtt_broker, mqtt_port);
  client.setCallback(callback);

  do {
    String client_id = "BOBSIEN-";
    client_id += String(WiFi.macAddress());

    if (client.connect(client_id.c_str(), mqtt_username, mqtt_password)) {
      Serial.println("Exito na conexão:");
      Serial.printf("Cliente %s conectado ao broker\n", client_id.c_str());
    } else {
      Serial.print("Falha ao conectar: ");
      Serial.print(client.state());
      Serial.println();
      Serial.print("Tentativa: ");
      Serial.println(tentativa);
      delay(2000);
    }
    tentativa++;
  } while (!client.connected() && tentativa < 5);

  if (tentativa < 5) {
    // publish and subscribe   
    client.publish(topic, Serial2.readStrig()); 
    client.subscribe(topic);
    return 1;
  } else {
    Serial.println("Não conectado");    
    return 0;
  }
}

void callback(char *topic, byte * payload, unsigned int length) {
  Serial.print("Message arrived in topic: ");
  Serial.println(topic);
  Serial.print("Message:");
  for (int i = 0; i < length; i++) {
    Serial.print((char) payload[i]);
  }
  Serial.println();
  Serial.println("-----------------------");
}
