#include <DHT.h> 

#include <SPI.h> 

#include <MFRC522.h> 

#include <Wire.h>                // Librería para comunicación I2C 

#include <LiquidCrystal_I2C.h>  

//Incluyo la Libreria del Servomotor 

#include <Servo.h> 

  

Servo servoPuerta; 

 

const int lcdAddress = 0x27;     // Dirección I2C del módulo LCD 

LiquidCrystal_I2C lcd(lcdAddress, 16, 2); // Configura el LCD de 16x2 

  

// Definimos el pin digital donde se conecta el sensor 

#define DHTPIN 2 

// Dependiendo del tipo de sensor 

#define DHTTYPE DHT11 

  

// Inicializamos el sensor DHT11 

DHT dht(DHTPIN, DHTTYPE); 

  

#define SS_PIN 10 

#define RST_PIN 9 

MFRC522 rfid(SS_PIN, RST_PIN);  // Crear una instancia del lector RFID 

  

// Definir los UIDs permitidos (en este caso, dos tarjetas permitidas) 

byte uidPermitida1[] = {0xBA, 0xA9, 0xA4, 0x15}; // Primera tarjeta 

byte uidPermitida2[] = {0x6C, 0xB2, 0x41, 0x18}; // Segunda tarjeta 

  

void setup() { 

   

  servoPuerta.attach(7); 

  servoPuerta.write(0); 

  

  dht.begin(); 

  

  Serial.begin(9600); 

  SPI.begin();  // Iniciar la comunicación SPI 

  rfid.PCD_Init();  // Iniciar el lector RFID 

  Serial.println("Escanear una tarjeta para leer el UID:"); 

  dht.readTemperature(); 

} 

 void loop() { 

  

    // Leemos la temperatura en grados centígrados (por defecto) 

  float t = dht.readTemperature(); 

   

  lcd.begin(); 

  lcd.backlight(); 

  lcd.clear(); 

  lcd.setCursor(0,0); 

  lcd.print("--Temperatura--"); 

  lcd.setCursor(0,1); 

  lcd.print(t); 

  lcd.print( "%"); 

  

  delay(500); 

  lcd.clear(); 

  lcd.print("Escribe cantidad de hielo"); 

  delay(500); 

   

  // Verificar si hay una nueva tarjeta presente 

  if (!rfid.PICC_IsNewCardPresent()) 

    return; 

  

  // Verificar si se puede leer la tarjeta 

  if (!rfid.PICC_ReadCardSerial()) 

    return; 

  

  // Imprimir el UID de la tarjeta escaneada 

  Serial.print("UID de la tarjeta: "); 

  for (byte i = 0; i < rfid.uid.size; i++) { 

    Serial.print(rfid.uid.uidByte[i] < 0x10 ? " 0" : " "); 

    Serial.print(rfid.uid.uidByte[i], HEX); 

  } 

  Serial.println(); 

  

  // Verificar si el UID coincide con alguno de los permitidos 

  if (compararUID(rfid.uid.uidByte, uidPermitida1) || compararUID(rfid.uid.uidByte, uidPermitida2)) { 

    Serial.println("Acceso permitido."); 

     

    lcd.clear(); 

     

    lcd.print("Escribe algo"); 

  if (Serial.available() > 0) { 

  String input = Serial.readString(); 

   

  Serial.print("Recibiste: "); 

  Serial.println(input); 

  

    lcd.setCursor(0,0); 

    lcd.print("En Proceso"); 

     

    servoPuerta.write(160); 

    delay(1000); 

    servoPuerta.write(0); 

    }     

  

  } else { 

    Serial.println("Acceso denegado."); 

  } 

  

  // Detener la lectura de la tarjeta 

  rfid.PICC_HaltA(); 

} 

 // Función para comparar dos UIDs byte a byte 

bool compararUID(byte *uidLeida, byte *uidPermitida) { 

  for (byte i = 0; i < 4; i++) {  // Recorre los 4 bytes del UID 

    if (uidLeida[i] != uidPermitida[i]) {  // Si algún byte es diferente 

      return false;  // Los UIDs no coinciden 

    } 

  } 

  return true;  // Si todos los bytes coinciden, los UIDs son iguales 

} 
