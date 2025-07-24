---
title: Trust Anchor 
---

!!! warning
	Check the [prerequisites section](../index.md#common-setup-steps) before proceeding with the deployment.

## Step by Step AWS deployment

!!! warning

	If you are joining an existing dataspace, this step should be skipped as you will use the trust anchor of the dataspace you want to join.

The Trust Anchor provides the basic trust infrastructure for the data space. It is usually the first component to be deployed if you are setting up a data space from scratch. 

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
!!! warning "Important"
    Note the security group ID returned by the create command.

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

!!! warning "Important"
    Note the instance ID returned by this command.

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

!!! danger
    Before deploying, you must modify the Trust Anchor's `values.yaml` file to use your actual IP address instead of `127.0.0.1.nip.io`. Modify `trust-anchor/values.yaml` file to use the external IP address instead of localhost. Replace the `tir` host reference `127.0.0.1.nip.io` with `YOUR_TRUST_ANCHOR_IP.nip.io`. This change ensures that the Trusted Issuer Registry (TIR) is accessible outside the local environment.

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