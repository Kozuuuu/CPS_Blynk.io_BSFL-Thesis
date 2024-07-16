# Thesis BSFL
 
#define BLYNK_TEMPLATE_ID "TMPL6_QPmj3Mv"
#define BLYNK_TEMPLATE_NAME "BSFL"
#define BLYNK_AUTH_TOKEN "kjLF5Nd9hOTRZGay8x-CmJ_PrUB7jyBo"
#define BLYNK_PRINT Serial

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <WiFiClient.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>
#include <ESP32Servo.h>

#define RELAY_FAN 2
#define RELAY_LED 26
#define RELAY_HEATER 27

#define DHTPIN 25
#define DHTTYPE    DHT11

#define VIRTUAL_PIN_Temperature V0
#define VIRTUAL_PIN_Humidity V1
#define VIRTUAL_PIN_Substrate_Sensor_1 V2
#define VIRTUAL_PIN_Substrate_Sensor_2 V3
#define VIRTUAL_PIN_Substrate_Sensor_3 V4
#define VIRTUAL_PIN_Sunlight_Sensor V5
#define VIRTUAL_PIN_Fan V6
#define VIRTUAL_PIN_Heater V7
#define VIRTUAL_PIN_Automated_Ventilation V8
#define VIRTUAL_PIN_Automated_Feeder_1 V9
#define VIRTUAL_PIN_Automated_Feeder_2 V10
#define VIRTUAL_PIN_Automated_Feeder_3 V11
// #define VIRTUAL_PIN_Light_Switch V12
#define VIRTUAL_PIN_Automated_ManualControl V20

int switchState;
int switchFeeder_1;
int switchFeeder_2;
int switchFeeder_3;
int switchFan;
int switchHeater;
int switchVentilation;
// int Lightswitch;

char auth[] = BLYNK_AUTH_TOKEN ; //Auth Token
char ssid[] = "POCO F5"; //name
char pass[] = "qwertyuiop"; //password 

BlynkTimer timer;

BLYNK_CONNECTED() {
  Blynk.syncVirtual(VIRTUAL_PIN_Temperature);
  Blynk.syncVirtual(VIRTUAL_PIN_Humidity);
  Blynk.syncVirtual(VIRTUAL_PIN_Substrate_Sensor_1);
  Blynk.syncVirtual(VIRTUAL_PIN_Substrate_Sensor_2);
  Blynk.syncVirtual(VIRTUAL_PIN_Substrate_Sensor_3);
  Blynk.syncVirtual(VIRTUAL_PIN_Sunlight_Sensor);
  Blynk.syncVirtual(VIRTUAL_PIN_Fan);
  Blynk.syncVirtual(VIRTUAL_PIN_Heater);
  Blynk.syncVirtual(VIRTUAL_PIN_Automated_Ventilation);
  Blynk.syncVirtual(VIRTUAL_PIN_Automated_Feeder_1);
  Blynk.syncVirtual(VIRTUAL_PIN_Automated_Feeder_2);
  Blynk.syncVirtual(VIRTUAL_PIN_Automated_Feeder_3);
  Blynk.syncVirtual(VIRTUAL_PIN_Automated_ManualControl);
  // Blynk.syncVirtual(VIRTUAL_PIN_Light_Switch);
}

static const int servoPin1 = 14;
static const int servoPin2 = 12;
static const int servoPin3 = 21;//
static const int servoPin4 = 22;

Servo servo1;
Servo servo2;
Servo servo3;
Servo servo4;

static const int servoPin5 = 23;
static const int servoPin6 = 15;
static const int servoPin7 = 18;

Servo servo5;
Servo servo6;
Servo servo7;

const int trigPin1 = 16;
const int echoPin1 = 4;
const int trigPin2 = 17;
const int echoPin2 = 5;
const int trigPin3 = 33;
const int echoPin3 = 32;

long duration1, duration2, duration3;
int distance1, distance2, distance3;

