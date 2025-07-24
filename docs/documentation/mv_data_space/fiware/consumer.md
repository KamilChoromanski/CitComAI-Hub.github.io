---
title: FDSC Consumer
---

!!! warning
	Check the [prerequisites section](../index.md#common-setup-steps) before proceeding with the deployment.

## Step by Step AWS deployment

The Consumer role allows you to request and consume data/services from providers in the data space.

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

!!! warning "Important"
    Note the security group ID returned by the create command.

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

!!! warning "Important"
    Note the instance ID returned by this command.

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
!!! warning "Important"
    Note the DID returned by the `did-helper`. It is the consumer DID.

### Step 10: Deploy Identity Secret

```bash
# Create secret with the identity
kubectl create secret generic consumer-identity --from-file=consumer-identity/cert.pfx -n consumer
```

### Step 11: Configure Values

!!! danger
    Before deploying, you must modify the Consumer's `values.yaml` file to use your actual IP address instead of `127.0.0.1.nip.io`. Modify `consumer/values.yaml`:

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
