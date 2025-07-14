---
title: Data Spaces
---

Data spaces (DS) refer to structured and managed environments where data from various sources is securely stored, shared, and utilized for AI and robotics applications within smart and sustainable cities. These **data spaces are the project's core technology**, enabling participants to **access** and leverage **high-quality data for testing, experimentation, and validation of AI technologies**.

Data spaces support **interoperability**, ensuring that data from different sources can be combined and used while **complying with regulations** such as the GDPR and other EU directives. They provide the necessary infrastructure for managing data in a way that supports ethical considerations, cybersecurity, and the broader goals of creating a more digital and sustainable urban environment.

??? info "More information"
    - [**Data Space Support Center (DSSC):**](https://dssc.eu/)  
        - [Data Space Definition](https://dssc.eu/space/BVE2/1071251613/Introduction+-+Key+Concepts+of+Data+Spaces#1.-What-is-a-data-space?)
    - [**Data Spaces for Smart Cities (DS4SCC):**](https://www.ds4sscc.eu/)
        - [Interactive portal for building data spaces in Smart Communities](https://inventory.ds4sscc.eu/)

![data_space](img/basic_architectural_concepts_ids.png)

## Minimum Viable Data Space

A Minimum Viable Data Space (MVDS) is a basic configuration of a data space that includes only the essential components required (*Trust Framework* and *Connector*) to ensure interoperability and enable the secure and sovereign exchange of information between organisations. Its minimal approach aims to reduce initial complexity, support technological adoption, and provide a way to test the ecosystem’s functionality before scaling to more comprehensive solutions.

- **Trust Anchor (TA)**: Responsible for managing trust in the data space. It is the manager of the identities of the different elements of the data space and of managing the trust in them. At least one TA shall exist in the data space, managed by the organization in charge of the data space. 

    !!! Tip "More details"
        Overview of open-source trust frameworks: [here](../../documentation/trust_frameworks/index.md)

- **Data Space Connector (DSC)**: Responsible for managing the communication between the different elements of the data space. It oversees managing authentication, authorization and data access control. There must be at least two DSCs, one per organization, to be able to affirm that a data space exists.

    !!! Tip "More details"
        Overview of open-source data spaces connectors: [here](../../documentation/data_space_connectors/index.md)

This type of data space serves as a testing environment that facilitates the validation of data exchange models and a gradual migration from existing systems. Thanks to its streamlined structure, the MVDS is especially well-suited for demonstrations, pilots, or early implementation stages in collaborative settings where data sharing is expected to be trustworthy and controlled.

### Interoperability Levels

Following the interoperability levels (0, 1 and 2; section [Interoperability Levels](../interoperability.md#interoperability-levels)), the MVDS aims to provide the minimal set of tools required to progress from interoperability level 1 to level 2.

## CitCom.ai Data Space

Data spaces are pivotal in accelerating innovation by facilitating collaboration among different stakeholders. They offer a **secure** and compliant framework for data exchange, ensuring that the **AI solutions developed within the project are both reliable and aligned with European standards**.

!!! Warning
    **CitCom.ai uses [FIWARE technology](https://github.com/FIWARE/data-space-connector/tree/main) for its data spaces**, although in the future it will evolve to a combination of Fiware and [Eclipse technology](https://github.com/eclipse-edc/).

The initial adoption of the FIWARE Data Space connector (DSC) within the CitCom.ai project is a strategic decision that aligns with the [Data Space Business Alliance](https://data-spaces-business-alliance.eu/) (DSBA) and the [Data Spaces for Smart Cities (DS4SCC)](https://www.ds4sscc.eu/) recommendations, ensuring a robust and interoperable framework for data exchange across Testing and Experimentation Facilities (TEFs).

