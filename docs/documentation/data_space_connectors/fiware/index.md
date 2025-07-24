---
title: FIWARE Data Space Connector
---

## Overview

The [FIWARE Data Space Connector (FDSC)](https://github.com/FIWARE/data-space-connector) is an integrated suite of components every organization participating in a data space should deploy to _connect_ to a data space. Following the DSBA recommendations, it allows to:

- Interface with Trust Services aligned with [EBSI specifications](https://api-pilot.ebsi.eu/docs/apis).
- Implement authentication based on [W3C DID](https://www.w3.org/TR/did-core/) with [VC/VP standards](https://www.w3.org/TR/vc-data-model/) and [SIOPv2](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html#name-cross-device-self-issued-op)/[OIDC4VP](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html#request_scope) protocols.
- Implement authorization based on attribute-based access control (ABAC) following an [XACML P*P architecture](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=xacml) using [Open Digital Rights Language (ODRL)](https://www.w3.org/TR/odrl-model/) and the [Open Policy Agent (OPA)](https://www.openpolicyagent.org/).
- Provide compatibility with [ETSI NGSI-LD](https://www.etsi.org/committee/cim) as data exchange API.
- Supports the [TMForum APIs](https://www.tmforum.org/oda/open-apis/) for contract negotiation.

!!! note
	Although the FIWARE Data Space Connector provides compatibility with NGSI-LD as the data exchange API, it could also be used for any other RESTful API by replacing or extending the PDP component of the connector.

??? info "Key points"

	- Final and ready-to-use software (versus the framework approach of Eclipse).
	- (Partial support for) IDS Dataspace Protocol (DSP).
	- Not as agnostic as Eclipse, although its modular approach makes it possible (in theory) to extend its capabilities.
	- It is not very tested; expect bugs and error reporting work.
	- Development is relatively slow.

## Getting started

A good way to start working with the connector is to deploy a [Minimum Viable Data Space (MVDS)](../../../getting_started/data_spaces/index.md#minimum-viable-data-space) using FIWARE's minimum infrastructure. This infrastructure provides a minimal implementation of a data space using Fiware technology, which allows test the FIWARE Data Space Connector and its components in a local environment.

<figure markdown>
  ![FIWARE minimal data space](img/minimum_dataspace_arch.svg){ loading=lazy }
</figure>

This MVDS is composed of the following blocks:

| Component | Description |
|-----------|-------------|
| **Fiware Data Space Operator or Trust Anchor** | The entity responsible for managing the issuers and credentials within the data space. It ensures the trustworthiness of the data space by managing the identities and credentials of participants. |
| **FDS Connector A (Provider)** | An entity that provides data from the data space. It acts as a data provider, allowing for data exchange within the data space. |
| **FDS Connector B (Consumer)** | An entity that consumes data from the data space. It acts as a data consumer, retrieving data from the data space without providing any data in return. |

!!! example

	- **FIWARE MVDS local example:** [Code](https://github.com/FIWARE/data-space-connector/blob/main/doc/deployment-integration/local-deployment/LOCAL.MD) repository.
	- **CitcomAI MVDS local example:** [Code](https://github.com/CitComAI-Hub/Minimum_Viable_DataSpace_Infrastructure) repository.

## Technical Details & Deployments

The [FIWARE Data Space Connector repository](https://github.com/FIWARE/data-space-connector) provides a Helm chart for deploying the connector in a Kubernetes cluster. The chart includes all the necessary components to set up a data space connector in both consumer and provider modes. The chart is designed to be flexible and can be customized to fit the specific needs of the data space.

### Consumer

The consumer mode of the FIWARE Data Space Connector is composed of the following components:

![FIWARE Data Space Connector Consumer](./img/consumer_arch.svg)

!!! example "Deployments"
	- Minimum AWS deployment example: [Code](../../mv_data_space/fiware/consumer.md)

| Component | Functionality | Description |
|:---------:|---------------|-------------|
| **DID (did-helper)** | <span style="padding:5px;background-color:#bebbbc">Config Services</span> | A component that provides support for W3C Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs). It helps in managing DIDs and VCs within the data space. |
| **Keycloak** | <span style="padding:5px;background-color:#12cbf8">Authentication</span> | An identity and access management solution that provides authentication and authorization services. It is used to manage user identities and access to resources within the data space. |
| **Rainbow** | <span style="padding:5px;background-color:#f1f812">IDSA Data Space Protocol</span> | Rainbow or also known as Dataspace Rainbow is an implementation of Dataspace Protocol 2024-1 promoted by IDSA (International Data Spaces Association). |
| **PostgreSQL** | <span style="padding:5px;background-color:#b97aff">Database</span> | A relational database management system that stores data related to the data space. |

### Provider

The provider mode of the FIWARE Data Space Connector is composed of the following components:

![FIWARE Data Space Connector Provider](./img/provider_arch.svg){ loading=lazy }

!!! example "Deployments"
	- Minimum AWS deployment example: [Code](../../mv_data_space/fiware/provider.md)

| Component | Functionality | Description |
|:---------:|---------------|-------------|
| **APISIX**	| <span style="padding:5px;background-color:#19f812">Authorization</span> | A component that provides API gateway functionality with a OPA plugin for traffic management. |
| **OPA**	| <span style="padding:5px;background-color:#19f812">Authorization</span> | An open-source policy engine that provides attribute-based access control (ABAC) for the data space. It evaluates policies and makes authorization decisions based on attributes and rules defined in the data space. |
| **ODRL-PAP** | <span style="padding:5px;background-color:#19f812">Authorization</span> | A component that implements the ODRL (Open Digital Rights Language) Policy Administration Point (PAP) for managing data access policies within the data space. |
| **Scopio** | <span style="padding:5px;background-color:#f86d12">Data Broker</span> | A data broker, facilitating the exchange of data between different participants in the data space. It manages data discovery and retrieval processes. |
| **VCVerifier** | <span style="padding:5px;background-color:#12cbf8">Authentication</span> | A component that verifies the authenticity of Verifiable Credentials (VCs) and exchanges them for tokens. It ensures that the credentials presented by participants are valid and trustworthy. |
| **Credential Config Service** | <span style="padding:5px;background-color:#12cbf8">Authentication</span> | A service that manages the configuration of credentials. Holds the information which VCs are required for accessing a service. |
| **Trusted Issuers List** | <span style="padding:5px;background-color:#12cbf8">Authentication</span> | A list of trusted issuers for the provider. Acts as Trusted Issuers List by providing an [EBSI Trusted Issuers Registry](https://hub.ebsi.eu/) API. |
| **TM Forum API** | <span style="padding:5px;background-color:#fe5b8c">Data Discovery</span> | A component that implements the [TM Forum APIs](https://www.tmforum.org/oda/open-apis/) for contract negotiation within the data space. It allows participants to negotiate and manage contracts related to data exchange. |
| **Contract Management** | <span style="padding:5px;background-color:#fe5b8c">Data Discovery</span> | Notification listener for contract management events out of TMForum. |
| **Rainbow** | <span style="padding:5px;background-color:#f1f812">IDSA Data Space Protocol</span> | Rainbow or also known as Dataspace Rainbow is an implementation of Dataspace Protocol 2024-1 promoted by IDSA (International Data Spaces Association). |
| **TPP** | <span style="padding:5px;background-color:#f1f812">IDSA Data Space Protocol</span> | Integration of checks for the transfer process protocol. |
| **PostgreSQL** | <span style="padding:5px;background-color:#b97aff">Database</span> | A relational database management system that stores data related to the data space. |
| **PostGIS** | <span style="padding:5px;background-color:#b97aff">Data Bases</span> | PostgreSQL Database with PostGIS extensions |
| **MySQL** | <span style="padding:5px;background-color:#b97aff">Data Bases</span> | An open-source relational database management system that uses SQL for data management. |
| **DID (did-helper)** | <span style="padding:5px;background-color:#bebbbc">Config Services</span> | A component that provides support for W3C Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs). It helps in managing DIDs and VCs within the data space. |