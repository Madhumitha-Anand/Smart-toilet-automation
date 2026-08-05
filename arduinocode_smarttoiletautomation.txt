#include <LiquidCrystal_I2C.h>
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT11
#define GAS_SENSOR 34
#define IR_SENSOR 27
#define RELAY_PIN 26
#define FAN_PIN 18

DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal_I2C lcd(0x27,16,2);

int irCount = 0;
bool pumpRunning = false;

void setup()
{
    Serial.begin(115200);

    pinMode(GAS_SENSOR, INPUT);
    pinMode(IR_SENSOR, INPUT);
    pinMode(RELAY_PIN, OUTPUT);
    pinMode(FAN_PIN, OUTPUT);

    dht.begin();

    lcd.init();
    lcd.backlight();
    lcd.print("SMART WASHROOM");
}

void loop()
{
    float temperature = dht.readTemperature();
    int gasValue = digitalRead(GAS_SENSOR);
    int irValue = digitalRead(IR_SENSOR);

    if(gasValue == LOW || temperature > 38)
    {
        digitalWrite(FAN_PIN, HIGH);
    }
    else
    {
        digitalWrite(FAN_PIN, LOW);
    }

    if(irValue == LOW)
    {
        irCount++;
    }

    if(irCount % 5 == 0)
    {
        digitalWrite(RELAY_PIN, LOW);
        delay(5000);
        digitalWrite(RELAY_PIN, HIGH);
    }

    lcd.setCursor(0,0);
    lcd.print("Temp:");
    lcd.print(temperature);

    delay(300);
}

#include <LiquidCrystal_I2C.h>
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT11

#define GAS_SENSOR 34
#define IR_SENSOR 27

#define RELAY_PIN 26
#define FAN_PIN 18

DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal_I2C lcd(0x27, 16, 2);

float temperature = 0;
int gasValue = 0;
int irValue = 0;

bool pumpRunning = false;
unsigned long pumpStartTime = 0;

int irCount = 0;
bool lastIRState = HIGH;

void setup()
{
    Serial.begin(115200);

    pinMode(GAS_SENSOR, INPUT);
    pinMode(IR_SENSOR, INPUT);

    pinMode(RELAY_PIN, OUTPUT);
    pinMode(FAN_PIN, OUTPUT);

    digitalWrite(RELAY_PIN, HIGH);
    digitalWrite(FAN_PIN, LOW);

    dht.begin();

    lcd.init();
    lcd.backlight();

    lcd.setCursor(0,0);
    lcd.print("SMART WASHROOM");
    lcd.setCursor(0,1);
    lcd.print("SYSTEM START");

    delay(3000);
    lcd.clear();
}

void loop()
{
    temperature = dht.readTemperature();
    gasValue = digitalRead(GAS_SENSOR);
    irValue = digitalRead(IR_SENSOR);

    bool gasDetected = (gasValue == LOW);
    bool heatDetected = (temperature > 38);

    if (gasDetected || heatDetected)
        digitalWrite(FAN_PIN, HIGH);
    else
        digitalWrite(FAN_PIN, LOW);

    if (irValue == LOW && lastIRState == HIGH)
    {
        irCount++;

        if (irCount % 5 == 0 && !pumpRunning)
        {
            pumpRunning = true;
            pumpStartTime = millis();
            digitalWrite(RELAY_PIN, LOW);

            lcd.clear();
            lcd.setCursor(0,0);
            lcd.print("COUNT:");
            lcd.print(irCount);
            lcd.setCursor(0
