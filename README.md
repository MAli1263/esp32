# esp32

// Pins definieren
const int trigPin = 12;    // SR01 V3 Trig
const int echoPin = 13;    // SR01 V3 Echo
const int ledPin = 18;     // White LED
const int buzzerPin = 16;  // Passive Buzzer

// I2C LCD Display
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// I2C-Adresse anpassen (0x27 oder 0x3F)
LiquidCrystal_I2C lcd(0x27, 16, 2); // Falls 0x27 nicht funktioniert, probiere 0x3F

// Variablen für die Logik
bool objectDetected = false;
unsigned long lastBlinkTime = 0;
const unsigned long blinkInterval = 500; // Blink-Intervall: 500 ms

void setup() {
  Serial.begin(115200);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);

  // LCD initialisieren
  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("System Ready");

  digitalWrite(ledPin, LOW);
  noTone(buzzerPin);

  Serial.println("System bereit. Warte auf Objekt...");
}

void loop() {
  // Abstand messen
  long distance = measureDistance();
  Serial.print("Abstand: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Objekt erkannt? (z. B. < 20 cm)
  if (distance < 20) {
    if (!objectDetected) {
      // Erstes Mal: Piepton + LED an + LCD aktualisieren
      tone(buzzerPin, 2000, 200); // 2000 Hz für 200 ms
      digitalWrite(ledPin, HIGH);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("SOMEONE IS");
      lcd.setCursor(0, 1);
      lcd.print("THERE");
      objectDetected = true;
      Serial.println("Objekt erkannt! Piepton + LED an + LCD aktualisiert.");
    }
    // LED blinken lassen
    blinkLED();
  }
  else {
    // Kein Objekt mehr: Alles zurücksetzen
    if (objectDetected) {
      noTone(buzzerPin);
      digitalWrite(ledPin, LOW);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("NO ONE HERE");
      objectDetected = false;
      Serial.println("Objekt verschwunden. LED aus, Buzzer stumm, LCD aktualisiert.");
    }
  }

  delay(50); // Kurze Pause für stabilere Messungen
}

// Abstandsmessung mit SR01 V3
long measureDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH);
  return (duration / 2) / 29.1; // Umrechnung in cm
}

// LED-Blinken
void blinkLED() {
  unsigned long currentMillis = millis();
  if (currentMillis - lastBlinkTime >= blinkInterval) {
    lastBlinkTime = currentMillis;
    digitalWrite(ledPin, !digitalRead(ledPin)); // Zustand umkehren
  }
}
