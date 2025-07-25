---
title: Join the Data Space
---

!!! Warning
    **CitCom.ai uses [FIWARE technology](https://github.com/FIWARE/data-space-connector/tree/main) for its data spaces**, although in the future it will evolve to a combination of Fiware and [Eclipse technology](https://github.com/eclipse-edc/).

The initial adoption of the FIWARE Data Space technology within the CitCom.ai project is a strategic decision that aligns with the [Data Space Business Alliance](https://data-spaces-business-alliance.eu/) (DSBA) and the [Data Spaces for Smart Cities (DS4SCC)](https://www.ds4sscc.eu/) recommendations, ensuring a robust and interoperable framework for data exchange across Testing and Experimentation Facilities (TEFs).

To access to a data space, you mainly need: 

1. **A digital certificate (Verifiable Credential)**: To be able to identify yourself as an organization within the data space. 

2. **A data space connector**: To be able to communicate with the data space.

## VC Issuer

The VC Issuer is a component that issues Verifiable Credentials (VCs) to entities within the data space. These credentials are used to authenticate and authorize access to resources in the data space.

<div class="grid cards" markdown>

-   :material-cog-outline:{ .lg .middle } __Issue your credentials__

    ---

    _Verifiable Credential details configuration._

    [:octicons-arrow-right-24: _More info_](../../documentation/verifiable_credentials/index.md#verifier-credential-issuer)

</div>

## Trust Anchor

The Trust Anchor (TA) is a critical component in the data space ecosystem. It serves as a trusted entity that issues and manages digital certificates (Verifiable Credentials) for organizations and individuals participating in the data space. The TA ensures that all participants are authenticated and authorized to access the resources within the data space.

!!! Tip "More details"
    Overview of open-source trust frameworks: [here](../../documentation/trust_frameworks/index.md)

### Sign Up

Depending on the configuration of the data space, the registration process may vary. Currently, most commonly, you will need to **contact the data space TA administrator** for information on the type of certificate you need and how to provide it so that they can authorize you as an authorized entity in the data space. 

In the future, this process will be automated, and you will be able to do it directly from the data space platform. Using the European digital identity, you will be able to register in the data space in a simple and secure way.

??? warning "TA Endpoint"

    To be part of the CitCom.ai data space, you need to register in the CitCom.ai Trust Anchor. The endpoint for the CitCom.ai Trust Anchor is: `https://xxxx`

## Data Space Connector

The Data Space Connector (DSC) is a software component that is responsible for managing the communication between the different elements of the data space. It oversees managing authentication, authorization, and data access control. Fiware provides a reference implementation of the DSC, which is available in the [Fiware GitHub repository](https://github.com/FIWARE/data-space-connector/tree/main).

**In all cases, you will need to deploy a Data Space Connector** in your organization to be able to share data in the data space. Depending if you want consume or provide data, you will need to deploy a different type of connector.

!!! Tip "More details"
    Overview of open-source data spaces connectors: [here](../../documentation/data_space_connectors/index.md)

<div class="grid cards" markdown>

-   :material-cog-outline:{ .lg .middle } __Consumer__

    ---

    The Consumer Role is responsible for consuming data from the data space. This role requires a Data Space Connector that is configured to access and retrieve data from the data space.

    [:octicons-arrow-right-24: _AWS Deployment_](../../documentation/mv_data_space/fiware/consumer.md)

    [:octicons-arrow-right-24: _Technical Details_](../../documentation/data_space_connectors/fiware/index.md#consumer)

-   :material-cog-outline:{ .lg .middle } __Provider__

    ---

    The Provider Role is responsible for providing data to the data space. This role requires a Data Space Connector that is configured to share data with the data space.

    [:octicons-arrow-right-24: _AWS Deployment_](../../documentation/mv_data_space/fiware/provider.md)

    [:octicons-arrow-right-24: _Technical Details_](../../documentation/data_space_connectors/fiware/index.md#provider)

</div>

## Data Federation

The Data Federation is a more complex scenario where multiple Data Spaces or data platform are federated to share data. Depending on the technology used, the federation process can be different. [Reference](../../documentation/data_federation/index.md)

## Verifiable Credentials management

The Verifiable Credentials (VCs) management is a crucial aspect of the data space, as it ensures that all participants are authenticated and authorized to access resources. The management of VCs is typically handled by an Identity Management system, such as Keycloak.

<div class="grid cards" markdown>

-   :material-cog-outline:{ .lg .middle } __Keycloak Configuration__

    ---

    The Keycloak Configuration is responsible for managing the authentication and authorization of users and services in the data space. This configuration is crucial for issuing Verifiable Credentials.

    [:octicons-arrow-right-24: _About Keycloak_](../../documentation/verifiable_credentials/index.md#identity-management-keycloak)

    [:octicons-arrow-right-24: _Hands-On Configuration_](../../documentation/verifiable_credentials/keycloak/index.md)

</div>