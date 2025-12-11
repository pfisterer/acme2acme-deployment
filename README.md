# 🛡️ Ansible Playbook: ACME-to-ACME Proxy Deployment (acme2certifier)

This Ansible Playbook automates the deployment of the `acme2certifier` ([grindsa/acme2certifier](https://github.com/grindsa/acme2certifier)) service within a K3s Kubernetes cluster. It just requires a SSH-accessible Ubuntu/Debian node.

`acme2certifier` acts as an **ACME Proxy**, allowing standard ACME clients (like Cert-Manager) to interact with a centralized proxy endpoint instead of directly communicating with an upstream ACME server that requires complex **External Account Binding (EAB)** credentials, such as **Harica**.

This proxy setup simplifies client configuration and enhances security by centralizing and protecting the sensitive EAB secrets.

---
## Project Rationale

The primary goal is to abstract the complexities of using ACME providers that require EAB (e.g., Harica).

* **Problem:** Direct usage of the upstream ACME server (Harica) mandates the distribution of sensitive EAB credentials (`KeyID` and `Secret`) to every client (e.g., every Cert-Manager instance).
* **Solution:** Deploying `acme2certifier` as an intermediary. Users point their ACME clients to your internal ACME proxy server. The proxy handles the challenges, communicates with Harica using the centralized EAB credentials, and issues certificates.
* **Benefit:** Clients only require the proxy's URL and can use standard challenges (HTTP-01 or DNS-01) without needing any EAB credentials.

---

## Usage

This script uses external Ansible roles. To install the required roles, run:

```bash
ansible-galaxy install -r requirements.yml
```

To configure and deploy the ACME proxy, customize the variables in your inventory or a dedicated vars file. To customize the k3s deployment, see [k3s-dhbw-cloud-role](https://github.com/pfisterer/k3s-dhbw-cloud-role). This is an exemplary inventory file (e.g., `inventory.yaml`):

```yaml
all:
    hosts:
        "141.1.2.3":
            ip_family: ipv4
            ansible_user: ubuntu

            # Include additional configuration variables for the k3s role    
        
            acme2acme_hostname: the.hostname.of.your.acme.proxy
            acme2acme_allowed_domainlist: '["*.allowed.subdomain.example.com"]'
            acme2acme_acme_url: https://acme-v02.harica.gr/acme/XXX-YYY-ZZZ-HHH-JHKJKKJHK/
            acme2acme_eab_kid: your-key-id
            acme2acme_eab_hmac_key: your-eab-secret
            acme2acme_tos_url: https://repo.harica.gr/documents/SA-ToU_EN.pdf
            acme2acme_debug: True
```

Execute the playbook using your environment-specific configuration:

```bash
ansible-playbook deploy.yaml -i inventory.yaml
```