DHT_Unified dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);

  dht.begin(); 

  pinMode(trigPin1, OUTPUT); 
  pinMode(echoPin1, INPUT); 
  pinMode(trigPin2, OUTPUT); 
  pinMode(echoPin2, INPUT); 
  pinMode(trigPin3, OUTPUT); 
  pinMode(echoPin3, INPUT);

  servo1.setPeriodHertz(50); 
  servo2.setPeriodHertz(50); 
  servo3.setPeriodHertz(50); 
  servo4.setPeriodHertz(50); 

  servo5.setPeriodHertz(50);  
  servo6.setPeriodHertz(50); 
  servo7.setPeriodHertz(50); 

  servo1.attach(servoPin1, 500, 2400);  
  servo2.attach(servoPin2, 500, 2400); 
  servo3.attach(servoPin3, 500, 2400);  
  servo4.attach(servoPin4, 500, 2400);

  servo5.attach(servoPin5, 500, 2400);  
  servo6.attach(servoPin6, 500, 2400);
  servo7.attach(servoPin7, 500, 2400);  

  servo1.write(0);
  servo2.write(0);
  servo3.write(0);
  servo4.write(0);

  servo5.write(0);
  servo6.write(0);
  servo7.write(0);

  pinMode(RELAY_FAN, OUTPUT);
  pinMode(RELAY_LED, OUTPUT);
  pinMode(RELAY_HEATER, OUTPUT);

  Blynk.begin(auth, ssid, pass); //memulai Blynk

  WiFi.begin(ssid, pass);
  timer.setInterval(2000L, checkBlynkStatus); // check if Blynk server is connected every 2 seconds
  Blynk.config(auth);
  delay(5000);
}

void checkBlynkStatus() { // called every 3 seconds by SimpleTimer
  bool isconnected = Blynk.connected();
  if (isconnected == false) {
    Serial.println("Blynk Not Connected");
  }
  if (isconnected == true) {
    Serial.println("Blynk Connected");
  }
}

BLYNK_WRITE(VIRTUAL_PIN_Automated_ManualControl){
  switchState = param.asInt();

  if (switchState == 1) {
    Serial.println("Manual Mode");
  } 
  if (switchState == 0) {
    Serial.println("Auto Mode");
  }
}

BLYNK_WRITE(VIRTUAL_PIN_Automated_Feeder_1){
  switchFeeder_1 = param.asInt();

  Serial.println(switchFeeder_1);

  if (switchState == 1 && switchFeeder_1 ==1){
    servo5.write(45);
  } else if (switchState == 1 && switchFeeder_1 == 0){
    servo5.write(0);
  } else if (switchState == 0 && switchFeeder_1 == 0){
    automated_ultrasonic1();
  } else if (switchState == 0 && switchFeeder_1 == 1){
    automated_ultrasonic1();
  }
}

BLYNK_WRITE(VIRTUAL_PIN_Automated_Feeder_2){
  switchFeeder_2 = param.asInt();

  Serial.println(switchFeeder_2);

  if (switchState == 1 && switchFeeder_2 ==1){
    servo6.write(45);
  } 
  else if (switchState == 1 && switchFeeder_2 == 0){
    servo6.write(0);
  } else if (switchState == 0 && switchFeeder_2 == 0){
    automated_ultrasonic2();
  } else if (switchState == 0 && switchFeeder_2 == 1){
    automated_ultrasonic2();
  }
}

BLYNK_WRITE(VIRTUAL_PIN_Automated_Feeder_3){
  switchFeeder_3 = param.asInt();

  Serial.println(switchFeeder_3);

  if (switchState == 1 && switchFeeder_3 ==1){
    servo7.write(45);
  } 
  else if (switchState == 1 && switchFeeder_3 == 0){
    servo7.write(0);
  } else if (switchState == 0 && switchFeeder_3 == 0){
    automated_ultrasonic3();
  } else if (switchState == 0 && switchFeeder_3 == 1){
    automated_ultrasonic3();
  }
}

