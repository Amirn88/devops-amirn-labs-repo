# Restricting Access to Kubernetes API Server for Specific IPs

In Google Cloud, access to the Kubernetes API server (port `tcp:6443`) can be restricted to specific IP addresses by modifying firewall rules. This document provides a step-by-step guide to achieving this.

---

## Steps to Restrict Access to Specific IP Addresses

### 1. Identify the Authorized IP Addresses

Gather the list of IP addresses or CIDR ranges that need access to the Kubernetes API server.

---

### 2. Remove Existing Open Access Rules (Optional)

If there are existing firewall rules that allow unrestricted access to port `6443`, disable or delete them.

- **List the Existing Rules**:
  ```bash
  gcloud compute firewall-rules list --filter="allowed.tcp:6443"
  ```

- **Delete or Update Open Rules**:
  ```bash
  gcloud compute firewall-rules delete <rule-name>
  ```
  Replace `<rule-name>` with the name of the firewall rule to be removed.

---

### 3. Create a Firewall Rule for Specific IPs

Create a new firewall rule to allow access to the Kubernetes API server only for the authorized IP addresses.

```bash
gcloud compute firewall-rules create allow-k8s-api-specific-ips \
    --network=<network-name> \
    --allow=tcp:6443 \
    --source-ranges=<authorized-ip-1>,<authorized-ip-2> \
    --description="Allow Kubernetes API access for specific IPs"
```

- Replace `<network-name>` with your VPC network name (e.g., `default`).
- Replace `<authorized-ip-1>,<authorized-ip-2>` with the list of authorized IP addresses.

---

### 4. Test the New Access Rule

- **Verify Access**:
  Test connectivity from one of the authorized IPs using `kubectl`.
  ```bash
  kubectl get nodes
  ```

- **Test Unauthorized Access**:
  Attempt to connect from an unauthorized IP to ensure the restriction is enforced.

---

### 5. (Optional) Limit Egress Traffic from API Server

If required, restrict outgoing traffic from the Kubernetes API server by defining an egress firewall rule.

```bash
gcloud compute firewall-rules create restrict-k8s-api-egress \
    --network=<network-name> \
    --direction=EGRESS \
    --deny=all \
    --destination-ranges=0.0.0.0/0 \
    --description="Restrict egress traffic from Kubernetes API server"
```

---

## Additional Considerations

### IP Whitelisting Updates

If new IP addresses need access in the future, update the existing rule:

```bash
gcloud compute firewall-rules update allow-k8s-api-specific-ips \
    --source-ranges=<updated-ip-list>
```

Replace `<updated-ip-list>` with the updated list of authorized IPs.

### Audit Firewall Changes

Regularly audit your firewall rules to ensure only intended IPs have access to sensitive ports like `6443`.

### Alternative: Use VPN or Bastion Host

For enhanced security, consider using a VPN or Bastion Host to access the Kubernetes API securely instead of exposing it directly to the internet.

---

