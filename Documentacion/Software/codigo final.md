#include <Adafruit_MPU6050.h>

#include <Adafruit_Sensor.h>

#include <Wire.h>

#define FORCE_SENSOR_PIN 35 // ESP32 pin GIOP35 (ADC1)

#define FORCE_SENSOR_PIN2 34 // ESP32 pin GIOP34 (ADC1)

Adafruit_MPU6050 mpu;

// Variables para almacenar los valores iniciales de calibración

float initialAccelX = 0.0, initialAccelY = 0.0, initialAccelZ = 0.0;

float initialGyroX = 0.0, initialGyroY = 0.0, initialGyroZ = 0.0;

// Variables para el control de tiempo con millis()

unsigned long lastForceSensorReadTime = 0;

unsigned long lastForceSensorReadTime2 = 0;

unsigned long lastCalibrationTime = 0;

unsigned long lastMpuEventTime = 0;

// Intervalos de tiempo

const unsigned long forceSensorInterval = 1000; // Intervalo para la lectura de los sensores de fuerza

const unsigned long calibrationInterval = 10; // Intervalo durante la calibración

const unsigned long mpuEventInterval = 100; // Intervalo para la lectura del MPU6050

void setup() {

  Serial.begin(115200);

  // Inicialización de MPU6050

  pinMode(4, OUTPUT);

  while (!Serial) delay(10); // Espera hasta que el puerto serie esté disponible

  Serial.println("Adafruit MPU6050 test!");

  if (!mpu.begin()) {

    Serial.println("Failed to find MPU6050 chip");

    while (1) {

      delay(10);

    }

  }

  Serial.println("MPU6050 Found!");

  mpu.setAccelerometerRange(MPU6050_RANGE_8_G);

  mpu.setGyroRange(MPU6050_RANGE_500_DEG);

  mpu.setFilterBandwidth(MPU6050_BAND_5_HZ);

  // Calibración inicial y almacenar los valores base

  Serial.println("Calibrating accelerometer and gyroscope...");

  calibrateMPU();

  Serial.println("Calibration complete.");

  // Configurar el ADC para los sensores de fuerza

  pinMode(FORCE_SENSOR_PIN, INPUT);

  pinMode(FORCE_SENSOR_PIN2, INPUT);

}

void loop() {

  unsigned long currentMillis = millis();

  // Lectura y procesamiento del MPU6050 a intervalos regulares

  if (currentMillis - lastMpuEventTime >= mpuEventInterval) {

    lastMpuEventTime = currentMillis;

    sensors_event_t a, g, temp;

    mpu.getEvent(&a, &g, &temp);

    float deltaAccelX = a.acceleration.x - initialAccelX;

    float deltaAccelY = a.acceleration.y - initialAccelY;

    float deltaAccelZ = a.acceleration.z - initialAccelZ;

    float deltaGyroX = g.gyro.x - initialGyroX;

    float deltaGyroY = g.gyro.y - initialGyroY;

    float deltaGyroZ = g.gyro.z - initialGyroZ;

    Serial.print("Cambio en Aceleracion X: ");

    Serial.print(deltaAccelX);

    Serial.print(", Y: ");

    Serial.print(deltaAccelY);

    Serial.print(", Z: ");

    Serial.print(deltaAccelZ);

    Serial.println(" m/s^2");

    Serial.print("Cambio en Rotacion X: ");

    Serial.print(deltaGyroX);

    Serial.print(", Y: ");

    Serial.print(deltaGyroY);

    Serial.print(", Z: ");

    Serial.print(deltaGyroZ);

    Serial.println(" rad/s");

    Serial.print("Temperatura: ");

    Serial.print(temp.temperature);

    Serial.println(" grados C");

    // Control de salida basado en temperatura

    if (temp.temperature > 31) {

      digitalWrite(4, HIGH);

    } else {

      digitalWrite(4, LOW);

    }

  }

  // Lectura y procesamiento de los sensores de fuerza a intervalos regulares

  if (currentMillis - lastForceSensorReadTime >= forceSensorInterval) {

    lastForceSensorReadTime = currentMillis;

    int analogReading = analogRead(FORCE_SENSOR_PIN);

    Serial.print("The force sensor value = ");

    Serial.print(analogReading);

    if (analogReading < 10) Serial.println(" -> no hay presion");

    else if (analogReading < 200) Serial.println(" -> presion baja");

    else if (analogReading < 500) Serial.println(" -> presion media");

    else if (analogReading < 800) Serial.println(" -> presion alta");

    else Serial.println(" -> presion muy alta");

  }

  if (currentMillis - lastForceSensorReadTime2 >= forceSensorInterval) {

    lastForceSensorReadTime2 = currentMillis;

    int analogReading2 = analogRead(FORCE_SENSOR_PIN2);

    Serial.print("The force sensor2 value = ");

    Serial.print(analogReading2);

    if (analogReading2 < 10) Serial.println(" -> no hay presion");

    else if (analogReading2 < 200) Serial.println(" -> presion baja");

    else if (analogReading2 < 500) Serial.println(" -> presion media");

    else if (analogReading2 < 800) Serial.println(" -> presion alta");

    else Serial.println(" -> presion muy alta");

  }

}

// Función de calibración para el MPU6050 usando millis()

void calibrateMPU() {

  long accelSumX = 0, accelSumY = 0, accelSumZ = 0;

  long gyroSumX = 0, gyroSumY = 0, gyroSumZ = 0;

  unsigned long calibrationStartTime = millis();

  int calibrationSamples = 100;

  int sampleCount = 0;

  while (sampleCount < calibrationSamples) {

    unsigned long currentMillis = millis();

    if (currentMillis - lastCalibrationTime >= calibrationInterval) {

      lastCalibrationTime = currentMillis;

      sensors_event_t a, g, temp;

      mpu.getEvent(&a, &g, &temp);

      accelSumX += a.acceleration.x;

      accelSumY += a.acceleration.y;

      accelSumZ += a.acceleration.z;

      gyroSumX += g.gyro.x;

      gyroSumY += g.gyro.y;

      gyroSumZ += g.gyro.z;

      sampleCount++;

    }

  }

  initialAccelX = accelSumX / (float)calibrationSamples;

  initialAccelY = accelSumY / (float)calibrationSamples;

  initialAccelZ = accelSumZ / (float)calibrationSamples;

  initialGyroX = gyroSumX / (float)calibrationSamples;

  initialGyroY = gyroSumY / (float)calibrationSamples;

  initialGyroZ = gyroSumZ / (float)calibrationSamples;

}

