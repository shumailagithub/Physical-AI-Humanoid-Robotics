---
sidebar_position: 3
title: "Chapter 2: Sensors and Data Collection"
---

# Chapter 2: Sensors and Data Collection

In the previous chapter, we explored the fundamental concept of Physical AI and built a simple sensing-processing-acting system. Now, we'll dive deeper into the first critical component of any Physical AI system: sensors and data collection.

## The Foundation: Sensors

Sensors are the eyes, ears, and sensory organs of Physical AI systems. They convert physical phenomena into digital information that AI systems can process. Understanding how to work with sensors is crucial to building effective Physical AI systems.

## Types of Sensors

There are dozens of sensor types, but they all fall into these main categories:

### Environmental Sensors
- **Temperature & Humidity**: Measure atmospheric conditions
- **Light Sensors**: Detect ambient light levels
- **Air Quality**: Measure pollutants, gases, or particulates
- **Barometric Pressure**: Detect atmospheric pressure changes

### Motion & Position Sensors
- **Accelerometers**: Measure acceleration and orientation
- **Gyroscopes**: Detect rotation and angular velocity
- **Magnetometers**: Measure magnetic fields (compass)
- **GPS**: Determine precise location

### Physical Interaction Sensors
- **Touch/Proximity**: Detect nearby objects or touch
- **Pressure**: Measure force applied to surfaces
- **Distance**: Ultrasonic, infrared, or LiDAR for measuring distances

## Your Project: Multi-Sensor Environmental Monitor

Let's build a comprehensive environmental monitoring system that collects data from multiple sensors simultaneously.

### Materials Needed
- ESP32 development board (has built-in WiFi for data transmission)
- DHT22 temperature/humidity sensor
- BMP280 pressure sensor (I2C interface)
- Photoresistor (light sensor)
- Resistors and breadboard

### Complete Sensor Code

```cpp
#include "DHT.h"
#include <Wire.h>
#include <Adafruit_BMP280.h>

#define DHT_PIN 4
#define DHT_TYPE DHT22

DHT dht(DHT_PIN, DHT_TYPE);
Adafruit_BMP280 bmp;

void setup() {
  Serial.begin(115200);
  dht.begin();
  
  if (!bmp.begin()) {
    Serial.println("Could not find a valid BMP280 sensor!");
    while (1);
  }
  
  // Configure BMP280 settings
  bmp.setSampling(Adafruit_BMP280::MODE_NORMAL,
                  Adafruit_BMP280::SAMPLING_X2,
                  Adafruit_BMP280::SAMPLING_X16,
                  Adafruit_BMP280::FILTER_X16,
                  Adafruit_BMP280::STANDBY_MS_500);
}

void loop() {
  // Collect data from all sensors
  float humidity = dht.readHumidity();
  float tempC = dht.readTemperature();
  float tempF = dht.readTemperature(true);
  float pressure = bmp.readPressure() / 100.0F;  // Convert to hPa
  float altitude = bmp.readAltitude(1013.25);    // Altitude in meters
  
  // Read light sensor (analog pin)
  int lightLevel = analogRead(A0);
  
  // Check for valid readings
  if (isnan(humidity) || isnan(tempC)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  
  // Format and print sensor data
  Serial.println("--- Environmental Data ---");
  Serial.print("Temperature: ");
  Serial.print(tempC);
  Serial.println("°C");
  Serial.print("Temperature: ");
  Serial.print(tempF);
  Serial.println("°F");
  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println("%");
  Serial.print("Pressure: ");
  Serial.print(pressure);
  Serial.println(" hPa");
  Serial.print("Approx. Altitude: ");
  Serial.print(altitude);
  Serial.println(" m");
  Serial.print("Light Level: ");
  Serial.println(lightLevel);
  
  // Send to AI processing system (we'll expand on this in later chapters)
  processEnvironmentalData(tempC, humidity, pressure, lightLevel);
  
  delay(2000);  // Wait 2 seconds between readings
}

void processEnvironmentalData(float temp, float humidity, float pressure, int light) {
  // This is where AI logic would go to identify patterns
  // For now, let's implement simple rule-based alerts
  
  if (temp > 30 || temp < 18) {
    Serial.println("ALERT: Temperature out of normal range!");
  }
  
  if (humidity > 80 || humidity < 30) {
    Serial.println("ALERT: Humidity out of normal range!");
  }
  
  if (light < 100) {
    Serial.println("Suggestion: Consider turning on lights");
  }
}
```

## Data Quality and Calibration

Raw sensor data often contains noise and inaccuracies. Key considerations for quality data collection:

1. **Calibration**: Adjust sensor readings to match known standards
2. **Filtering**: Apply mathematical filters to reduce noise
3. **Validation**: Check for realistic value ranges
4. **Sampling Rate**: Balance between responsiveness and data load

### Example: Simple Moving Average Filter

```cpp
class MovingAverageFilter {
  private:
    float* values;
    int size;
    int index;
    float sum;
  
  public:
    MovingAverageFilter(int sz) {
      size = sz;
      values = new float[size];
      index = 0;
      sum = 0;
      
      for(int i = 0; i < size; i++) {
        values[i] = 0;
      }
    }
    
    float addValue(float newValue) {
      sum -= values[index];
      values[index] = newValue;
      sum += values[index];
      
      index = (index + 1) % size;
      
      return sum / size;
    }
};

// Usage example:
MovingAverageFilter tempFilter(5); // 5-point moving average

// In your sensor reading loop:
float filteredTemp = tempFilter.addValue(rawTemperature);
```

## Preparing Data for AI

The data you collect from sensors becomes the foundation for AI processing. For effective AI integration, data should be:

1. **Consistent**: Regular sampling intervals
2. **Clean**: Removed outliers and noise
3. **Labeled**: Contextual information where possible
4. **Accessible**: Stored in formats the AI can process

## Key Takeaways

- Sensors are the primary input mechanism for Physical AI systems
- Multiple sensors can provide rich, multi-dimensional data about environments
- Data quality significantly impacts AI performance
- Proper filtering and preprocessing are essential for good AI results

In the next chapter, we'll implement AI logic that processes this sensor data to recognize patterns, make predictions, and take intelligent actions.