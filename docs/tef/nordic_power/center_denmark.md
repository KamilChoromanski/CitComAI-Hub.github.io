---
title: Center Denmark
---

## Overview

Center Denmark is located in Fredericia, Denmark, and serves as a national hub for digitalization across the utility sector. As a TEF Site in the CitCom.ai project, Center Denmark focuses on enabling AI-driven innovation by providing access to high-quality, real-world data from electricity, water, and heating systems (including gas and PtX where relevant).

Center Denmark provides secure and structured data infrastructure that supports collaboration between utilities, AI innovators, and researchers. The site plays a central role in bridging the gap between data availability and AI application by making validated utility datasets accessible for development, testing, and scaling of smart solutions that support the green transition.

In short: Center Denmark provides access to high-quality utility data through a dedicated data portal, offering both downloadable datasets and API-based access – ready for use in AI, analytics, and research.

## Services Offered

The following services provided by Center Denmark as part of the CitCom.ai ecosystem are focused on enabling secure, structured, and value-driven access to utility data. Whether for collaboration between utilities and service providers, academic research, or early-stage experimentation, the services make it possible to work with real-world data from electricity, water, heating, gas, and PtX – all compliant with GDPR and tailored to different needs.

### Service 1: Access to Utility Data for Co-developed Digital Services  
This service supports collaboration between utility companies and service providers by granting access to structured and enriched utility data. The data is typically used for AI and analytics purposes such as forecasting, predictive maintenance, or grid planning.  
Center Denmark manages the entire data process – from collection and cleansing to enrichment and delivery – ensuring high data quality and full GDPR compliance.

📄 Documentation: *Link to documentation will be inserted later*

---

### Service 2: Build-Your-Own Utility Dataset  
This modular service enables data consumers (e.g. researchers, startups) to configure and request customized datasets based on available utility data.  
Users select a base dataset (electricity, water, heating) and can combine it with open data sources such as weather data, building information (BBR), electricity spot prices, or demographic data. The final dataset is delivered at the desired processing level (e.g. anonymised or pseudonymised), and pricing depends on the complexity and scope of the request.

📄 Documentation: *Link to documentation will be inserted later*

---

### Service 3: Open Utility Datasets  
This service provides free access to pre-approved, anonymised datasets via Center Denmark’s data portal.  
It is designed for early-stage experimentation, prototyping, education, and exploratory analysis. The datasets are ready to use and can be combined with publicly available data sources such as weather and building data, without any need for customization.

📄 Documentation: *Link to documentation will be inserted later*


## Infrastructure Components

Describe the key infrastructure components available at the TEF Site, including data platforms, local digital twins, specific hardware, IoT platforms, or any other relevant technologies.

- **Data Platforms**: [Description of the data platforms available]
- **Local Digital Twins**: [Details about any local digital twin infrastructure]
- **Specific Hardware**: [Details about specialized hardware available, such as sensors, servers, etc.]
- **IoT Platforms**: [Information about IoT systems or platforms in use at the site]
- **Visualization platforms**: [Information about large scale visualisation components]
- **Other**: [Any other relevant infrastructure to showcase]

<table>
  <tr>
    <th colspan="2" style="text-align: center;">Specifications</th>
  </tr>
  <tr>
    <td><strong>Data Broker<strong></td>
    <td>
      {{ config.extra.labels.data_brokers.kafka }}<br>
      <strong>- API:</strong> {{ config.extra.labels.api_brokers.custom }}<br>
      <strong>- Version:</strong>&lt;no_specified\>
    </td>
  </tr>
  <tr>
    <td><strong>Data Source<strong></td>
    <td>Nifi</td>
  </tr>
  <tr>
    <td><strong>IdM &amp; Auth<strong></td>
    <td>&lt;no_specified\></td>
  </tr>
  <tr>
    <td><strong>Data Publication<strong></td>
    <td>MQTT, AMQP</td>
  </tr>
</table>

### Architecture

Provide a high-level overview of the architecture of the TEF Site, including the key components and technologies used. Include any relevant diagrams or visualizations to help stakeholders understand the infrastructure.

![center_denmark_arch](./img/center_denmark-arch.png)

### European Data Space for Smart Communities (DS4SSCC)

{{ config.extra.labels.ds4ssc_compliant.yes_comp.data_sources }} {{ config.extra.labels.ds4ssc_compliant.yes_comp.data_broker }} {{ config.extra.labels.ds4ssc_compliant.yes_comp.data_api }} {{ config.extra.labels.ds4ssc_compliant.no_comp.data_idm_auth }} {{ config.extra.labels.ds4ssc_compliant.yes_comp.data_publication }}

![center_denmark_arch-ds4sscc](./img/center_denmark_ds4sscc-arch.svg)

## Available Data Sources

Since Center Denmark builds customised datasets based on specific needs, there are no predefined or fixed datasets available.  
Instead, the platform provides access to a wide range of **trusted data sources** from both utilities and public registers. These sources can be combined and processed to create datasets tailored to individual use cases – whether for AI training, research, forecasting, or infrastructure planning.  

The full overview of available data sources and formats can be explored at [**portal.centerdenmark.com**](https://portal.centerdenmark.com).

| **Category**           | **Examples**                                     | **Description**                                                                 |
|------------------------|--------------------------------------------------|---------------------------------------------------------------------------------|
| **Utility data**       | Electricity, water, heating, gas, PtX            | Consumption and production data from utility partners                          |
| **Grid topology**      | Electricity grids, district heating networks     | Aggregated network structures where permitted                                  |
| **Weather data**       | DMI, local weather stations                      | Temperature, wind speed/direction, solar radiation                             |
| **Building data**      | BBR (Building and Dwelling Register)             | Building type, energy label, heating system, floor area                        |
| **Price data**         | Nord Pool spot prices                            | Hourly electricity market prices                                               |
| **Geospatial references** | DAR, GIS layers                              | Geographical mapping and address-based enrichment                              |
| **Demographic data**   | Statistics Denmark                               | Population, household types, income levels (when permitted and relevant)       |

> All data is processed in compliance with GDPR and access is granted based on appropriate legal agreements with data owners.

## Contact Information

For any inquiries or collaboration opportunities related to the TEF Site at Center Denmark, please contact the relevant team member:

- **Site Coordinator**  
  **Britt Reenberg**  
  Head of PMO  
  📧 brir@centerdenmark.com

- **Technical Support**  
  **Thomas Thue Sørensen**  
  Senior Data Engineer  
  📧 thomas.soerensen@centerdenmark.com

- **General Inquiries**  
  **Andreas Okkels O. Thomsen**  
  Senior Business Developer  
  📧 andreas.thomsen@centerdenmark.com


---

!!! info
    This page is part of the documentation hub for the CitCom.ai project. Please ensure that the information is up-to-date and accurate.
