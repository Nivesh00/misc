# Snipe IT

Add your variables to the `inventory.yml` file before running any ansible command

> [!IMPORTANT]
> Cert Manager is needed for TLS cert management

## Deploy Cert Manager
> Inventory from this repo can be used to deploy Cert Manager

- Clone git repository
```bash
git clone https://gitlab.com/urban-dataspace-platform/core-platform.git
```

- Run playbook with necessary tags
```bash
ansible-playbook 02_core_platform/full_install.yml --tags operator_cert_manager
```

## Deploy Snipe IT
Ansible playbook to deploy MariaDB and Snipe IT

```bash
ansible-playbook -l localhost -i inventory playbook.yml
```