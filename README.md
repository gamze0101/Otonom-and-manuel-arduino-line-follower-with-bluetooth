# Otonom-and-manuel-arduino-line-follower-with-bluetooth
Hem manuel hem otonom robot araba- Gamze
#include <SoftwareSerial.h>

// Bluetooth için yazılım serisi (RX = 4, TX = 5)
SoftwareSerial BT(4, 5);

// Motor pinleri
#define ENA 9
#define motorA1 A1
#define motorA2 A2
#define motorB1 A3
#define motorB2 A4
#define ENB 10

// Sensör pinleri
#define sensor1 7  // Orta sensör
#define sensor2 8  // Sol sensör
#define sensor3 11 // Sağ sensör
#define sensor4 12 // En sağ sensör
#define sensor5 2  // En sol sensör

// Tuş pini
#define buttonPin 3

// Kontrol değişkenleri
int controlMode = 0;  // 0: Sensör modu, 1: Bluetooth modu
char command;
unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 50;

void setup() {
  Serial.begin(9600);     // Seri bağlantıyı başlat
  BT.begin(9600);         // Bluetooth bağlantısını başlat

  // Motor pinlerini çıkış olarak ayarla
  pinMode(motorA1, OUTPUT);
  pinMode(motorA2, OUTPUT);
  pinMode(motorB1, OUTPUT);
  pinMode(motorB2, OUTPUT);
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);

  // Sensör pinlerini giriş olarak ayarla
  pinMode(sensor1, INPUT);
  pinMode(sensor2, INPUT);
  pinMode(sensor3, INPUT);
  pinMode(sensor4, INPUT);
  pinMode(sensor5, INPUT);

  // Tuş pinini giriş olarak ayarla ve pull-up direncini etkinleştir
  pinMode(buttonPin, INPUT_PULLUP);

  Serial.println("Başlangıç: Çizgi takip modu aktif.");  // Başlangıç mesajı
}

void loop() {
  checkButton();              // Tuşu kontrol et
  if (BT.available()) {
    Serial.println("Bluetooth verisi mevcut."); // Debug mesajı
    processBluetoothCommand(); // Bluetooth komutunu işle
  }
  if (controlMode == 0) {
    Serial.println("Sensör modunda."); // Debug mesajı
    followLine();             // Çizgi takip fonksiyonunu çalıştır
  } else {
    Serial.println("Bluetooth modunda."); // Debug mesajı
  }
}

void checkButton() {
  if (digitalRead(buttonPin) == LOW && (millis() - lastDebounceTime) > debounceDelay) {
    lastDebounceTime = millis();
    controlMode = !controlMode; // Kontrol modunu değiştir
    stopMotors();               // Motorları durdur
    String modeMessage = controlMode ? "Bluetooth modu aktif." : "Çizgi takip modu aktif.";
    Serial.println(modeMessage); // Mod mesajını yazdır
    BT.println(modeMessage);     // Mod mesajını Bluetooth ile gönder
    delay(500);                  // Kısa bir bekleme süresi
  }
}

void processBluetoothCommand() {
  if (BT.available()) {
    Serial.println("Bluetooth verisi okunuyor."); // Debug mesajı
    command = BT.read();         // Bluetooth komutunu oku
    controlMode = 1;             // Kontrol modunu Bluetooth moduna ayarla
    String btMessage = "Bluetooth komutu alındı: ";
    Serial.println(btMessage + command);  // Bluetooth komut mesajını yazdır
    controlMotors(command);      // Motorları kontrol et
  }
}

void followLine() {
  int sensorValues[] = {
    digitalRead(sensor5), // en sol
    digitalRead(sensor2), // sol
    digitalRead(sensor1), // orta
    digitalRead(sensor3), // sağ
    digitalRead(sensor4)  // en sağ
  };

  Serial.print("Sensörler: ");
  for (int i = 0; i < 5; i++) {
    Serial.print(sensorValues[i]);
    Serial.print(" ");
  }
  Serial.println();

  if (sensorValues[2] == 0 && sensorValues[1] == 1 && sensorValues[3] == 1) {
    forward();
  } else if (sensorValues[1] == 0 && sensorValues[2] == 1 && sensorValues[3] == 1) {
    turnLeft();
  } else if (sensorValues[1] == 1 && sensorValues[2] == 1 && sensorValues[3] == 0) {
    turnRight();
  } else if (sensorValues[1] == 0 && sensorValues[2] == 0 && sensorValues[3] == 1) {
    turnLeft();
  } else if (sensorValues[1] == 1 && sensorValues[2] == 0 && sensorValues[3] == 0) {
    turnRight();
  } else if (sensorValues[1] == 1 && sensorValues[2] == 1 && sensorValues[3] == 1 && sensorValues[4] == 1 && sensorValues[0] == 1) {
    stopMotors();
  }
}

void controlMotors(char cmd) {
  switch (cmd) {
    case 'F': forward(); break;
    case 'B': moveBackward(); break;
    case 'L': turnLeft(); break;
    case 'R': turnRight(); break;
    case 'S': stopMotors(); break;
    default: Serial.println("Geçersiz komut!"); break;
  }
}

// Motor hareketlerini kontrol eden fonksiyonlar
void motorControl(int motorA1State, int motorA2State, int motorB1State, int motorB2State) {
  digitalWrite(motorA1, motorA1State);
  digitalWrite(motorA2, motorA2State);
  digitalWrite(motorB1, motorB1State);
  digitalWrite(motorB2, motorB2State);
  analogWrite(ENA, 110);  // Motor hızını ayarla
  analogWrite(ENB, 110);  // Motor hızını ayarla
}

void forward() {
  Serial.println("İleri hareket");
  motorControl(LOW, HIGH, HIGH, LOW);
}

void turnRight() {
  Serial.println("Sağa dönüyor");
  motorControl(HIGH, LOW, HIGH, LOW);
}

void turnLeft() {
  Serial.println("Sola dönüyor");
  motorControl(LOW, HIGH, LOW, HIGH);
}

void moveBackward() {
  Serial.println("Geri hareket");
  motorControl(HIGH, LOW, LOW, HIGH);
}

void stopMotors() {
  Serial.println("Motorlar durdu");
  motorControl(LOW, LOW, LOW, LOW);
}
