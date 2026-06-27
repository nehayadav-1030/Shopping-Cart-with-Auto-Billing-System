#include <SPI.h>
#include <MFRC522.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define SS_PIN 10
#define RST_PIN 9
#define BUZZER 8

MFRC522 rfid(SS_PIN, RST_PIN);
LiquidCrystal_I2C lcd(0x27,16,2);

int total = 0;

// Replace these with your RFID Tag IDs
byte item1[4] = {0xA3,0xB2,0xC1,0xD4};
byte item2[4] = {0x11,0x22,0x33,0x44};
byte item3[4] = {0x55,0x66,0x77,0x88};

bool compareUID(byte a[], byte b[])
{
  for(int i=0;i<4;i++)
  {
    if(a[i]!=b[i])
      return false;
  }
  return true;
}

void setup()
{
  Serial.begin(9600);
  SPI.begin();
  rfid.PCD_Init();

  lcd.init();
  lcd.backlight();

  pinMode(BUZZER,OUTPUT);

  lcd.setCursor(0,0);
  lcd.print("SMART CART");
  lcd.setCursor(0,1);
  lcd.print("Auto Billing");

  delay(2000);
  lcd.clear();
}

void loop()
{
  lcd.setCursor(0,0);
  lcd.print("Scan Product");

  if(!rfid.PICC_IsNewCardPresent())
    return;

  if(!rfid.PICC_ReadCardSerial())
    return;

  digitalWrite(BUZZER,HIGH);
  delay(200);
  digitalWrite(BUZZER,LOW);

  if(compareUID(rfid.uid.uidByte,item1))
  {
      total += 50;
      lcd.clear();
      lcd.print("Milk Rs.50");
  }

  else if(compareUID(rfid.uid.uidByte,item2))
  {
      total += 120;
      lcd.clear();
      lcd.print("Rice Rs.120");
  }

  else if(compareUID(rfid.uid.uidByte,item3))
  {
      total += 30;
      lcd.clear();
      lcd.print("Bread Rs.30");
  }

  else
  {
      lcd.clear();
      lcd.print("Unknown Item");
  }

  delay(1500);

  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Total:");
  lcd.print(total);
  lcd.print(" Rs");

  delay(2000);

  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();
}
