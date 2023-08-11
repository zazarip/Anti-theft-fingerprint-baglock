#include "TinyGPS++.h"
#include <SoftwareSerial.h>
#include <Servo.h>

#include <Adafruit_Fingerprint.h>
#if (defined(__AVR__) || defined(ESP8266)) && !defined(__AVR_ATmega2560__)
SoftwareSerial mySerial(2, 3);

#else
#define mySerial Serial1
#endif

Servo myservo;
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);
int buzzer = 4;
int count, flag, count2 = 0;
float a, d;

void setup()
{
  Serial.begin(9600);
  pinMode(buzzer, OUTPUT);
  myservo.attach(8);
  myservo.write(0);
  while (!Serial);  
  delay(100);
  Serial.println("\n\nAdafruit finger detect test");

  // set the data rate for the sensor serial port
  finger.begin(57600);
  delay(5);
  if (finger.verifyPassword()) {
    Serial.println("Found fingerprint sensor!");
  } else {
    Serial.println("Did not find fingerprint sensor :(");
    while (1) {
      delay(1);
    }
  }

  Serial.println(F("Reading sensor parameters"));
  finger.getParameters();
  Serial.print(F("Status: 0x")); Serial.println(finger.status_reg, HEX);
  Serial.print(F("Sys ID: 0x")); Serial.println(finger.system_id, HEX);
  Serial.print(F("Capacity: ")); Serial.println(finger.capacity);
  Serial.print(F("Security level: ")); Serial.println(finger.security_level);
  Serial.print(F("Device address: ")); Serial.println(finger.device_addr, HEX);
  Serial.print(F("Packet len: ")); Serial.println(finger.packet_len);
  Serial.print(F("Baud rate: ")); Serial.println(finger.baud_rate);

  finger.getTemplateCount();

  if (finger.templateCount == 0) {
    Serial.print("Sensor doesn't contain any fingerprint data. Please run the 'enroll' example.");
  }
  else {
    Serial.println("Waiting for valid finger...");
    Serial.print("Sensor contains "); Serial.print(finger.templateCount); Serial.println(" templates");
  }
  for (int bz = 0; bz < 1; bz++)
  {
    digitalWrite(buzzer, HIGH);
    delay(200);
    digitalWrite(buzzer, LOW);
    delay(200);
  }
  myservo.write(0);
}

void loop()                  
{
  getFingerprintID();
  delay(50);      
}

uint8_t getFingerprintID() {
  uint8_t p = finger.getImage();
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image taken");
      digitalWrite(buzzer, HIGH); delay(30);
      digitalWrite(buzzer, LOW);  delay(30);
      break;
    case FINGERPRINT_NOFINGER:
      Serial.println("No finger detected");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");
      return p;
    case FINGERPRINT_IMAGEFAIL:
      Serial.println("Imaging error");
      return p;
    default:
      Serial.println("Unknown error");
      return p;
  }

  p = finger.image2Tz();
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Image converted");
      break;
    case FINGERPRINT_IMAGEMESS:
      Serial.println("Image too messy");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Communication error");
      return p;
    case FINGERPRINT_FEATUREFAIL:
      Serial.println("Could not find fingerprint features");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      Serial.println("Could not find fingerprint features");
      return p;
    default:
      Serial.println("Unknown error");
      return p;
  }

  p = finger.fingerSearch();
  if (p == FINGERPRINT_OK) {
    Serial.println("Found a print match!");
    digitalWrite(buzzer, HIGH); delay(200);
    digitalWrite(buzzer, LOW);  delay(200);
    digitalWrite(buzzer, HIGH); delay(200);
    digitalWrite(buzzer, LOW);  delay(200);
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
    Serial.println("Communication error");
    return p;
  } else if (p == FINGERPRINT_NOTFOUND) {
    count++;
    Serial.print("count : ");
    Serial.println(count);
    Serial.println("Did not find a match");
    digitalWrite(buzzer, HIGH); delay(70);
    digitalWrite(buzzer, LOW);  delay(70);
    digitalWrite(buzzer, HIGH); delay(70);
    digitalWrite(buzzer, LOW);  delay(70);

    if (count == 3)
    {
      for (int bz = 0; bz < 3; bz++)
      {
        digitalWrite(buzzer, HIGH);
        delay(50);
        digitalWrite(buzzer, LOW);
        delay(50);
      }
      Serial.println("Send gps signal");
      send_gps();
      count = 0;
    }
    return p;
  } else {
    Serial.println("Unknown error");
    return p;
  }

  Serial.print("Found ID #");
  Serial.print(finger.fingerID);
  Serial.print(" with confidence of ");
  Serial.println(finger.confidence);

  if (finger.fingerID == 1 && flag == 0)
  {
    Serial.println("Unlocked");
    myservo.write(90);
    delay(1000);
    flag = 1;
    count = 0;
    return finger.fingerID;
  }
  else if (finger.fingerID == 1 && flag == 1);
  {
    Serial.println("Locked");
    myservo.write(0);
    delay(1000);
    flag = 0;
    count = 0;
  }
  return finger.fingerID;
}

