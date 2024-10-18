## Sentinel Synergy: Revolutionizing Aqua Sump Systems with Automated Ventilation and Integrated Self-Diagnostics for Advanced Home Safety
Traditional systems often focus on detecting a single type of gas, lack advanced user interfaces, and operate in isolation from other home systems. Moreover, these devices typically lack self-maintenance features, which can lead to undetected malfunctions over time.

The Sentinel Synergy project addresses this need by integrating industrial-grade features into a household IoT solution, aiming to set a new standard in residential protection.       

## About
Household gas detection systems are often limited to monitoring a single gas, leaving homeowners vulnerable to a wider range of potential hazards. 
Current consumer-grade gas detectors typically feature basic, non-intuitive alarms, such as simple beeps or LED indicators. 
Household gas detectors are usually standalone devices that alert users but do not take automatic action.
Consumer-grade gas detectors generally lack the advanced self-diagnostics and auto-calibration features found in industrial systems. Without these capabilities, household devices are prone to sensor drift, reduced accuracy, and undetected failures, compromising long-term safety.
Addressing these challenges, the Sentinel Synergy project seeks to enhance home safety by integrating multi-gas detection, advanced user interfaces, automatic shutoff, smart ventilation control, and self-diagnostics into a comprehensive IoT-based system.

## Features
<!--List the features of the project as shown below-->
- Automated Ventilation
- Integrated Self-Diagnostics
- Advanced Home Safety Integration
- Energy Efficiency

## Requirements
### HARDWARE REQUIREMENTS
1.SIM800L/SIM900 GSM Module: To handle SMS communication.
2.Relay Module: To control the motor for ventilation.
3.MQ-137 For detecting ammonia (NH3)
4.MQ-136 For detecting H2S 
5.MQ-4 Gas Sensors: For detecting methane (CH4) levels.

### SOFTWARE REQUIREMENTS
1. Arduino Software
2. ESP32 Drivers
3. <SoftwareSerial.h> library
4. <Wire.h> library

Detection Accuracy: 96.7%
Note: These metrics can be customized based on your actual performance evaluations.

## Important Code Segment
```
#include <SoftwareSerial.h>
#include <Wire.h>

// Pin Definitions
#define RELAY_PIN  5
#define MOTOR_ON   HIGH
#define MOTOR_OFF  LOW

#define MQ_137_PIN 34   // Ammonia
#define MQ_136_PIN 35   // H2S
#define MQ_4_PIN   32   // Methane

// Thresholds for gas levels
#define GAS_ALERT_THRESHOLD 30  // 30% PPM Threshold for alert
#define GAS_CRITICAL_THRESHOLD 80  // 80% PPM Threshold for auto ventilation

// GSM
SoftwareSerial sim800(16, 17);  // RX, TX (choose GPIOs)
// Variables for gas sensor readings
float ammonia_level = 0;
float h2s_level = 0;
float methane_level = 0;
// Relay control variables
bool motor_status = false;
bool motor_auto_override = false;
unsigned long motor_stop_time = 0;
// Helper functions
void sendSMS(String message) {
    sim800.println("AT+CMGF=1");    // Set SMS to text mode
    delay(1000);
    sim800.println("AT+CMGS=\"+1234567890\"");  // Replace with the destination phone number
    delay(1000);
    sim800.println(message);   // Message body
    delay(1000);
    sim800.println((char)26);  // ASCII code for CTRL+Z to send SMS
    delay(1000);
}

// Function to read gas levels and return percentage
float readGasSensor(int pin) {
    int analogValue = analogRead(pin);
    return (analogValue / 4095.0) * 100;  // Convert ADC value (0-4095) to percentage
}

// SMS Commands Handler
void processSMSCommand(String command) {
    if (command == "ON" && !motor_auto_override) {
        digitalWrite(RELAY_PIN, MOTOR_ON);
        motor_status = true;
    }
    if (command == "OFF" && !motor_auto_override) {
        digitalWrite(RELAY_PIN, MOTOR_OFF);
        motor_status = false;
    }
}

// Auto motor control based on gas levels
void checkGasLevels() {
    ammonia_level = readGasSensor(MQ_137_PIN);
    h2s_level = readGasSensor(MQ_136_PIN);
    methane_level = readGasSensor(MQ_4_PIN);

    String message = "Gas levels: NH3: " + String(ammonia_level) + "%, H2S: " + String(h2s_level) + "%, CH4: " + String(methane_level) + "%";

    // Send alert if levels are above threshold
    if (ammonia_level > GAS_ALERT_THRESHOLD || h2s_level > GAS_ALERT_THRESHOLD || methane_level > GAS_ALERT_THRESHOLD) {
        message += "\nAlert! Gas levels above 30%.";
        sendSMS(message);
    }

    // Auto motor activation if levels exceed 80%
    if (ammonia_level > GAS_CRITICAL_THRESHOLD || h2s_level > GAS_CRITICAL_THRESHOLD || methane_level > GAS_CRITICAL_THRESHOLD) {
        motor_auto_override = true;
        digitalWrite(RELAY_PIN, MOTOR_ON);
        motor_status = true;
        sendSMS("Critical! Gas levels above 80%. Motor running.");
    }
}

// Setup and loop
void setup() {
    pinMode(RELAY_PIN, OUTPUT);
    digitalWrite(RELAY_PIN, MOTOR_OFF);  // Start with motor off

    sim800.begin(9600);   // Start communication with GSM module
    Serial.begin(115200); // Start Serial monitor

    // Additional setup code for gas sensors...
}

void loop() {
    checkGasLevels();

    // Periodically check for SMS command
    if (sim800.available()) {
        String smsCommand = sim800.readString();
        smsCommand.trim();
        processSMSCommand(smsCommand);
    }

    // Handle motor override duration when gas is above 80%
    if (motor_auto_override && millis() - motor_stop_time > 600000) {  // 10 minutes
        digitalWrite(RELAY_PIN, MOTOR_OFF);
        motor_auto_override = false;
    }

    delay(5000);  // Check every 5 seconds
}

```


## Results and Impact
What can be accomplised?
- Enhanced home safety
- Automated Safety Response
- Reliable long-term operation
- Reduction in maintenance cost
- Improved user awareness and response 

## Articles published / References

Implementation of Real Time Approach for Early Warning Gas Leakage Detection ,Anisa Qistina Binti Nor Azuwan; Tuan Norjihan Binti Tuan Yaakub;Anees Bt Abdul Aziz
2022 IEEE 12th Symposium on Computer Applications & Industrial Electronics (ISCAIE)Year: 2022 | Conference Paper | Publisher: IEEE

Gas Detection Approaches in Smart Houses,Hashem Almarzooqi;Ibrahim Alzubaidi;Ali Albahrani;Ahmed Almansoori;Maad Shatnawi,2019 International Conference on Computational Science and Computational Intelligence (CSCI)Year: 2019 | Conference Paper | Publisher: IEEE

Gas Detection And Environmental Monitoring Using Raspberry Pi Pico, 2024 IEEE Students Conference on Engineering and Systems (SCES), June 21-23, 2024, Prayagraj, India 






