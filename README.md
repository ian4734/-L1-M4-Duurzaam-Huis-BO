# M4-Duurzaam-Huis

TAKEN:

kyano
1
De actuele buiten temperatuur
2
Buiten- en binnentemperatuur
3
Ik kan op het dashboard zien of een lamp, in het huisje, aan of uit is.
Ik kan via het dashboard een lamp, in het huisje, aan of uit zetten.
Ik kan als het een bepaalde tijd is een lamp, in het huisje, uit of aan laten gaan
(doen we allemaal samen)

ian
1
De weersverwachting
2
Opbrengst energie van zonnepanelen
3
Ik kan op het dashboard zien of een lamp, in het huisje, aan of uit is.
Ik kan via het dashboard een lamp, in het huisje, aan of uit zetten.
Ik kan als het een bepaalde tijd is een lamp, in het huisje, uit of aan laten gaan
(doen we allemaal samen)

deon
1
De zonsopkomst en zonsondergang (heb ik af)
2
Energieverbruik
3
Ik kan op het dashboard zien of een lamp, in het huisje, aan of uit is.
Ik kan via het dashboard een lamp, in het huisje, aan of uit zetten.
Ik kan als het een bepaalde tijd is een lamp, in het huisje, uit of aan laten gaan
(doen we allemaal samen) 

CODE ARDUINO:
#include <DHT.h>
#include <LiquidCrystal.h>

// ---------- Pin-configuratie ----------
#define DHTPIN_IN   14      // DHT11 Binnen (mogelijk checken)
#define DHTPIN_OUT  24      // DHT11 buiten  (mogelijk nog checken)
#define DHTTYPE     DHT11

const int trig   = 2;
const int echo   = 4;
const int buzzer = 13;

// pinnen voor zonnepanelen en windmolen
const int SOLAR_A_PIN = A5;
const int SOLAR_B_PIN = A4;
const int WIND_PIN    = A6;

// ---------- Objecten ----------
DHT dhtIn(DHTPIN_IN, DHTTYPE);
DHT dhtOut(DHTPIN_OUT, DHTTYPE);
const int rs = 3, enable = 5,  d4 = 9, d5 = 10, d6 = 11, d7 = 12;
LiquidCrystal LCD(rs,enable, d4, d5, d6, d7);


// ---------- Globale waarden ----------
float TempInside  = 0;
float TempOutside = 0;

void setup() {
  Serial.begin(9600);

  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
  pinMode(buzzer, OUTPUT);

  dhtIn.begin();
  dhtOut.begin();

  LCD.begin(16, 2);
  LCD.setCursor(0, 0);
  LCD.print("Initialiseren...");
  delay(2000);
  LCD.clear();
}

void loop() {
  // Toon temperatuur ~5s, dan zonne-energie ~5s, dan windenergie ~5s.
  // Elke "tick" duurt ~50 ms, 300 ticks ~= 15 seconden.
  for (uint16_t i = 0; i < 300; i++) {
    if (i == 0) {
      LCD.clear();
      DisplayTemperature();
    }
    else if (i == 100) {
      LCD.clear();
      DisplaySolarPower();
    }
    else if (i == 200) {
      LCD.clear();
      DisplayWindPower();
    }

    // Afstand meten en eventueel piepen
    if (ReadUltrasonicSensor() < 20) {
      digitalWrite(buzzer, HIGH);   // buzzer aan
      delay(5);
      digitalWrite(buzzer, LOW);    // buzzer uit
      delay(45);
    }
    else {
      delay(50);
    }
  }
}

float ReadUltrasonicSensor() {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  // Echo meten met timeout (30 ms) zodat de loop niet vastloopt
  long duration = pulseIn(echo, HIGH, 30000);

  float distance = duration * 0.0343 / 2.0;

  // Geen echo (timeout) -> behandel als ver weg, niet als 0 cm
  if (distance <= 0) distance = 999;

  return distance;
}

void DisplayTemperature() {
  // Lees binnen- & buitentemperatuur van de DHT11's
  ReadDHT11();

  // Toon op het LCD
  LCD.setCursor(0, 0);
  LCD.print("Temp In: ");
  LCD.print(TempInside, 1);
  LCD.print("c");

  LCD.setCursor(0, 1);
  LCD.print("Temp Out: ");
  LCD.print(TempOutside, 1);
  LCD.print("c");
}

void ReadDHT11() {
  float tIn  = dhtIn.readTemperature();
  float tOut = dhtOut.readTemperature();

  if (!isnan(tIn))  TempInside  = round(tIn  * 10) / 10.0;
  if (!isnan(tOut)) TempOutside = round(tOut * 10) / 10.0;

  Serial.println();
  Serial.println("Temp Out: " + String(TempOutside) + " C");

  LCD.setCursor(0, 0);
  LCD.print("Temp In: "  + String(TempInside)  + " C");

  LCD.setCursor(0, 1);
  LCD.print("Temp Out: " + String(TempOutside) + " C");
}

void DisplaySolarPower() {
  int solarA = analogRead(SOLAR_A_PIN);
  int solarB = analogRead(SOLAR_B_PIN);

  LCD.setCursor(0, 0);
  LCD.print("Solar A: ");
  LCD.print(solarA);

  LCD.setCursor(0, 1);
  LCD.print("Solar B: ");
  LCD.print(solarB);
}

void DisplayWindPower() {
  int wind = analogRead(WIND_PIN);

  LCD.setCursor(0, 0);
  LCD.print("Windmill: ");
  LCD.print(wind);
}
