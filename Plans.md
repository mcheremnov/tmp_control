System Overview
A microcontroller reads the outdoor temperature and adjusts an indoor valve (e.g., heating/cooling water flow) to maintain a target indoor temperature.

Hardware Components
Microcontroller
	∙	Recommended: ESP32 or Arduino Mega
	∙	ESP32 is preferred if you want Wi-Fi (remote monitoring/control)
Sensors
	∙	Outdoor temp: DS18B20 (waterproof, -55°C to +125°C) or DHT22
	∙	Indoor temp: DHT22 or BME280 (also reads humidity/pressure)
Valve Actuator
	∙	Servo motor (for proportional control) — e.g., MG996R
	∙	Or a motorized ball valve (12V, on/off or proportional)
Power Supply
	∙	5V for microcontroller + sensors
	∙	12V for motorized valve (with a buck converter or separate rail)
Optional
	∙	OLED display (show temps + valve %)
	∙	Relay module (if using a solenoid valve)
	∙	MOSFET driver (for motorized valves)

Wiring Overview

DS18B20 (outdoor) ──► GPIO (1-Wire)  ─┐
DHT22   (indoor)  ──► GPIO (digital) ─┤── ESP32 / Arduino
Servo / Valve     ──► GPIO (PWM)     ─┘
OLED (optional)   ──► I2C (SDA/SCL)


Control Logic

1. Read outdoor temp (T_out)
2. Read indoor temp  (T_in)
3. Compute error = T_target − T_in
4. Adjust valve position based on error:
     - If T_in < T_target → open valve more  (more heat)
     - If T_in > T_target → close valve more (less heat)
     - Use PID or simple threshold logic
5. Limit valve to 0–100% range
6. Repeat every N seconds


Control Strategy Options



|Strategy          |Complexity|Best For            |
|------------------|----------|--------------------|
|On/Off (bang-bang)|Simple    |Basic systems       |
|Proportional (P)  |Medium    |Smoother control    |
|PID               |Advanced  |Precise, stable temp|


	- []	Choose your valve type — servo (proportional) vs. motorized ball valve (on/off or proportional)
	- []	Pick your MCU — ESP32 if you want app/web control, Arduino if keeping it simple
	- []	Decide on control strategy — PID for comfort, simple threshold for ease
	- []	Prototype on breadboard first, then move to PCB or enclosure

Example PID Pseudocode
``` c
float Kp = 2.0, Ki = 0.5, Kd = 1.0;
float error, integral = 0, prev_error = 0;

void loop() {
  float T_in = readIndoorTemp();
  float T_out = readOutdoorTemp();

  error = T_target - T_in;
  integral += error * dt;
  float derivative = (error - prev_error) / dt;

  float valve_pos = Kp*error + Ki*integral + Kd*derivative;
  valve_pos = constrain(valve_pos, 0, 100); // 0–100%

  setValve(valve_pos);
  prev_error = error;
  delay(1000); // 1s loop
}
```

ESP32 GPIO assignments:
	∙	GPIO 4 → DS18B20 DATA (outdoor sensor, 1-Wire)
	∙	GPIO 15 → DHT22 DATA (indoor sensor)
	∙	GPIO 21/22 → OLED SDA/SCL (I2C, dashed = optional)
	∙	GPIO 19 → MOSFET gate → Motorized valve (PWM)
Critical wiring notes:
	∙	The DS18B20 needs a 4.7kΩ pullup resistor between DATA and 3.3V
	∙	The DHT22 needs a 10kΩ pullup resistor between DATA and 3.3V
	∙	The valve runs on 12V from an external PSU — never power it from the ESP32 directly
	∙	The MOSFET (IRLZ44N) acts as the switch between the ESP32 and the valve, with a flyback diode to protect against motor back-EMF