#define SW_VERSION " ThinkSpeak.com" // SW version will appears at innitial LCD Display
#include <ESP8266WiFi.h>
#include<dht.h>
dht DHT;

#define DHT11_PIN D1
WiFiClient client;
WiFiServer server(80);

const char* MY_SSID = "WiFiAAA";
const char* MY_PWD = "Januka@333";
const char* TS_SERVER = "api.thingspeak.com";
String TS_API_KEY = "38Y0F9GXRTFVK4FA";

void connectWifi()
{
  Serial.print("Connecting to " + *MY_SSID);
  WiFi.begin(MY_SSID, MY_PWD);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(1000);
    Serial.print("+");
  }
  Serial.println("");
  Serial.println("WiFi Connected");
  Serial.println("");
  server.begin();
  Serial.println("Server started");
  Serial.print("Use this URL to connect: ");
  Serial.print("http://");
  Serial.print(WiFi.localIP());
  Serial.println("/");
}

void sendDataTS(void)
{
  int chk = DHT.read11(DHT11_PIN);
  Serial.print("Humidity: " );
  Serial.print(DHT.humidity, 1);
  Serial.print(" Temparature: ");
  Serial.println(DHT.temperature, 1);
  delay(2000);
   
  if (client.connect(TS_SERVER, 80))
  {
    Serial.print(" sending to thingspeak ");
    String postStr = TS_API_KEY;
    postStr += "&field1=";
    postStr += String(DHT.humidity, 1);
    postStr += "&field2=";
    postStr += String(DHT.temperature, 1);
    postStr += "\r\n\r\n";

    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + TS_API_KEY + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(postStr.length());
    client.print("\n\n");
    client.print(postStr);
    delay(100);
    Serial.print(" done with thingspeak ");
  }
  client.stop();
}

void setup()
{
  Serial.begin(9600);
  delay(10);
  connectWifi();
  }

void loop()
{
  sendDataTS();
  delay(5000);
}
