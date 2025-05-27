---
title: Center Denmark
---

## Overview

_Provide a brief overview of the TEF Site, including its location, objectives, and role within the CitCom.ai ecosystem. Describe any relevant background information that stakeholders might find helpful._

Example:

The [TEF Site Name] is located in [City, Country] and is dedicated to advancing research and development in [relevant domain]. This site is equipped with state-of-the-art infrastructure and is a key site in the CitCom.ai project, facilitating collaboration between cities and comunities, industrial partners (AI innovators) and research institutions.

## Services Offered

The following services provided by Center Denmark as part of the CitCom.ai ecosystem are focused on enabling **secure, structured, and value-driven access to utility data**. Whether for collaboration between utilities and service providers, academic research, or early-stage experimentation, the services make it possible to work with real-world data from electricity, water, heating, gas, and PtX – all compliant with GDPR and tailored to different needs.

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

## Relevant datasets of the site

Describe the relevant datasets available at the site

- **Dataset_1**: [Description of the data set and link to Data Catalog: eg https://citcomai-hub.github.io/data_catalog/metadata_datasets/south_spain_valencia/]
- **Dataset_2**: [Description of the data set and link to Data Catalog: eg https://citcomai-hub.github.io/data_catalog/metadata_datasets/south_spain_valencia/]
- **Dataset_3**: [Description of the data set and link to Data Catalog: eg https://citcomai-hub.github.io/data_catalog/metadata_datasets/south_spain_valencia/]

## Key Stakeholders and Partners

Provide a list of the key stakeholders and partners involved in the TEF Site. Include any academic institutions, industry collaborators, and other stakeholders.

- **Stakeholder 1**: [Name and description of the stakeholder, e.g., university, research institute, industry partner]
- **Stakeholder 2**: [Description]
- **Stakeholder 3**: [Description]

## Contact Information

Provide contact details for those responsible for the TEF Site or who can provide more information to collaborators or users.

- **Site Coordinator**: [Name and contact details]
- **Technical Support**: [Name and contact details]
- **General Inquiries**: [Name and contact details]

## Additional Information

Any other relevant information that might be useful to collaborators or developers working with the TEF Site, such as specific protocols, access instructions, or unique capabilities.

Example:
The TEF Site offers unique capabilities in [specific field], and it is open to collaboration with other EU projects in the area of [related field].

## Documentation and Resources

Link to any relevant documentation or resources, such as technical specifications, API documentation, or guides for using services at the TEF Site.

- [Documentation Link 1](#)
- [Documentation Link 2](#)

---

!!! info
    This page is part of the documentation hub for the CitCom.ai project. Please ensure that the information is up-to-date and accurate.
