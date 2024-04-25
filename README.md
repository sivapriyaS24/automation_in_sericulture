#define BLYNK_TEMPLATE_ID "TMPL3rgdEQ7fd"
#define BLYNK_TEMPLATE_NAME "SERICULTURE UNIT"
#define BLYNK_AUTH_TOKEN "p-ayiLLLZYBf1XiRYcV-08BvtZCVuPgn"

#define BLYNK_PRINT Serial
#define DHTTYPE DHT11
#define DHTPIN D4 
#include <ESP8266WiFi.h> 
#include <BlynkSimpleEsp8266.h>
#include "DHT.h"
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

int led = 14; // Connected to D5 pin 
int buzzer = 12; //Connected to D6 pin 
int MQ135 = A0; //Connected to A0 pin 
int DHTPIN = 2;//connected to D4 pin 
int ldr=10;//connected to SDD3 pin
int ldr_led= 13;// connected to D7 pin
int Relay_PIN = 9; // connected to D8 pin

char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "Galaxy M32B468";  // type your wifi name
char pass[] = "pjothi03";  // type your wifi password
String apiKey = "750AOM3NFNTFO747"; 
const char* server = "api.thingspeak.com";

float humDHT = 0;
float tempDHT = 0;
int Val=0;
DHT dht(DHTPIN, DHTTYPE);
WiFiClient client;

void setup()
{
  pinMode(led, OUTPUT);
  pinMode(buzzer, OUTPUT);;
  pinMode(MQ135, INPUT);
  pinMode(DHTPIN,INPUT);
  pinMode(ldr, INPUT);
  pinMode(ldr_led, OUTPUT);
  pinMode(Relay_PIN, OUTPUT);
  digitalWrite(Relay_PIN, LOW);
  digitalWrite(led, LOW);
  digitalWrite(buzzer, LOW);
  // Debug console
  Serial.begin(115200);
  Blynk.begin(auth, ssid, pass);
  dht.begin();
  lcd.init();
  lcd.backlight();
  Serial.println("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, pass);
 
      while (WiFi.status() != WL_CONNECTED) 
     {
            delay(500);
            Serial.print(".");
     }
      Serial.println("");
      Serial.println("WiFi connected");
}

void loop() 
{
  int MQ135_sensor_read = analogRead(MQ135); //reading the sensor data on A0 pin
  Serial.print("Gas sensor Status:");
  Serial.println(MQ135_sensor_read);
 lcd.setCursor(0, 0);
  lcd.print("Gas:");
  lcd.print(MQ135_sensor_read);
  if (MQ135_sensor_read > 100)
  {
    digitalWrite(led, HIGH);
    digitalWrite(buzzer, HIGH);
    }
  else
    {
     digitalWrite(led, LOW);
     digitalWrite(buzzer, LOW);
    }
  float humDHT = dht.readHumidity();
  float tempDHT = dht.readTemperature();
  if (isnan(humDHT) || isnan(tempDHT))
  {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
    if (client.connect(server,80))   //   "184.106.153.149" or api.thingspeak.com
                      {  
                            
                             String postStr = apiKey;
                             postStr +="&field1=";
                             postStr += String(tempDHT);
                             postStr +="&field2=";
                             postStr += String(humDHT);
                             postStr +="&field3=";
                             postStr += String(MQ135_sensor_read);
                             postStr += "\r\n\r\n";
 
                             client.print("POST /update HTTP/1.1\n");
                             client.print("Host: api.thingspeak.com\n");
                             client.print("Connection: close\n");
                             client.print("X-THINGSPEAKAPIKEY: "+apiKey+"\n");
                             client.print("Content-Type: application/x-www-form-urlencoded\n");
                             client.print("Content-Length: ");
                             client.print(postStr.length());
                             client.print("\n\n");
                             client.print(postStr); 
                         }
  client.stop();
  Serial.print(F("Temperature: "));
  Serial.print(tempDHT);
  Serial.print(F("°C "));
  Serial.println();
  Serial.print(F("Humidity: "));
  Serial.print(humDHT);
  Serial.print(F("%"));
  Serial.println();
  lcd.setCursor(9,0);
  lcd.print("T:");
  lcd.print(tempDHT);
  lcd.setCursor(0,1);
  lcd.print("H:");
  lcd.print(humDHT); 
 int value = digitalRead(ldr);
  if(value == 1){
    digitalWrite(ldr_led, HIGH);
  }
  else{
    digitalWrite(ldr_led, LOW);
  }
  if((tempDHT>30.0)||(MQ135_sensor_read>50))
  {
    digitalWrite(Relay_PIN, HIGH);
  }
  else 
  {
    digitalWrite(Relay_PIN, LOW);
  }
  Serial.println(value); 
  Blynk.virtualWrite(V0, tempDHT);
  Blynk.virtualWrite(V1, humDHT);
  Blynk.virtualWrite(V2,MQ135_sensor_read); 
  Blynk.virtualWrite(V3,ldr); 
  Blynk.virtualWrite(V4,Relay_PIN);
      delay(500);
      Blynk.run();
}
