# Super Node: South | TEF Node: Italy

## Site: UpTown | AirQuality 

| ATTRIBUTE    | TYPE     | UNITS (SI)   | DESCRIPTION/COMMENTS                      |
|:-------------|:---------|:-------------|:------------------------------------------|
| id           | Text     |           | Sensor identifier                         |
| temperature  | Number   | K            | Ambient temperature                       |
| humidity     | Number   | %            | Relative humidity                         |
| co           | Number   | mg/m³        | Carbon monoxide concentration             |
| nh3          | Number   | µg/m³        | Ammonia concentration                     |
| no           | Number   | µg/m³        | Nitric oxide concentration                |
| no2          | Number   | µg/m³        | Nitrogen dioxide concentration            |
| o3           | Number   | µg/m³        | Ozone concentration                       |
| pm25         | Number   | µg/m³        | Particulate matter < 2.5 µm concentration |
| pm10         | Number   | µg/m³        | Particulate matter < 10 µm concentration  |
| so2          | Number   | µg/m³        | Sulfur dioxide concentration              |
| indoor       | Boolean  |           | True if the sensor is located indoors     |
| manufacturer | Text     |           | Sensor manufacturer name                  |
| sensorName   | Text     |           | Human-readable sensor name                |
| location     | geo:json |           | Geographical coordinates of the sensor    |
| timestamp    | Number   |           | Unix timestamp of the measurement         |

## Site: UpTown | Biodiversity 

| ATTRIBUTE                      | TYPE   | UNITS (SI)   | DESCRIPTION/COMMENTS                        |
|:-------------------------------|:-------|:-------------|:--------------------------------------------|
| id                             | Text   |           | Sensor identifier                           |
| timestamp                      | Number |           | Unix timestamp of the measurement           |
| device_hostname                | Text   |           | Device hostname                             |
| firmware_version               | Text   |           | Current firmware version                    |
| state                          | Text   |           | Reported high-level state of the device     |
| activity_state                 | Text   |           | Current activity state (e.g., active, idle) |
| device_temperature             | Number | °C          | Internal device temperature                 |
| battery_voltage                | Number | V            | Battery voltage                             |
| battery_level                  | Number | %            | Battery charge level                        |
| battery_power_input            | Text   |           | Raw battery input power data                |
| last_ping                      | Number |           | Timestamp of last device ping               |
| last_audio_record              | Number |           | Timestamp of last audio recording           |
| last_detection_ts              | Number |           | Timestamp of last detection event           |
| last_detection_scientific_name | Text   |           | Scientific name of last detected species    |
| last_detection_common_name_en  | Text   |           | Common name (EN) of last detected species   |
| last_detection_common_name_it  | Text   |           | Common name (IT) of last detected species   |
| last_detection_confidence      | Number |           | Confidence score of last detection          |

## Site: UpTown | Energy distribution 

| ATTRIBUTE                    | TYPE     | UNITS (SI)   | DESCRIPTION/COMMENTS                                         |
|:-----------------------------|:---------|:-------------|:-------------------------------------------------------------|
| id                           | Text     |           | Sensor identifier                                            |
| timestamp                    | Number   |           | Unix timestamp of the measurement                            |
| user_identifier              | Text     |           | Anonymized unique identifier for the user                    |
| pod_identifier               | Number   |           | Anonymized identifier of the heating POD unit                |
| pod_coordinates              | geo:json |           | Geographical coordinates of the POD                          |
| user_type                    | Text     |           | Category or role of the user (e.g., residential, commercial) |
| external_temperature         | Number   | °C           | Outdoor air temperature                                      |
| heating_temperature_delivery | Number   | °C           | Temperature of the fluid delivered to the heating system     |
| heating_temperature_return   | Number   | °C           | Temperature of the fluid returning from the heating system   |
| heating_volume               | Number   | m³/h         | Volume flow rate of heating fluid                            |
| heating_power                | Number   | kW           | Thermal power delivered for heating                          |
| energy_consumption           | Number   | kWh          | Cumulative heating energy consumed                           |

