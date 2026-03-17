# PORTA ELETTRONICA
Questo documento di testo si occupa di descrivere il nostro processo seguendo vari passaggi.

**1. Componenti**

*V = In possesso, X = Da comprare/Acquisire*

Per realizzare questo progetto, io e il mio compare abbiamo bisogno del seguente materiale:
- 1 Arduino UNO **(V)**
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
- Polistirolo **(X)**

**2. Procedimento**

**3. Codice (C++)**

**UD SCAN**
 
/*
   RFID RC522 - Arduino UNO
   Lettura UID tag
*/

#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 10
#define RST_PIN 9

MFRC522 rfid(SS_PIN, RST_PIN);

void setup() {

  Serial.begin(9600);
  SPI.begin();          // Avvia SPI
  rfid.PCD_Init();      // Avvia RC522

  Serial.println("Avvicina una tessera RFID...");
}

void loop() {

  // Controlla se è presente una nuova carta
  if (!rfid.PICC_IsNewCardPresent()) {
    return;
  }

  // Legge la carta
  if (!rfid.PICC_ReadCardSerial()) {
    return;
  }

  Serial.print("UID tag: ");

  for (byte i = 0; i < rfid.uid.size; i++) {
    Serial.print(rfid.uid.uidByte[i], HEX);
    Serial.print(" ");
  }

  Serial.println();

  rfid.PICC_HaltA();
}

**MAIN CODE**

/* RFID Door Lock System - Arduino UNO */

#include <Servo.h>
#include <LiquidCrystal_I2C.h>
#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 10
#define RST_PIN 9
#define BUZZER 2
#define SERVO_PIN 3

String UID = "XX XX XX XX";   // inserisci qui l'UID della tua tessera

byte lock = 0;

Servo servo;
LiquidCrystal_I2C lcd(0x27, 16, 2);
MFRC522 rfid(SS_PIN, RST_PIN);

void setup() {

  Serial.begin(9600);

  pinMode(BUZZER, OUTPUT);

  servo.attach(SERVO_PIN);
  servo.write(70);   // posizione iniziale

  lcd.init();
  lcd.backlight();

  SPI.begin();
  rfid.PCD_Init();

  lcd.setCursor(0,0);
  lcd.print("RFID Door Lock");
  delay(2000);
  lcd.clear();
}

void loop() {

  lcd.setCursor(4,0);
  lcd.print("Welcome!");
  lcd.setCursor(1,1);
  lcd.print("Put your card");

  if (!rfid.PICC_IsNewCardPresent()) return;
  if (!rfid.PICC_ReadCardSerial()) return;

  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Scanning");

  String ID = "";

  for (byte i = 0; i < rfid.uid.size; i++) {

    lcd.print(".");
    ID.concat(String(rfid.uid.uidByte[i] < 0x10 ? " 0" : " "));
    ID.concat(String(rfid.uid.uidByte[i], HEX));

    delay(300);
  }

  ID.toUpperCase();

  Serial.println(ID);

  // CARD CORRETTA
  if (ID.substring(1) == UID && lock == 0) {

    digitalWrite(BUZZER, HIGH);
    delay(300);
    digitalWrite(BUZZER, LOW);

    servo.write(50);

    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Door Locked");

    delay(1500);
    lcd.clear();

    lock = 1;
  }

  else if (ID.substring(1) == UID && lock == 1) {

    digitalWrite(BUZZER, HIGH);
    delay(300);
    digitalWrite(BUZZER, LOW);

    servo.write(110);

    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Door Unlocked");

    delay(1500);
    lcd.clear();

    lock = 0;
  }

  // CARD SBAGLIATA
  else {

    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Wrong Card!");

    for(int i=0;i<3;i++){
      digitalWrite(BUZZER,HIGH);
      delay(200);
      digitalWrite(BUZZER,LOW);
      delay(200);
    }

    delay(1500);
    lcd.clear();
  }

  rfid.PICC_HaltA();
}

**4. Dimostrazione & Progetto finito**