BLYNK_WRITE(VIRTUAL_PIN_Fan){
  switchFan = param.asInt();

  if (switchState == 1 && switchFan ==1){
    digitalWrite(RELAY_FAN, HIGH);
  } 
  else if (switchState == 1 && switchFan == 0){
    digitalWrite(RELAY_FAN, LOW);
  } else if (switchState == 0 && switchFan == 0){
    Automated_Temperature();
  } else if (switchState == 0 && switchFan == 1){
    Automated_Temperature();
  }
}

BLYNK_WRITE(VIRTUAL_PIN_Heater){
  switchHeater = param.asInt();

  if (switchState == 1 && switchHeater == 1){
    digitalWrite(RELAY_HEATER, HIGH);
  } 
  else if (switchState == 1 && switchHeater == 0){
    digitalWrite(RELAY_HEATER, LOW);
  } else if (switchState == 0 && switchHeater == 0){
    Automated_Temperature();
  } else if (switchState == 0 && switchHeater == 1){
    Automated_Temperature();
  }
}

BLYNK_WRITE(VIRTUAL_PIN_Automated_Ventilation){
  switchVentilation = param.asInt();

  if (switchState == 1 && switchVentilation == 1){
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);
    servo1.write(90);
    servo2.write(90);
    servo3.write(90);
    servo4.write(90);
  } 
  else if (switchState == 1 && switchVentilation == 0){
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(0);
    servo2.write(0);
    servo3.write(0);
    servo4.write(0);
  } else if (switchState == 0 && switchVentilation == 0){
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(0);
    servo2.write(0);
    servo3.write(0);
    servo4.write(0);
  } else if (switchState == 0 && switchVentilation == 1){
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(0);
    servo2.write(0);
    servo3.write(0);
    servo4.write(0);
  }
}

// BLYNK_WRITE(VIRTUAL_PIN_Light_Switch){
//   Lightswitch = param.asInt();

//   if (switchState == 1 && Lightswitch ==1){
//     digitalWrite(RELAY_LED, HIGH);
//   } 
//   else if (switchState == 1 && Lightswitch == 0){
//     digitalWrite(RELAY_LED, LOW);
//   } else if (switchState == 0 && Lightswitch == 0){
//     Automated_Sunlight_Sensor();
//   } else if (switchState == 0 && Lightswitch == 1){
//     Automated_Sunlight_Sensor();
//   }
// }

void  automated_ultrasonic1() {
  digitalWrite(trigPin1, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin1, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin1, LOW);

  duration1 = pulseIn(echoPin1, HIGH);

  distance1 = duration1 * 0.034 / 2;

  Serial.print("Distance 1: ");
  Serial.println(distance1);

  Blynk.virtualWrite(VIRTUAL_PIN_Substrate_Sensor_1, (String(distance1) + " cm"));

  if (distance1 > 20) {
    servo5.write(45);
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Feeder_1, HIGH);
  }
  if (distance1 < 16) {
    servo5.write(0);
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Feeder_1, LOW); 
  } 
  delay(1000);
}

void  automated_ultrasonic2() {
  digitalWrite(trigPin2, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin2, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin2, LOW);

  duration2 = pulseIn(echoPin2, HIGH);

  distance2 = duration2 * 0.034 / 2;

  Serial.print("Distance 2: ");
  Serial.println(distance2);

  Blynk.virtualWrite(VIRTUAL_PIN_Substrate_Sensor_2, (String(distance2) + " cm"));

  if (distance2 > 20) {
    servo6.write(45);
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Feeder_2, HIGH);
  }
  if (distance2 < 16) {
    servo6.write(0);
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Feeder_2, LOW); 
  } 
  delay(1000);
}

