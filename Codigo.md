#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>
#define DHTTYPE DHT11 //Estabelecendo o tipo do sensor
#define DHTPIN 4 //Pino de dados do ESP (GPIO4)

//Biblioteca do ESP8266
#include <ESP8266WiFi.h>
const char* ssid = "Y-WIFI"; //Nome da rede a se conectar
const char* password = "33127174"; //Senha da rede

//API KEY do ThingSpeak
String apikey = "E64FEYAHHUX0YS55";
const char* server = "api.thingspeak.com";

DHT dht(DHTPIN, DHTTYPE);
WiFiClient client;

void setup(){
  Serial.begin(9600); //Configuracao UART
  WiFi.begin(ssid, password); //Inicia Wifi


//Aguardar a conexao com o roteador
  while(WiFi.status() != WL_CONNECTED){
    delay(500);
    Serial.print("...");
}
dht.begin(); //Inicia sensor

//Logs da conexao de rede na porta serial
  Serial.println("");
  Serial.print("Conectado a rede: ");
  Serial.println(ssid);
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());

}
void loop(){
   float umidade = dht.readHumidity(); //Leitura da umidade
   float temperatura = dht.readTemperature();//Leitura da temperatura
    //Verificacao de erro de leitura
  if(isnan(umidade)|| isnan(temperatura)){
        Serial.println("Erro de leitura do sensor");
  return;
  }
//Conexao TCP para envio de dados
if (client.connect(server,80)){
  String postStr = apikey;
         postStr +="&amp;field1=";
         postStr += String(temperatura);
         postStr += "&amp;field2=";
         postStr += String(umidade);
         postStr += "\r\n\r\n";
         
     client.print("POST /update HTTP/1.1\n");
     client.print("Host: api.thingspeak.com\n");
     client.print("Connection: close\n");
     client.print("X-THINGSPEAKAPIKEY: "+apikey+"\n");
     client.print("Content-Type: application/x-www-form-urlencoded\n");
     client.print("Content-Length: ");
     client.print(postStr.length());
     client.print("\n\n");
     client.print(postStr);
}
client.stop();
//Check de dados com a porta serial
    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.print(" C, Umidade: ");
    Serial.println(umidade);
   
}