int getFingerprintIDez() {
  uint8_t p = finger.getImage();
  if (p != FINGERPRINT_OK)  return -1;

  p = finger.image2Tz();
  if (p != FINGERPRINT_OK)  return -1;

  p = finger.fingerFastSearch();
  if (p != FINGERPRINT_OK)  return -1;

  Serial.print("Found ID #");
  Serial.print(finger.fingerID);
  Serial.print(" with confidence of ");
  Serial.println(finger.confidence);
  return finger.fingerID;
}

void send_gps()
{
  SoftwareSerial SIM900(5, 6); // 5 = TX, 6 = RX
  SIM900.begin(115200);
  Serial.println("Please wait...");
  SIM900.print("AT+CMGF=1\r");
  delay(1000);
  SIM900.print("AT+CNMI=2,2,0,0,0\r");
  delay(1000);
  SIM900.end();
  SoftwareSerial serial_gps(10, 12);  // 12 = TX, 10 = RX
  TinyGPSPlus gps;
  Serial.begin(9600);
  serial_gps.begin(9600);
  Serial.println("GPS Start");
  for (int i = 1; i <= 16; i++) {
    while (serial_gps.available())
    {
      gps.encode(serial_gps.read());
    }
    a = gps.location.lat();
    d = gps.location.lng();
    Serial.print("latt:");
    Serial.println(a, 6);
    Serial.print("long:");
    Serial.println(d, 6);
    if (gps.location.isUpdated())
    {

    }
  }
  serial_gps.end();
  SendMessage1();
  delay(2000);
  SendMessage2();
  finger.begin(57600);
  delay(5);
  if (finger.verifyPassword()) {
    Serial.println("Found fingerprint sensor!");
  } else {
    Serial.println("Did not find fingerprint sensor :(");
    while (1) {
      delay(1);
    }
  }
}

void SendMessage1()
{
  SoftwareSerial SIM900(5, 6);// 5 = TX, 6 = RX
  SIM900.begin(115200);
  SIM900.println("AT+CMGF=1");
  delay(1000);
  SIM900.println("AT+CMGS=\"+60136123138\"\r");
  delay(1000);
  SIM900.println("Theft alert! Please copy lat/long and paste to google to know latest location of bagpack.");
  delay(1000);
  SIM900.write(0x1a);
  delay(1000);
}

void SendMessage2()
{
  SoftwareSerial SIM900(5, 6);// 5 = TX, 6 = RX
  SIM900.begin(115200);
  SIM900.println("AT+CMGF=1");
  delay(1000);
  SIM900.println("AT+CMGS=\"+60136123138\"\r");
  delay(1000);
  SIM900.print(a, 6);
  SIM900.print(",");
  SIM900.println(d, 6);
  SIM900.write(0x1a);
  delay(1000);
}