void  automated_ultrasonic3() {
  digitalWrite(trigPin3, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin3, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin3, LOW);

  duration3 = pulseIn(echoPin3, HIGH);

  distance3 = duration3 * 0.034 / 2;

  Serial.print("Distance 3: ");
  Serial.println(distance3);

  Blynk.virtualWrite(VIRTUAL_PIN_Substrate_Sensor_3, (String(distance3) + " cm"));

  if (distance3 > 20) {
    servo7.write(45);
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Feeder_3, HIGH);
  }
  if (distance3 < 16) {
    servo7.write(0);
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Feeder_3, LOW); 
  } 
  delay(1000);
}

void  manual_ultrasonic1() {
  digitalWrite(trigPin1, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin1, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin1, LOW);

  duration1 = pulseIn(echoPin1, HIGH);

  distance1 = duration1 * 0.034 / 2;

  Serial.print("Distance 1: ");
  Serial.println(distance1);

  Blynk.virtualWrite(VIRTUAL_PIN_Substrate_Sensor_1, (String(distance1) + " cm"));

  if (switchFeeder_1 == 1) {
    servo5.write(45);
  } 
  if (switchFeeder_1 == 0) {
    servo5.write(0);
  }
  delay(1000);
}

void  manual_ultrasonic2() {
  digitalWrite(trigPin2, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin2, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin2, LOW);

  duration2 = pulseIn(echoPin2, HIGH);

  distance2 = duration2 * 0.034 / 2;

  Serial.print("Distance 2: ");
  Serial.println(distance2);

  Blynk.virtualWrite(VIRTUAL_PIN_Substrate_Sensor_2, (String(distance2) + " cm"));

  if (switchFeeder_2 == 1) {
    servo6.write(45);
  } 
  if (switchFeeder_2 == 0) {
    servo6.write(0);
  }
  delay(1000);
}

void  manual_ultrasonic3() {
  digitalWrite(trigPin3, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin3, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin3, LOW);

  duration3 = pulseIn(echoPin3, HIGH);

  distance3 = duration3 * 0.034 / 2;

  Serial.print("Distance 3: ");
  Serial.println(distance3);

  Blynk.virtualWrite(VIRTUAL_PIN_Substrate_Sensor_3, (String(distance3) + " cm"));

  if (switchFeeder_3 == 1) {
    servo7.write(45);
  } 
  if (switchFeeder_3 == 0) {
    servo7.write(0);
  }
  delay(1000);
}

void Automated_Temperature(){
  delay(1000);
  sensors_event_t event;
  dht.temperature().getEvent(&event);

  if (isnan(event.temperature)) {
    Serial.println(F("Error reading temperature!"));
  }
  else {
    Serial.print(F("Temperature: "));
    Serial.print(event.temperature);
    Serial.println(F("°C"));
  } 

  Blynk.virtualWrite(VIRTUAL_PIN_Temperature, event.temperature);
 
  if(event.temperature > 30) {
    Blynk.virtualWrite(VIRTUAL_PIN_Fan, HIGH);
    Blynk.virtualWrite(VIRTUAL_PIN_Heater, LOW);
    digitalWrite(RELAY_FAN, HIGH);
    digitalWrite(RELAY_HEATER, LOW);
  } else if(event.temperature < 30) {
    Blynk.virtualWrite(VIRTUAL_PIN_Fan, LOW);
    Blynk.virtualWrite(VIRTUAL_PIN_Heater, HIGH);
    digitalWrite(RELAY_FAN, LOW); 
    digitalWrite(RELAY_HEATER, HIGH); 
  } 
  delay(1000);
} 

