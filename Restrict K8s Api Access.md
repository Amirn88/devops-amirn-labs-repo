# Steps to Restrict Access to Kubernetes API for Specific IPs

Restricting access to the Kubernetes API server to specific IP addresses is a crucial security measure to prevent unauthorized access and enhance cluster security. Follow the steps below for a clear and easy implementation:

## Why Restrict Kubernetes API Access?

- **Enhanced Security:** Limits access to the Kubernetes API server to authorized users and systems only.
- **Reduced Attack Surface:** Prevents exposure of the API server to potential attacks from the public internet.
- **Compliance:** Meets security and compliance standards by ensuring only trusted IPs can access sensitive systems.

---

## Step-by-Step Guide

### 1. **Identify Authorized IP Addresses**
   - Collect the IP addresses or CIDR ranges of the users and systems that require access to the Kubernetes API server.

### 2. **Review Existing Firewall Rules**
   - Check current rules allowing access to port `6443` (default Kubernetes API port):
     ```bash
     gcloud compute firewall-rules list --filter="allowed.tcp:6443"
     ```
   - Identify any rules that allow open access to this port.

### 3. **Remove Unrestricted Access Rules** (if necessary)
   - Disable or delete any open access rules:
     ```bash
     gcloud compute firewall-rules delete <rule-name>
     ```

### 4. **Create a Firewall Rule for Specific IPs**
   - Allow traffic to port `6443` only from authorized IPs:
     ```bash
     gcloud compute firewall-rules create allow-k8s-api-specific-ips \
         --network=<network-name> \
         --allow=tcp:6443 \
         --source-ranges=<authorized-ip-1>,<authorized-ip-2> \
         --description="Allow Kubernetes API access for specific IPs"
     ```
     - Replace `<network-name>` with your VPC network (e.g., `default`).
     - Replace `<authorized-ip-1>,<authorized-ip-2>` with the actual authorized IPs.

### 5. **Test the New Rule**
   - Verify that authorized IPs can connect to the API server using `kubectl`.
   - Attempt access from unauthorized IPs to ensure the restrictions are in place.

### 6. **(Optional) Restrict Egress Traffic**
   - If needed, restrict outgoing traffic from the Kubernetes API server:
     ```bash
     gcloud compute firewall-rules create restrict-k8s-api-egress \
         --network=<network-name> \
         --direction=EGRESS \
         --deny=all \
         --destination-ranges=0.0.0.0/0 \
         --description="Restrict egress traffic from Kubernetes API server"
     ```

---

## Best Practices

1. **Use the Principle of Least Privilege:**
   - Grant access only to IPs and users that need it.

2. **Regularly Audit Rules:**
   - Periodically review firewall rules to ensure they remain aligned with your security requirements.

3. **Update Rules as Needed:**
   - If new IPs need access, update the existing rule:
     ```bash
     gcloud compute firewall-rules update allow-k8s-api-specific-ips \
         --source-ranges=<updated-ip-list>
     ```

4. **Consider VPN or Bastion Hosts:**
   - For added security, use VPN or bastion hosts to manage API access.

By following these steps, you can restrict access to your Kubernetes API server, enhancing the overall security and minimizing risks to your cluster.

