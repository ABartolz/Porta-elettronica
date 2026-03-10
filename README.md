# PORTA ELETTRONICA
Questo documento di testo si occupa di descrivere il nostro processo seguendo vari passaggi.

**1. Componenti**

*V = In possesso, X = Da comprare/Acquisire*

Per realizzare questo progetto, io e il mio compare abbiamo bisogno del seguente materiale:
- 1 Arduino nano **(X)**
- 1 RFID module **(X)**
- 1 Breadboard small **(V)**
- 1 5V Buzzer **(X)**
- X Quantità di jumper /Bozza/ **(V)**
- 2 Door hinges **(X)**
- 1 RFID reader **(X)**
- 1 Schermo LCD **(X)**
- 1 L2C module **(X)**
- X RFID tags /Bozza/ **(X)**
- 1 SG90 servo motor **(X)**
- 1 lucchetto fisico per la porta **(X)**
- Cartone **(V)**

**2. Procedimento**

**3. Codice (C++)**

**UD SCAN**
 
#include <SPI.h>
#include <MFRC522.h>


#define RST_PIN 9
#define SS_PIN  10


MFRC522 mfrc522(SS_PIN, RST_PIN);


void setup() {
  Serial.begin(9600);
  while (!Serial);
  SPI.begin();
  mfrc522.PCD_Init();
  delay(4);
  mfrc522.PCD_DumpVersionToSerial();
  Serial.println(F("Scan PICC to see UID, SAK, type, and data blocks..."));
}


void loop() {
  if ( ! mfrc522.PICC_IsNewCardPresent()) {
    return;
  }


  if ( ! mfrc522.PICC_ReadCardSerial()) {
    return;
  }
  mfrc522.PICC_DumpToSerial(&(mfrc522.uid));
}

**MAIN CODE**

/*Door lock system with Arduino Nano
   Home Page
*/

#include <Servo.h>
#include <LiquidCrystal_I2C.h>
#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 10
#define RST_PIN 9
#define buzzer 2
#define servoPin 3

String UID = "*************";//Enter your card ID
byte lock = 0;

Servo servo;
LiquidCrystal_I2C lcd(0x27, 16, 2);
MFRC522 rfid(SS_PIN, RST_PIN);

void setup() {
  Serial.begin(9600);
  servo.write(70);
  lcd.init();
  lcd.backlight();
  servo.attach(servoPin);
  SPI.begin();
  rfid.PCD_Init();
  pinMode(buzzer, OUTPUT);
}

void loop() {
  lcd.setCursor(4, 0);
  lcd.print("Welcome!");
  lcd.setCursor(1, 1);
  lcd.print("Put your card");

  if ( ! rfid.PICC_IsNewCardPresent())
    return;
  if ( ! rfid.PICC_ReadCardSerial())
    return;

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Scanning");
  Serial.print("NUID tag is :");
  String ID = "";
  for (byte i = 0; i < rfid.uid.size; i++) {
    lcd.print(".");
    ID.concat(String(rfid.uid.uidByte[i] < 0x10 ? " 0" : " "));
    ID.concat(String(rfid.uid.uidByte[i], HEX));
    delay(300);
  }
  ID.toUpperCase();

  if (ID.substring(1) == UID && lock == 0 ) {
    digitalWrite(buzzer, HIGH);
    delay(300);
    digitalWrite(buzzer, LOW);
    servo.write(50);
    delay(100);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Locked");
    delay(1500);
    lcd.clear();
    lock = 1;
  } 
  
  else if (ID.substring(1) == UID && lock == 1 ) {
    digitalWrite(buzzer, HIGH);
    delay(300);
    digitalWrite(buzzer, LOW);
    servo.write(110);
    delay(100);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Unlocked");
    delay(1500);
    lcd.clear();
    lock = 0;
  } 
  
  else {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Wrong card!");
    digitalWrite(buzzer, HIGH);
    delay(200);
    digitalWrite(buzzer, LOW);
    delay(200);
    digitalWrite(buzzer, HIGH);
    delay(200);
    digitalWrite(buzzer, LOW);
    delay(200);
    digitalWrite(buzzer, HIGH);
    delay(200);
    digitalWrite(buzzer, LOW);
    delay(200);
    delay(1500);
    lcd.clear();
  }
}

**4. Dimostrazione & Progetto finito**
