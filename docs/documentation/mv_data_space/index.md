---
title: MV Data Space on AWS Cloud
---

## Overview

This guide provides step-by-step instructions for deploying a [Minimum Viable Data Space (MVDS)](../../getting_started/data_spaces/index.md#minimum-viable-data-space) on AWS with three distinct roles using Fiware's Technology ([Trust Framework](../trust_frameworks/fiware_trust_anchor/index.md), [FDS Connector](../data_space_connectors/fiware/index.md)). Each role is completely independent and can be deployed separately:

- **Trust Anchor**: It manages the identities and credentials of participants in the data space, ensuring trustworthiness and security. **Only one instance** of this role is needed in the data space.
- **Consumer**: Requests and consumes data/services from providers. **Each participant needs** its own consumer instance.
- **Provider**: Offers data/services to consumers. **Each participant needs** its own provider instance.

!!! warning "Prerequisites"

    Before starting any deployment, ensure you have:

    * [x] AWS CLI installed and configured with appropriate credentials.
    * [x] `kubectl` installed on your system ([Installation Guide](https://kubernetes.io/docs/tasks/tools/)).
    * [x] `helm` installed on your system ([Installation Guide](https://helm.sh/docs/intro/install/)).
    * [x] Basic understanding of Kubernetes and AWS EC2.

## Common Setup Steps

### Get Your Public IP

First, determine your current public IP address for security group configuration:

```bash
curl -s https://checkip.amazonaws.com
```

Note this IP address - you'll need it for security group configuration in each role.

### Create SSH Key Pair

In addition, we will also need an SSH key to access EC2 instances. Create an SSH key pair that will be used across all deployments:

```bash
aws ec2 create-key-pair \
  --key-name dataspace-key \
  --query 'KeyMaterial' \
  --output text > dataspace-key.pem

chmod 400 dataspace-key.pem
```

### Clone deployment repository

```bash
# Clone
git clone https://github.com/wistefan/deployment-demo.git
cd deployment-demo

# Open with your preferred editor. For intance:
code .
```

## Components Deployment

Below you'll find deployment instructions for each component in our Minimum Viable Data Space. Choose the component you need to deploy based on your role in the data space. Remember that a complete data space requires at least one Trust Anchor and at least one pair of Provider and Consumer. Follow the links for detailed deployment instructions specific to each component.

<div class="grid cards" markdown>

-   :material-account:{ .lg .middle } __Consumer__

    ---

    Fiware Data Space Connector (_Consumer role_) that is configured to access and retrieve data/services from the data space.

    [:octicons-arrow-right-24: _AWS_](../../documentation/mv_data_space/fiware/consumer.md)

    [:octicons-arrow-right-24: _Technical Details_](../../documentation/data_space_connectors/fiware/index.md#consumer)

-   :material-factory:{ .lg .middle } __Provider__

    ---

    Fiware Data Space Connector (_Provider role_) that is configured to share data/services with the data space.

    [:octicons-arrow-right-24: _AWS_](../../documentation/mv_data_space/fiware/provider.md)

    [:octicons-arrow-right-24: _Technical Details_](../../documentation/data_space_connectors/fiware/index.md#provider)

-   :material-security:{ .lg .middle } __Trust Anchor__

    ---

    !!! warning
        
        It is not necessary to deploy this if you want to connect to an existing Data Space.

    It serves as a trusted entity that issues and manages digital certificates (Verifiable Credentials) for organizations and individuals participating in the data space.

    [:octicons-arrow-right-24: _AWS_](../../documentation/mv_data_space/fiware/trust_anchor.md)

    [:octicons-arrow-right-24: _Technical Details_](../trust_frameworks/fiware_trust_anchor/index.md)

</div>
