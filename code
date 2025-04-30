**Code to Check The Monitor Readings**

#include <PZEM004Tv30.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <WiFi.h>
#include <ThingSpeak.h>
#include <DHT.h>
//#include <hd44780.h>

#define relay 25
#define DHTPIN 32 // GPIO pin connected to the DHT sensor
#define DHTTYPE DHT11 // DHT sensor type (DHT11, DHT22, AM2302, etc.)
DHT dht(DHTPIN, DHTTYPE);

// WiFi credentials
const char *ssid = "vivo Y21";
const char *password = "123456789";

// ThingSpeak channel settings
const char *apiKey = "GVW7VRT5A8RNTT5G";
const unsigned long channelId = 2767527;
const char* server = "api.thingspeak.com";

WiFiClient client;
LiquidCrystal_I2C lcd(0x27,20,4); // I2C address 0x27, 20 columns and 4 rows

// Define PZEM pins and serial interface based on the microcontroller
#define PZEM_RX_PIN 16
#define PZEM_TX_PIN 17
#define PZEM_SERIAL Serial2
PZEM004Tv30 pzem(PZEM_SERIAL, PZEM_RX_PIN, PZEM_TX_PIN);

void setup() {
    Serial.begin(115200);
    pinMode(relay, OUTPUT);
    dht.begin();

    // Initialize LCD display
    lcd.begin(); // Correctly specify the LCD dimensions
    lcd.backlight();
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Connecting to WiFi...");

    // Connect to WiFi
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(5000);
        lcd.print(".");
    }
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("WiFi connected");

    // Initialize ThingSpeak
    ThingSpeak.begin(client);
}

void loop() {
    float humidity = dht.readHumidity();
    float temperature = dht.readTemperature();

    if (isnan(humidity) || isnan(temperature)) {
        Serial.println("Failed to read from DHT sensor!");
        return;
    }

    Serial.print("Humidity: ");
    Serial.print(humidity);
    Serial.print(" %\t");
    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.println(" °C");

    float voltage = pzem.voltage();
    float current = pzem.current();
    float power = pzem.power();
    float energy = pzem.energy();
    float frequency = pzem.frequency();
    float pf = pzem.pf();

    if (voltage >= 200) {
        digitalWrite(relay, LOW);
    } else {
        digitalWrite(relay, HIGH);
    }

    // Check if the data is valid
    if (!isnan(voltage) && !isnan(current) && !isnan(power) && !isnan(energy) && !isnan(frequency) && !isnan(pf)) {
        Serial.print("Voltage: "); Serial.print(voltage); Serial.println("V");
        Serial.print("Current: "); Serial.print(current); Serial.println("A");
        Serial.print("Power: "); Serial.print(power); Serial.println("W");
        Serial.print("Energy: "); Serial.print(energy); Serial.println("kWh");
        Serial.print("Frequency: "); Serial.print(frequency); Serial.println("Hz");
        Serial.print("PF: "); Serial.println(pf);

        // Display data on LCD
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print(" Volt:");
        lcd.print(voltage);
        lcd.print("V");
        lcd.setCursor(0, 1);
        lcd.print("Cur:");
        lcd.print(current);
        lcd.print("A ");
        lcd.print("Hum:");
        lcd.print(humidity);
        lcd.print("%");
        lcd.setCursor(0, 2);
        lcd.print("Pow:");
        lcd.print(power);
        lcd.print("W  ");
        lcd.print("PF:");
        lcd.print(pf);
        lcd.setCursor(0, 3);
        lcd.print("Temp:");
        lcd.print(temperature);
        lcd.print("C");


        // Send data to ThingSpeak
        ThingSpeak.setField(1, voltage);
        ThingSpeak.setField(4, current);
        ThingSpeak.setField(2, humidity);
        ThingSpeak.setField(5, energy);
        ThingSpeak.setField(6, frequency);
        ThingSpeak.setField(7, pf);
        ThingSpeak.setField(3, power);
        ThingSpeak.writeFields(channelId, apiKey);

    } else {
        Serial.println("Error reading data from PZEM");
    }

    delay(2000); // Delay before next loop iteration
}
