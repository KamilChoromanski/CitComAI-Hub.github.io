# Super Node: North | TEF Node: Center Denmark

## Site: Center Denmark | District Heating Data 

| ATTRIBUTE   | TYPE   | UNITS (SI)   | DESCRIPTION/COMMENTS   |
|-------------|--------|--------------|------------------------|
| id                   | INTEGER  |            | Identifier                                                                             |
| timestamp            | DATETIME |            | Date and time are shown in UTC in ISO 8601 format. Timestamps are represented in full hours, i.e., minutes and seconds are 00. |
| location             | STRING   |            | Address – following DAR (Danish Address Register)                                     |
| energy               | DOUBLE   | MWh        | The heating consumption of the district heating meter water supply.                   |
| energy_computed      | DOUBLE   | MWh        | Computed heat energy consumption based on the flow and temperature difference. Coefficients are determined for the water at 60 Degrees Celsius. |
| flow                 | DOUBLE   | m³/h       | The flow rate of the district heating meter water supply.                             |
| forward_temperature  | DOUBLE   | °C         | The forward temperature of the district heating meter water supply.                   |
| return_temperature   | DOUBLE   | °C         | Return temperature     

## Site: Center Denmark | Water Data 

| ATTRIBUTE   | TYPE     | UNITS (SI)   | DESCRIPTION/COMMENTS                                                                                                           |
|:------------|:---------|:-------------|:-------------------------------------------------------------------------------------------------------------------------------|
| id          | INTEGER  |           | Identifier                                                                                                                     |
| timestamp   | DATETIME |           | Date and time are shown in UTC in ISO 8601 format. Timestamps are represented in full hours, i.e., minutes and seconds are 00. |
| location    | STRING   |           | Adress - following DAR (Danish Adress Register)                                                                                |
| volumen     | DOUBLE   | M3           | Unit for Volumen                                                                                                               |
| flow        | DOUBLE   | m³/h         | The flow rate of the water supply.                                                                                             |

## Site: Center Denmark | Electricity Data 

| ATTRIBUTE   | TYPE     | UNITS (SI)   | DESCRIPTION/COMMENTS                                                                                                           |
|:------------|:---------|:-------------|:-------------------------------------------------------------------------------------------------------------------------------|
| id          | INTEGER  |           | Identifier                                                                                                                     |
| timestamp   | DATETIME |           | Date and time are shown in UTC in ISO 8601 format. Timestamps are represented in full hours, i.e., minutes and seconds are 00. |
| location    | STRING   |           | Adress - following DAR (Danish Adress Register)                                                                                |
| value       | DOUBLE   | kWh          | The actual value of the reading.                                                                                               |
