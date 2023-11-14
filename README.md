# posture-detection
how to write coding to detect posture angle with esp32,mpu6050 

#define BLYNK_TEMPLATE_ID "TMPL6fPddz4IZ"
#define BLYNK_TEMPLATE_NAME "mpu"
#define BLYNK_AUTH_TOKEN "OG3wTFZ8Qi80WHZITLWzedkafvmwbb0p"

#define BLYNK_PRINT Serial
#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <Wire.h>

Adafruit_MPU6050 mpu;

char ssid[] = "..."; // Your WiFi credentials.
char pass[] = " ";
 
const int MPU_addr = 0x68;
int16_t AcX, AcY, AcZ, Tmp, GyX, GyY, GyZ;
 
int minVal = 265;
int maxVal = 402;


void setup() {
  Wire.begin();
  Wire.beginTransmission(MPU_addr);
  Wire.write(0x6B);
  Wire.write(0);
  Wire.endTransmission(true);
  Serial.begin(9600);
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);
}

void loop() {
  Wire.beginTransmission(MPU_addr);
  Wire.write(0x3B);
  Wire.endTransmission(false);
  Wire.requestFrom(MPU_addr, 14, true);
  AcX = Wire.read() << 8 | Wire.read();
  AcY = Wire.read() << 8 | Wire.read();
  AcZ = Wire.read() << 8 | Wire.read();

  double x1 ; 
  double y1 ; 
  double z1 ; 

  double x2 ; 
  double y2 ; 
  double z2 ;

int xAng = map(AcX,minVal,maxVal,-90,90);
int yAng = map(AcY,minVal,maxVal,-90,90);
int zAng = map(AcZ,minVal,maxVal,-90,90);
 
x1= RAD_TO_DEG * (atan2(-yAng, -zAng)+PI);
y1= RAD_TO_DEG * (atan2(-xAng, -zAng)+PI);
z1= RAD_TO_DEG * (atan2(-yAng, -xAng)+PI);

  double dotProduct = x1 * x2 + y1 * y2 + z1 * z2;

  // Calculate magnitudes
  double magnitude1 = sqrt(x1 * x1 + y1 * y1 + z1 * z1);
  double magnitude2 = sqrt(x2 * x2 + y2 * y2 + z2 * z2);

  // Calculate cosine of the angle
  double cosTheta = dotProduct / (magnitude1 * magnitude2);

  // Calculate angle in radians
  double angleRad = acos(cosTheta);

  // Convert angle to degrees
  double angleDeg = angleRad * RAD_TO_DEG;


  Serial.print("AngleX= ");
  Serial.println(x1);
 
  Serial.print("AngleY= ");
  Serial.println(y1);
 
  Serial.print("AngleZ= ");
  Serial.println(z1);

  Serial.print("Angle between vectors= ");
  Serial.println(angleDeg);

  Serial.println("-----------------------------------------");

  Blynk.virtualWrite(V1, x1);
  Blynk.virtualWrite(V2, y1);
  Blynk.virtualWrite(V3, z1);
  Blynk.virtualWrite(V4, x2);
  Blynk.virtualWrite(V5, y2);
  Blynk.virtualWrite(V6, z2);
  Blynk.virtualWrite(V7, angleDeg);

  delay(1000);
  Blynk.run();
}