void Manual_Temperature(){
  delay(1000);
  sensors_event_t event;
  dht.temperature().getEvent(&event);

  if (isnan(event.temperature)) {
    Serial.println(F("Error reading temperature!"));
  }
  else {
    Serial.print(F("Temperature: "));
    Serial.print(event.temperature);
    Serial.println(F("°C"));
  }

  Blynk.virtualWrite(VIRTUAL_PIN_Temperature, event.temperature);
 
  if(switchFan == 1 && switchHeater == 1) {
    digitalWrite(RELAY_FAN, HIGH);
    digitalWrite(RELAY_HEATER, HIGH);
  } else if (switchFan == 1 && switchHeater == 0) {
    digitalWrite(RELAY_FAN, HIGH);
    digitalWrite(RELAY_HEATER, LOW);
  } else if (switchFan == 0 && switchHeater == 0) {
    digitalWrite(RELAY_FAN, LOW);
    digitalWrite(RELAY_HEATER, LOW);
  } else if (switchFan == 0 && switchHeater == 1) {
    digitalWrite(RELAY_FAN, LOW);
    digitalWrite(RELAY_HEATER, HIGH);
  }
  delay(1000);
} 

void Manual_Humidity() {
  delay(1000);
  sensors_event_t event;
  dht.humidity().getEvent(&event);

  if (isnan(event.relative_humidity)) {
    Serial.println(F("Error reading humidity!"));
  }
  else {
    Serial.print(F("Humidity: "));
    Serial.print(event.relative_humidity);
    Serial.println(F("%"));
  }

  Blynk.virtualWrite(VIRTUAL_PIN_Humidity, event.relative_humidity);
 
  if(switchVentilation == 1) {
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);   
    servo1.write(90);
    servo2.write(90);
    servo3.write(90);
    servo4.write(90);
  } else if (switchVentilation == 0) {
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(0);
    servo2.write(0);
    servo3.write(0);
    servo4.write(0);
  }
  delay(1000);
}

void Automated_Humidity() {
  delay(1000);
  sensors_event_t event;
  dht.humidity().getEvent(&event);

  if (isnan(event.relative_humidity)) {
    Serial.println(F("Error reading humidity!"));
  }
  else {
    Serial.print(F("Humidity: "));
    Serial.print(event.relative_humidity);
    Serial.println(F("%"));
  }

  Blynk.virtualWrite(VIRTUAL_PIN_Humidity, event.relative_humidity);
 
  if(event.relative_humidity > 65) {
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Ventilation, HIGH);
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);   
    servo1.write(90);
    servo2.write(90);
    servo3.write(90);
    servo4.write(90);
  }else if(event.relative_humidity < 65) {
    Blynk.virtualWrite(VIRTUAL_PIN_Automated_Ventilation, LOW);
    servo1.write(60);
    servo2.write(60);
    servo3.write(60);
    servo4.write(60);
    delay(500);
    servo1.write(30);
    servo2.write(30);
    servo3.write(30);
    servo4.write(30);
    delay(500);
    servo1.write(0);
    servo2.write(0);
    servo3.write(0);
    servo4.write(0);
  }
  delay(1000);
}

// void Automated_Sunlight_Sensor(){

//   Blynk.virtualWrite(VIRTUAL_PIN_Temperature, ?);
 
//   if(data.temperature > 30) {
//     Blynk.virtualWrite(VIRTUAL_PIN_Sunlight_Sensor, HIGH);
//     digitalWrite(RELAY_LED, HIGH);
//   }

//   if(data.temperature < 30) {
//     Blynk.virtualWrite(VIRTUAL_PIN_Sunlight_Sensor, LOW);
//     digitalWrite(RELAY_LED, LOW); 
//   } 
//   delay(1000);
// } 

void Condition() {
  if(switchState == 1) {
    manual_ultrasonic1();
    manual_ultrasonic2();
    manual_ultrasonic3();
    Manual_Temperature();
    Manual_Humidity();
  } else if(switchState == 0) {
    automated_ultrasonic1();
    automated_ultrasonic2();
    automated_ultrasonic3();
    Automated_Temperature();
    Automated_Humidity();
  }
  delay(1000);
}

void loop(){
  Blynk.run(); //menjalankan blynk
  timer.run(); //menjalankan timer

  Condition();
  delay(1000);
}





