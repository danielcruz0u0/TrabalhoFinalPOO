# # MonitorizaPT

MonitorizaPT is a Java 17 desktop project made for the final OOP assignment at IPLuso. The program simulates some environmental sensors (temperature, humidity and air quality). The sensors generate dummy values, send them through MQTT, receive simple commands, and the data is shown in a Swing dashboard. The idea is to follow what was asked in the assignment, like the fixed reading interval, alerts, MQTT topics and the basic monitoring interface.

## Features

- Seven sensors with fixed locations in Portugal, each one with its own type.
- Reading loop uses `while(true)` and `Thread.sleep(3333)` as requested in the brief.
- Sometimes it produces higher values on purpose to test alerts.
- Alert limits:
  - temperature > 30°C
  - humidity > 80%
  - air quality index > 50
- JSON payload contains the field `hash_validacao`, calculated from the payload without spaces.
- MQTT topics:
  - data → `envira/pt/sensores/dados/<Localizacao>`
  - commands → `envira/pt/sensores/comandos/<Localizacao>`
- Swing UI with the title `"MonitorizaPT - Sensores Ambientais v1.0"`, showing MQTT connection state, a table with sensors, control buttons and a log area.

## Project Structure

- `pt.monitorizapt.domain` — interface, abstract base class, enums and record for sensor data  
- `pt.monitorizapt.sensors` — concrete classes for temperature, humidity and air quality  
- `pt.monitorizapt.mqtt` — MQTT client manager (publish and subscribe)  
- `pt.monitorizapt.service` — `SensorController` controls activation, loop and commands  
- `pt.monitorizapt.ui` — Swing table model and main window  
- `pt.monitorizapt.app.MonitorizaPTApplication` — main entry point of the application

## Prerequisites

- Java 17+
- Maven
- Internet connection for the MQTT broker

## Default broker

tcp://172.237.103.61:1883

It can be changed with:

-Dmonitorizapt.broker=<url>

(for example, the IP from the assignment)

## Building & Running

mvn clean package
mvn exec:java -Dexec.mainClass=pt.monitorizapt.app.MonitorizaPTApplication
When the program starts, the Swing window opens at 800x600 with the required title. The user can choose a location, change the interval if needed (default = 3333 ms), start or stop the sensor, test the broker connection and clear the logs. The log area shows timestamps, values sent and MQTT commands received.

MQTT Payload Schema
{
  "campus": "Lisboa - Campus IPLuso",
  "sensor": "PT-SENSOR-LISBOA_CAMPUS_IPLUSO",
  "ID Unico": "PT-SENSOR-LISBOA_CAMPUS_IPLUSO",
  "Owner": "Update_Name_And_Number",
  "tipo": "temperatura",
  "valor": 24.73,
  "unidade": "Celsius",
  "alerta": false,
  "timestamp": 1734947823000,
  "hash_validacao": "<value>"
}

Commands are sent to:
envira/pt/sensores/comandos/<Localizacao>
Example:

{
  "acao": "ATIVAR",
  "intervalo": 3333
}
The field acao accepts ATIVAR or DESATIVAR.

## Sensors & Locations
Location	Type	MQTT Topic
Lisboa - Campus IPLuso	Temperature	envira/pt/sensores/dados/Lisboa_Campus_IPLuso
Lisboa - Baixa	Humidity	envira/pt/sensores/dados/Lisboa_Baixa
Porto - Matosinhos	Air Quality	envira/pt/sensores/dados/Porto_Matosinhos
Coimbra - Centro	Temperature	envira/pt/sensores/dados/Coimbra_Centro
Faro - Marina	Humidity	envira/pt/sensores/dados/Faro_Marina
Braga - Sameiro	Air Quality	envira/pt/sensores/dados/Braga_Sameiro
Évora - Universidade	Temperature	envira/pt/sensores/dados/Evora_Universidade

All locations also subscribe to the command topics, so sensors can be activated remotely.

## Testing Tips
You can test using MQTT Explorer or mosquitto_sub

Subscribe to:

envira/pt/sensores/dados/#
Example command:

mosquitto_pub -t envira/pt/sensores/comandos/Lisboa_Campus_IPLuso -m '{"acao":"ATIVAR","intervalo":3333}'
To use the broker from the assignment:

-Dmonitorizapt.broker=tcp://172.237.103.61:1883
