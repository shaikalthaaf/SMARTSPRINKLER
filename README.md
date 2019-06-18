# SMART SPRINKLER
#include <DHT.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include<Servo.h>

void callback(char* subtopic, byte* payload, unsigned int payloadLength);
const char* ssid = "k6power";
const char* password = "nitish88812142799";

#define ORG "28fhza"
#define DEVICE_TYPE "nodemuc"
#define DEVICE_ID "2799"
#define TOKEN "123456789"

#define DHTPIN D2    // what pin we're connected to
#define DHTTYPE DHT11   // define type of sensor DHT 11
DHT dht (DHTPIN, DHTTYPE);

 # define SERVOPIN D4

Servo myservo;

int sensorPin = A0;  
int sensorValue = 0;

String command;

char server[] = ORG ".messaging.internetofthings.ibmcloud.com";
char subtopic[] = "iot-2/cmd/home/fmt/String";
char topic[] = "iot-2/evt/Data/fmt/json";
char authMethod[] = "use-token-auth";
char token[] = TOKEN;
char clientId[] = "d:" ORG ":" DEVICE_TYPE ":" DEVICE_ID;


WiFiClient wifiClient;
PubSubClient client(server, 1883, callback, wifiClient);
void setup() {
 Serial.begin(115200);
 Serial.println();
 dht.begin();
  myservo.detach();

  pinMode(DHTPIN,OUTPUT);
wifiConnect();
 mqttConnect();
}

void loop() {

float h = dht.readHumidity();
float t = dht.readTemperature();

if (isnan(h) || isnan(t))
{
Serial.println("Failed to read from DHT sensor!");
delay(1000);
return;
}

 sensorValue = analogRead(sensorPin);
  int percentValue = map(sensorValue, 1023, 465, 0, 100);
  if(percentValue <30)
  {
    myservo.attach(SERVOPIN);
    for(int i=0;i<180;i++)
    {
       myservo.write(i);
       delay(50);
     }
  }
     else
     {
      myservo.detach();
      }
    
    
PublishData(t,h,percentValue);
delay(1000);
 
  if (!client.loop()) {
    mqttConnect();
  }
delay(100);
}

void mqttConnect() {
  if (!client.connected()) {
    Serial.print("Reconnecting MQTT client to "); 
    Serial.println(server);
    while (!client.connect(clientId, authMethod, token)) {
      Serial.print(".");
      delay(500);
    }
    
    initManagedDevice();
    Serial.println();
  }
}
void initManagedDevice() {
  if (client.subscribe(subtopic)) {
    Serial.println("subscribe to cmd OK");
  } else {
    Serial.println("subscribe to cmd FAILED");
  }
}

void callback(char* subtopic, byte* payload, unsigned int payloadLength) {
  Serial.print("callback invoked for topic: "); 
  Serial.println(subtopic);

  for (int i = 0; i < payloadLength; i++) {
    command += (char)payload[i];
  }
  
Serial.println(command);


if(command == "sprinkleron")
{
  
  myservo.attach(SERVOPIN);
  for(int i=0;i<180;i++)
{
  myservo.write(i);
delay(50);
}
 for(int i=180;i<0;i--)
{
  myservo.write(i);
delay(50);
}
}
else if(command == "sprinkleroff"){
 
  myservo.detach();
 
}
command ="";

}

void PublishData(float temp, float humid,int moist){
 if (!!!client.connected()) {
 Serial.print("Reconnecting client to ");
 Serial.println(server);
 while (!!!client.connect(clientId, authMethod, token)) {
 Serial.print(".");
 delay(500);
 }
 Serial.println();
 }
  String payload = "{\"d\":{\"temperature\":";
  payload += temp;
  payload+="," "\"humidity\":";
  payload += humid;
  payload+="," "\"moisture\":";
  payload += moist;
  payload += "}}";
 Serial.print("Sending payload: ");
 Serial.println(payload);
  
 if (client.publish(topic, (char*) payload.c_str())) {
 Serial.println("Publish ok");
 } else {
 Serial.println("Publish failed");
 }
}
void wifiConnect() {
  Serial.print("Connecting to "); Serial.print(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.print("nWiFi connected, IP address: "); Serial.println(WiFi.localIP());
}
