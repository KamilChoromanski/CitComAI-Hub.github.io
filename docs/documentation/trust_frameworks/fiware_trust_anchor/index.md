---
title: Technical Details
---

## Technical Details

The **Trusted-Issuers-List Service** provides an [EBSI-Trusted Issuers Registry](https://hub.ebsi.eu/apis/pilot/trusted-issuers-registry/v4) implementation to act as the Trusted-List-Service in the DSBA Trust and IAM Framework. In addition, a Trusted Issuers List API to manage the issuers is provided.

Both APIs [Trusted-Issuers-List API](https://github.com/FIWARE/trusted-issuers-list/blob/main/api/trusted-issuers-list.yaml) and [Trusted-Issuers-Registry API](https://github.com/FIWARE/trusted-issuers-list/blob/main/api/trusted-issuers-registry.yaml), are found on the **same port** (*by default 8080*) but different contexts.

- **Trusted-Issuers-List API**: `/issuer`
- **Trusted-Issuers-Registry API**: `/v4/issuers/`

!!! info "Fiware Trust Anchor"
    The Fiware Trust Anchor is based in: [FIWARE Trusted Issuers List](https://github.com/FIWARE/trusted-issuers-list)

![trust_anchor](./img/trust_anchor_arch.svg)


> _The default setup of the connector requires an EBSI-Trusted Issuers Registry to provide the list of participants. The local Data Spaces comes with the FIWARE Trusted Issuers List as a rather simple implementation of that API, providing CRUD functionality for Issuers and storage in an MySQL Database. After deployment, the API is available at http://tir.127.0.0.1.nip.io:8080. Both participants are automatically registered as "Trusted Issuers" in the registry with their did's._

## API details (version 0.0.2)

### Trusted Issuers List

<swagger-ui src="https://raw.githubusercontent.com/FIWARE/trusted-issuers-list/refs/tags/0.0.2/api/trusted-issuers-list.yaml"/>

### Trusted Issuers Registry

<swagger-ui src="https://raw.githubusercontent.com/FIWARE/trusted-issuers-list/refs/tags/0.0.2/api/trusted-issuers-registry.yaml"/>
