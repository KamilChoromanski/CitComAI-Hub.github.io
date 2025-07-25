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

## Per-Role Cleanup

??? tip "Trust Anchor"

    ```bash
    export TRUST_ANCHOR_INSTANCE_ID="i-xxxxxxxxx"
    export TRUST_ANCHOR_ALLOCATION_ID="eipalloc-xxxxxxxxx"

    helm uninstall trust-anchor
    aws ec2 terminate-instances --instance-ids $TRUST_ANCHOR_INSTANCE_ID --region $AWS_REGION
    aws ec2 release-address --allocation-id $TRUST_ANCHOR_ALLOCATION_ID --region $AWS_REGION
    aws ec2 delete-security-group --group-name trust-anchor-sg --region $AWS_REGION
    ```

??? tip "Consumer"

    ```bash
    export CONSUMER_INSTANCE_ID="i-xxxxxxxxx"
    export CONSUMER_ALLOCATION_ID="eipalloc-xxxxxxxxx"

    helm uninstall consumer-dsc -n consumer
    kubectl delete namespace consumer
    aws ec2 terminate-instances --instance-ids $CONSUMER_INSTANCE_ID --region $AWS_REGION
    aws ec2 release-address --allocation-id $CONSUMER_ALLOCATION_ID --region $AWS_REGION
    aws ec2 delete-security-group --group-name consumer-sg --region $AWS_REGION
    ```

??? tip "Provider"

    ```bash
    export PROVIDER_INSTANCE_ID="i-xxxxxxxxx"
    export PROVIDER_ALLOCATION_ID="eipalloc-xxxxxxxxx"

    helm uninstall provider-dsc -n provider
    kubectl delete namespace provider
    aws ec2 terminate-instances --instance-ids $PROVIDER_INSTANCE_ID --region $AWS_REGION
    aws ec2 release-address --allocation-id $PROVIDER_ALLOCATION_ID --region $AWS_REGION
    aws ec2 delete-security-group --group-name provider-sg --region $AWS_REGION
    ```

## Background Information

### nip.io Service
nip.io is a free wildcard DNS service that converts subdomains like `service.1.2.3.4.nip.io` into A records pointing to `1.2.3.4`. This eliminates the need for custom DNS configuration during development.

### Architecture
Each role runs on its own EC2 instance with a dedicated k3s Kubernetes cluster. The Ingress Controller in each cluster routes traffic based on hostnames to the appropriate internal services.