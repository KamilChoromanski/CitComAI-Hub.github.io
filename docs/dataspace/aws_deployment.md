# AWS Data Space Deployment Guide (EC2)

## Overview

This guide provides step-by-step instructions for deploying a data space on AWS with three distinct roles. Each role is completely independent and can be deployed separately:

- **Trust Anchor**: Provides the foundational trust infrastructure
- **Consumer**: Requests and consumes data from providers
- **Provider**: Offers data services to consumers
## Prerequisites

Before starting any deployment, ensure you have:
- AWS CLI installed and configured with appropriate credentials
- `kubectl` installed on your system ([Installation Guide](https://kubernetes.io/docs/tasks/tools/))
- `helm` installed on your system
- Basic understanding of Kubernetes and AWS EC2

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

## Trust Anchor Deployment

The Trust Anchor provides the basic trust infrastructure for the data space. It is usually the first component to be deployed if you are setting up a data space from scratch. If you are joining an existing dataspace, this step should be skipped as you will use the trust anchor of the dataspace you want to join.

### Step 1: Create Security Group

Create a dedicated security group for the Trust Anchor:

```bash
# Set your configuration
export YOUR_PUBLIC_IP="YOUR_IP_HERE"  # Replace with your public IP
export AWS_REGION="eu-west-1"         # Replace with your preferred region

# Create security group
aws ec2 create-security-group \
  --group-name trust-anchor-sg \
  --description "Security group for Trust Anchor" \
  --region $AWS_REGION

# Add SSH access from your IP
aws ec2 authorize-security-group-ingress \
  --group-name trust-anchor-sg \
  --protocol tcp \
  --port 22 \
  --cidr ${YOUR_PUBLIC_IP}/32 \
  --region $AWS_REGION

# Add Kubernetes API access from your IP
aws ec2 authorize-security-group-ingress \
  --group-name trust-anchor-sg \
  --protocol tcp \
  --port 6443 \
  --cidr ${YOUR_PUBLIC_IP}/32 \
  --region $AWS_REGION

# Add HTTP/HTTPS access (public)
aws ec2 authorize-security-group-ingress \
  --group-name trust-anchor-sg \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region $AWS_REGION
```

**Important**: Note the security group ID returned by the create command.

### Step 2: Launch Trust Anchor Instance
For the Trust Anchor instance we use Ubuntu 22.04 LTS image (`ami-0694d931cee176e7d`) and `t3.medium` instance type.  Feel free to change these parameters, especially if you see that the load to be supported is greater than the capacity of the virtual machine.

```bash
# Replace with your security group ID
export TRUST_ANCHOR_SG_ID="sg-xxxxxxxxx"

# Launch Trust Anchor instance
aws ec2 run-instances \
  --image-id ami-0694d931cee176e7d \
  --instance-type t3.medium \
  --key-name dataspace-key \
  --security-group-ids $TRUST_ANCHOR_SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=trust-anchor}]' \
  --region $AWS_REGION
```

**Important**: Note the instance ID returned by this command.

### Step 3: Assign Elastic IP

```bash
# Replace with your Trust Anchor instance ID
export TRUST_ANCHOR_INSTANCE_ID="i-xxxxxxxxx"

# Allocate Elastic IP
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=trust-anchor-ip}]' \
  --region $AWS_REGION

# Associate IP to instance (replace ALLOCATION_ID with the one returned above)
aws ec2 associate-address \
  --instance-id $TRUST_ANCHOR_INSTANCE_ID \
  --allocation-id ALLOCATION_ID_FROM_ABOVE \
  --region $AWS_REGION
```

### Step 4: Verify Instance Status

```bash
aws ec2 describe-instances \
  --instance-ids $TRUST_ANCHOR_INSTANCE_ID \
  --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], PublicIpAddress, State.Name]' \
  --output table \
  --region $AWS_REGION
```

### Step 5: Install k3s

```bash
# Replace with your Trust Anchor public IP
export TRUST_ANCHOR_IP="YOUR_TRUST_ANCHOR_IP"

# Connect to the instance
ssh -i "dataspace-key.pem" ubuntu@$TRUST_ANCHOR_IP

# Install k3s
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--tls-san $TRUST_ANCHOR_IP" sh -

# Get the kubeconfig
sudo cat /etc/rancher/k3s/k3s.yaml
```

### Step 6: Configure Local Access

On your local machine, create a kubeconfig file for the Trust Anchor:

```bash
# Create k3s-trust-anchor.yaml with the content from the previous step (cat command)
# Replace 127.0.0.1 with your public Trust Anchor IP in the server field
# The file should contain:
# server: https://YOUR_TRUST_ANCHOR_IP:6443

# Test the connection
export KUBECONFIG=k3s-trust-anchor.yaml
kubectl get nodes
```

### Step 7: Configure Storage

```bash
# Enable storage provisioner
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml

# Wait a few seconds for it to start. You can check its status with
kubectl get pods -n local-path-storage
```

### Step 8: Add Helm Repository

```bash
helm repo add data-space-connector https://fiware.github.io/data-space-connector/
helm repo update
```

### Step 9: Configure Values

**CRITICAL**: Before deploying, you must modify the Trust Anchor's `values.yaml` file to use your actual IP address instead of `127.0.0.1.nip.io`. Modify `trust-anchor/values.yaml` file to use the external IP address instead of localhost. Replace the `tir` host reference `127.0.0.1.nip.io` with `YOUR_TRUST_ANCHOR_IP.nip.io`. This change ensures that the Trusted Issuer Registry (TIR) is accessible outside the local environment.

```yaml
trusted-issuers-list:
  tir:
    enabled: true
    hosts:
      - host: tir.YOUR_TRUST_ANCHOR_IP.nip.io
  til:
    enabled: true
    hosts:
      - host: til.127.0.0.1.nip.io # Do not modify
```

### Step 10: Create namespace

```bash
# Create namespace
kubectl create namespace trust-anchor
```

### Step 11: Deploy Trust Anchor

```bash
# Deploy using your modified values file
helm install trust-anchor data-space-connector/trust-anchor --version 0.2.0 -f trust-anchor/values.yaml --namespace=trust-anchor

# Monitor deployment
watch kubectl get pods -n trust-anchor
```

### Step 12: Changes and updates
```bash
# Upgrade
helm upgrade trust-anchor data-space-connector/trust-anchor -f trust-anchor/values.yaml --namespace trust-anchor

# Monitor
watch kubectl get pods -n trust-anchor
```

## Consumer Deployment

The Consumer role allows you to request and consume data from providers in the data space.

### Step 1: Create Security Group

Create a dedicated security group for the Consumer:

```bash
# Set your configuration
export YOUR_PUBLIC_IP="YOUR_IP_HERE"  # Replace with your public IP
export AWS_REGION="eu-west-1"         # Replace with your preferred region

# Create security group
aws ec2 create-security-group \
  --group-name consumer-sg \
  --description "Security group for Consumer" \
  --region $AWS_REGION

# Add SSH access from your IP
aws ec2 authorize-security-group-ingress \
  --group-name consumer-sg \
  --protocol tcp \
  --port 22 \
  --cidr ${YOUR_PUBLIC_IP}/32 \
  --region $AWS_REGION

# Add Kubernetes API access from your IP
aws ec2 authorize-security-group-ingress \
  --group-name consumer-sg \
  --protocol tcp \
  --port 6443 \
  --cidr ${YOUR_PUBLIC_IP}/32 \
  --region $AWS_REGION

# Add HTTP/HTTPS access (public)
aws ec2 authorize-security-group-ingress \
  --group-name consumer-sg \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region $AWS_REGION
```

**Important**: Note the security group ID returned by the create command.

### Step 2: Launch Consumer Instance
For the Consumer instance we use Ubuntu 22.04 LTS image (`ami-0694d931cee176e7d`) and `t3.large` instance type. Feel free to change these parameters, especially if you see that the load to be supported is greater than the capacity of the virtual machine.

```bash
# Replace with your security group ID
export CONSUMER_SG_ID="sg-xxxxxxxxx"

# Launch Consumer instance
aws ec2 run-instances \
  --image-id ami-0694d931cee176e7d \
  --instance-type t3.large \
  --key-name dataspace-key \
  --security-group-ids $CONSUMER_SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=consumer}]' \
  --region $AWS_REGION
```

**Important**: Note the instance ID returned by this command.

### Step 3: Assign Elastic IP

```bash
# Replace with your Consumer instance ID
export CONSUMER_INSTANCE_ID="i-xxxxxxxxx"

# Allocate Elastic IP
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=consumer-ip}]' \
  --region $AWS_REGION

# Associate IP to instance (replace ALLOCATION_ID with the one returned above)
aws ec2 associate-address \
  --instance-id $CONSUMER_INSTANCE_ID \
  --allocation-id ALLOCATION_ID_FROM_ABOVE \
  --region $AWS_REGION
```

### Step 4: Verify Instance Status

```bash
aws ec2 describe-instances \
  --instance-ids $CONSUMER_INSTANCE_ID \
  --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], PublicIpAddress, State.Name]' \
  --output table \
  --region $AWS_REGION
```

### Step 5: Install k3s

```bash
# Replace with your Consumer public IP
export CONSUMER_IP="YOUR_CONSUMER_IP"

# Connect to the instance
ssh -i "dataspace-key.pem" ubuntu@$CONSUMER_IP

# Install k3s
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--tls-san $CONSUMER_IP" sh -

# Get the kubeconfig
sudo cat /etc/rancher/k3s/k3s.yaml
```

### Step 6: Configure Local Access

On your local machine, create a kubeconfig file for the Consumer:

```bash
# Create k3s-consumer.yaml with the content from the previous step
# Replace 127.0.0.1 with your Consumer IP in the server field
# The file should contain:
# server: https://YOUR_CONSUMER_IP:6443

# Test the connection
export KUBECONFIG=k3s-consumer.yaml
kubectl get nodes
```

### Step 7: Configure Storage 

```bash
# Enable storage provisioner
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml

# Wait a few seconds for it to start. You can check its status with
kubectl get pods -n local-path-storage
```

### Step 8: Create namespace
```bash
kubectl create namespace consumer
```

### Step 9: Create Consumer Identity

```bash
# Create directory for identity files
mkdir consumer-identity

# Generate the private key - dont get confused about the curve, openssl uses the name `prime256v1` for `secp256r1`(as defined by P-256)
openssl ecparam -name prime256v1 -genkey -noout -out consumer-identity/private-key.pem

# Generate corresponding public key
openssl ec -in consumer-identity/private-key.pem -pubout -out consumer-identity/public-key.pem

# Create a (self-signed) certificate
openssl req -new -x509 -key consumer-identity/private-key.pem -out consumer-identity/cert.pem -days 360

# Export the keystore
openssl pkcs12 -export -inkey consumer-identity/private-key.pem -in consumer-identity/cert.pem -out consumer-identity/cert.pfx -name didPrivateKey

# Check the contents
keytool -v -keystore consumer-identity/cert.pfx -list -alias didPrivateKey

# Generate did from the keystore
wget https://github.com/wistefan/did-helper/releases/download/0.1.1/did-helper
chmod +x did-helper
./did-helper -keystorePath ./consumer-identity/cert.pfx -keystorePassword=test
```

**Important**: Note the DID returned by the `did-helper`. It is the consumer DID.


### Step 10: Deploy Identity Secret

```bash
# Create secret with the identity
kubectl create secret generic consumer-identity --from-file=consumer-identity/cert.pfx -n consumer
```

### Step 11: Configure Values

**CRITICAL**: Before deploying, you must modify the Consumer's `values.yaml` file to use your actual IP address instead of `127.0.0.1.nip.io`. Modify `consumer/values.yaml`:


```yaml
# 1. Replace the localhost address for the Keycloak ingress hostname:
keycloak:
  ingress:
    enabled: true
    hostname: keycloak-consumer.YOUR_CONSUMER_IP.nip.io


# 2. Replace the localhost also for KC_HOSTNAME in extraVars:
- name: KC_HOSTNAME
    value: keycloak-consumer.YOUR_CONSUMER_IP.nip.io

# 3. In realm, replace:
realm:
frontendUrl: http://keycloak-consumer.127.0.0.1.nip.io:8080

# with
realm:
frontendUrl: http://keycloak-consumer.YOUR_CONSUMER_IP.nip.io

# 4. Replace DID with you own, previously generated consumer DID.
- name: DID
    value: "did:key:xxxxxxxxxx"
```

### Step 12: Add Helm Repository

```bash
helm repo add data-space-connector https://fiware.github.io/data-space-connector/
helm repo update
```

### Step 13: Deploy Consumer

```bash
# Deploy using your modified values file
helm install consumer-dsc data-space-connector/data-space-connector --version 7.17.0 -f consumer/values.yaml --namespace=consumer

# Monitor deployment
watch kubectl get pods -n consumer
```

## Provider Deployment

The Provider role allows you to offer data services to consumers in the data space.

### Step 1: Create Security Group

Create a dedicated security group for the Provider:

```bash
# Set your configuration
export YOUR_PUBLIC_IP="YOUR_IP_HERE"  # Replace with your public IP
export AWS_REGION="eu-west-1"         # Replace with your preferred region

# Create security group
aws ec2 create-security-group \
  --group-name provider-sg \
  --description "Security group for Provider" \
  --region $AWS_REGION

# Add SSH access from your IP
aws ec2 authorize-security-group-ingress \
  --group-name provider-sg \
  --protocol tcp \
  --port 22 \
  --cidr ${YOUR_PUBLIC_IP}/32 \
  --region $AWS_REGION

# Add Kubernetes API access from your IP
aws ec2 authorize-security-group-ingress \
  --group-name provider-sg \
  --protocol tcp \
  --port 6443 \
  --cidr ${YOUR_PUBLIC_IP}/32 \
  --region $AWS_REGION

# Add HTTP/HTTPS access (public)
aws ec2 authorize-security-group-ingress \
  --group-name provider-sg \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region $AWS_REGION
```

**Important**: Note the security group ID returned by the create command.

### Step 2: Launch Provider Instance
For the Trust Anchor instance we use Ubuntu 22.04 LTS image (`ami-0694d931cee176e7d`) and `t3.xlarge` instance type.  Feel free to change these parameters, especially if you see that the load to be supported is greater than the capacity of the virtual machine.

```bash
# Replace with your security group ID
export PROVIDER_SG_ID="sg-xxxxxxxxx"

# Launch Provider instance
aws ec2 run-instances \
  --image-id ami-0694d931cee176e7d \
  --instance-type t3.xlarge \
  --key-name dataspace-key \
  --security-group-ids $PROVIDER_SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=provider}]' \
  --region $AWS_REGION
```

**Important**: Note the instance ID returned by this command.

### Step 3: Assign Elastic IP

```bash
# Replace with your Provider instance ID
export PROVIDER_INSTANCE_ID="i-xxxxxxxxx"

# Allocate Elastic IP
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=provider-ip}]' \
  --region $AWS_REGION

# Associate IP to instance (replace ALLOCATION_ID with the one returned above)
aws ec2 associate-address \
  --instance-id $PROVIDER_INSTANCE_ID \
  --allocation-id ALLOCATION_ID_FROM_ABOVE \
  --region $AWS_REGION
```

### Step 4: Verify Instance Status

```bash
aws ec2 describe-instances \
  --instance-ids $PROVIDER_INSTANCE_ID \
  --query 'Reservations[*].Instances[*].[Tags[?Key==`Name`].Value | [0], PublicIpAddress, State.Name]' \
  --output table \
  --region $AWS_REGION
```

### Step 5: Prepare Instance Storage

Since the Provider handles more data, you may need to increase the EBS volume size:

1. Go to AWS Console → EC2 → Volumes
2. Find the volume associated with your Provider instance
3. Select it and click "Actions" → "Modify volume"
4. Increase the size to at least 16 GB
5. Save the changes

```bash
# Replace with your Provider public IP
export PROVIDER_IP="YOUR_PROVIDER_IP"

# Connect to the instance
ssh -i "dataspace-key.pem" ubuntu@$PROVIDER_IP

# Update and install utilities
sudo apt-get update && sudo apt-get install -y cloud-guest-utils

# Expand the partition and filesystem
sudo growpart /dev/nvme0n1 1
sudo resize2fs /dev/root

# Verify the changes
df -h
```

### Step 6: Install k3s

```bash
# Install k3s
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--tls-san $PROVIDER_IP" sh -

# Get the kubeconfig
sudo cat /etc/rancher/k3s/k3s.yaml
```

### Step 7: Configure Local Access

On your local machine, create a kubeconfig file for the Provider:

```bash
# Create k3s-provider.yaml with the content from the previous step
# Replace 127.0.0.1 with your Provider IP in the server field
# The file should contain:
# server: https://YOUR_PROVIDER_IP:6443

# Test the connection
export KUBECONFIG=k3s-provider.yaml
kubectl get nodes
```

### Step 8: Configure Storage and Namespace

```bash
# Enable storage provisioner
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml

# Wait a few seconds for it to start. You can check its status with
kubectl get pods -n local-path-storage

# Create namespace
kubectl create namespace provider
```

### Step 9: Create Provider Identity

```bash
# Create directory for identity files

# generate the private key - dont get confused about the curve, openssl uses the name `prime256v1` for `secp256r1`(as defined by P-256)
openssl ecparam -name prime256v1 -genkey -noout -out provider-identity/private-key.pem

# generate corresponding public key
openssl ec -in provider-identity/private-key.pem -pubout -out provider-identity/public-key.pem

# create a (self-signed) certificate
openssl req -new -x509 -key provider-identity/private-key.pem -out provider-identity/cert.pem -days 360

# export the keystore
openssl pkcs12 -export -inkey provider-identity/private-key.pem -in provider-identity/cert.pem -out provider-identity/cert.pfx -name didPrivateKey

# check the contents
keytool -v -keystore provider-identity/cert.pfx -list -alias didPrivateKey

# generate did from the keystore
wget https://github.com/wistefan/did-helper/releases/download/0.1.1/did-helper
chmod +x did-helper
./did-helper -keystorePath ./provider-identity/cert.pfx -keystorePassword=test
```

**Important**: Note the DID returned by the `did-helper`. It is the provider DID.


### Step 10: Deploy Identity Secret

```bash
# Create secret with the identity
kubectl create secret generic provider-identity --from-file=provider-identity/cert.pfx -n provider
```

### Step 11: Configure Values
**CRITICAL**: Before deploying, you must modify the Providers's `values.yaml` file to use your actual IP address instead of `127.0.0.1.nip.io`. Modify `provider/values.yaml` file to use the external IP address instead of localhost. Other variables such as the provider DID should also be modified. In your `provider/values.yaml` file, make these changes:

```yaml
# Summary of Changes in provider/values.yaml

## 1. Hostnames updated from localhost (127.0.0.1.nip.io) to YOUR_PROVIDER_IP (YOUR_PROVIDER_IP.nip.io)
- provider-verifier.127.0.0.1.nip.io            → provider-verifier.YOUR_PROVIDER_IP.nip.io
# - til-provider.127.0.0.1.nip.io                 → til-provider.YOUR_PROVIDER_IP.nip.io
- mp-data-service.127.0.0.1.nip.io              → mp-data-service.YOUR_PROVIDER_IP.nip.io
# - pap-provider.127.0.0.1.nip.io                 → pap-provider.YOUR_PROVIDER_IP.nip.io
- tm-forum-api.127.0.0.1.nip.io                 → tm-forum-api.YOUR_PROVIDER_IP.nip.io

## 2. DID & TIR configuration updated
- tirAddress: http://tir.127.0.0.1.nip.io:8080  → tirAddress: http://trusted-issuers-list:8080
- did: did:key:zDnaeQfjsx66YNYV86SDBB1e5kunWKJcWwk686dvjirEE7pqW  → did: did:key:provider_key

## 3. Server host URLs updated
- host: http://provider-verifier.127.0.0.1.nip.io:8080
  → host: http://provider-verifier.YOUR_PROVIDER_IP.nip.io

## 4. Added fullnameOverride to trusted-issuers-list
+ fullnameOverride: trusted-issuers-list

## 5. APISIX routes and upstream hostnames updated
- hostname: mp-data-service.127.0.0.1.nip.io     → hostname: mp-data-service.YOUR_PROVIDER_IP.nip.io
- host: mp-data-service.127.0.0.1.nip.io         → host: mp-data-service.YOUR_PROVIDER_IP.nip.io

## 6. ODRL PAP organization DID updated
- value: did:key:zDnaeQfjsx66YNYV86SDBB1e5kunWKJcWwk686dvjirEE7pqW
  → value: did:key:provider_key

## 7. Scorpio trustedParticipantsLists endpoints updated
- http://tir.trust-anchor.svc.cluster.local:8080 → http://tir.TRUS_ANCHOR_IP.nip.io
```

### Step 12: Add Helm Repository

```bash
helm repo add data-space-connector https://fiware.github.io/data-space-connector/
helm repo update
```

### Step 13: Deploy Provider

```bash
# Deploy using your modified values file
helm install provider-dsc data-space-connector/data-space-connector \
  --version 7.17.0 \
  -f provider/values.yaml \
  --namespace=provider

# Monitor deployment
kubectl get pods -n provider -w
```

### Changes and updates
```bash
# Update
helm upgrade provider-dsc data-space-connector/data-space-connector -f provider/values.yaml --namespace provider

# Monitor
watch kubectl get pods -n provider
```

## Cleanup

### Per-Role Cleanup

**Trust Anchor:**
```bash
export TRUST_ANCHOR_INSTANCE_ID="i-xxxxxxxxx"
export TRUST_ANCHOR_ALLOCATION_ID="eipalloc-xxxxxxxxx"

helm uninstall trust-anchor
aws ec2 terminate-instances --instance-ids $TRUST_ANCHOR_INSTANCE_ID --region $AWS_REGION
aws ec2 release-address --allocation-id $TRUST_ANCHOR_ALLOCATION_ID --region $AWS_REGION
aws ec2 delete-security-group --group-name trust-anchor-sg --region $AWS_REGION
```

**Consumer:**
```bash
export CONSUMER_INSTANCE_ID="i-xxxxxxxxx"
export CONSUMER_ALLOCATION_ID="eipalloc-xxxxxxxxx"

helm uninstall consumer-dsc -n consumer
kubectl delete namespace consumer
aws ec2 terminate-instances --instance-ids $CONSUMER_INSTANCE_ID --region $AWS_REGION
aws ec2 release-address --allocation-id $CONSUMER_ALLOCATION_ID --region $AWS_REGION
aws ec2 delete-security-group --group-name consumer-sg --region $AWS_REGION
```

**Provider:**
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