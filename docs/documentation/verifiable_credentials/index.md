---
title: Verifier Credentials
---

## Verifier Credential Issuer

Every participant in a data space must have a Decentralized Identifier (DID) that identifies them. Also, Keycloak needs this key to set the organization's DID as the issuer in the Verifiable Credential.

To generate the DID in the format required by Keycloak, there is a Dockerized tool called _[did-helper](https://github.com/wistefan/did-helper)_ that produces all the necessary files by configuring just a few parameters to identify your organization:

```shell
docker run -v $(pwd):/cert \
    -e OUTPUT_FORMAT="env" \
    -e OUTPUT_FILE="/cert/did.env" \
    -e KEY_ALIAS="didPrivateKey" \
    -e STORE_PASS="fill_me" \
    -e COUNTRY="BE" \
    -e STATE="BRUSSELS" \
    -e LOCALITY="Brussels" \
    -e ORGANIZATION="Fancy Marketplace Co." \
    -e COMMON_NAME="www.fancy-marketplace.biz" \
    quay.io/wi_stefan/did-helper:0.2.0
```

This will generate two files required by Keycloak to issue Verifiable Credentials: `did.env` and `cert.pfx`.

!!! warning

    Although the process of generating the DID and its use in Keycloak is automated in local FIWARE deployments, this practice, as stated by FIWARE, is not acceptable in production environments. The generation and custody of the DID should be the responsibility of the organization.

<!-- This guide covers generating a DID that identifies the organization and, afterwards, the necessary configuration to use this DID in the Keycloak deployment. -->

## Identity Management - Keycloak

[Keycloak](https://www.keycloak.org/) is a powerful open-source Identity and Access Management solution that can be used to issue Verifiable Credentials (VCs) for users or services in a Data Space. It provides a flexible and secure way to manage identities, roles, and access policies. Some of the key features of Keycloak include:

1. Manage identities of users and services
2. Issue Verifiable Credentials
3. Control access to resources
4. Establish trust through digital identity verification

!!! info

    When configured with your organization's DID, **Keycloak can issue trusted Verifiable Credentials that other participants in the data space can verify**.

### Concepts

Keycloak operates with the concept of [realms](#realms), [clients](#clients), and [users](#users):

#### Realms

A realm is a isolated logical grouping of users, roles, and clients.

Within a realm:

- Users and their credentials are managed.
- Clients are configured, which are applications that use Keycloak for authentication.
- Roles, groups, access policies, etc. are defined.
- Authentication and login flows can be customized (screens, MFA, etc.).
- Each realm is completely independent from others.

#### Clients

Clients are applications that use Keycloak for authentication. For each provider that you want to connect to the Data Space, you need to create a new client. Each client can be configured with its own settings, such as:

- Client ID and secret.
- Redirect URIs.
- Authentication flows.
- Access policies.

#### Users

Users are the entities or members of your organization that will authenticate against Keycloak to access the data space. They can be individuals or services. Each user can have:

- A unique username and password.
- Roles assigned to them.
- Attributes and metadata.

### Configuration

To configure Keycloak for issuing Verifiable Credentials, follow these steps: [Hands-On Configuration](./keycloak/index.md).
