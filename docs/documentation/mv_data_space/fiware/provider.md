---
title: FDSC Provider
---

!!! warning
	Check the [prerequisites section](../index.md#common-setup-steps) before proceeding with the deployment.

## Step by Step AWS deployment

The Provider role allows you to offer data/services to consumers in the data space.

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

!!! warning "Important"
    Note the security group ID returned by the create command.

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

!!! warning "Important"
    Note the instance ID returned by this command.

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

!!! warning "Important"
    Note the DID returned by the `did-helper`. It is the consumer DID.

### Step 10: Deploy Identity Secret

```bash
# Create secret with the identity
kubectl create secret generic provider-identity --from-file=provider-identity/cert.pfx -n provider
```

### Step 11: Configure Values

!!! danger
    
    Before deploying, you must modify the Providers's `values.yaml` file to use your actual IP address instead of `127.0.0.1.nip.io`. Modify `provider/values.yaml` file to use the external IP address instead of localhost. Other variables such as the provider DID should also be modified. In your `provider/values.yaml` file, make these changes:

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

