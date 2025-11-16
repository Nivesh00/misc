# Network Mapper

> See file [README_NETWORK_MAPPER.md](./README_NETWORK_MAPPER.md)

# Folder structure:

```
├── README.md
├── README_NETWORK_MAPPER.md
├── default_inventory.yml
├── playbook.yml
├── tasks/
├── templates/
├── traffic-png
│   └── <stack>.png
└── vars
    ├── 00_default.yml
    └── network_policy
        ├── 00_non_tenant.yml
        └── 01_tenant.yml
```

- `README_NETWORK_MAPPER.md`: explains why [Otterize Network Mapper](https://docs.otterize.com/reference/network-mapper) was used and how to use it
- `default_inventory.yml`: Ansible Inventory
- `playbook.yml`: Ansible Playbook
- `tasks` dir: tasks for Ansible Playbook
- `templates` dir: templates file for network policies
- `traffic-png` dir: directory for traffic maps created by Otterize Network Mapper. PNGs are named after the namespace they map
- `vars` dir: directory for Ansible variable files
    - `network_policy` dir: directory with files where pod labels and selectors are set 

## `vars/network_policy/` Directory

- Network policy configurations are added and change in files found in this directory

- `00_non_tenant.yml` has all configurations for components that are __not__ tenants

- `01_default_tenant.yml` has all configurations for components that are tenants

> Note that network policy configuration file can be used per tenant instead of the default configuration file `01_default_tenant.yml`

- To do so, in inventory file, set `use_default_policy` to false

    ```yml
    ## inventory.yml
    ...
        inv_network_policy:
          enable: true
          use_default_policy: false
    ...
    ```

- __IMPORTANT:__ if `use_default_policy` is set to false, the configuration file for the tenant must have the exact same name as `<tenant_realm>.yml`, e.g.

    - for following configurations
    ```yml
    ## inventory.yml
    ...
        ## Tenants
        inv_tenants:
          - tenant_realm: "berlin-realm" # Leader tenant
            tenant_domain: "{{ DOMAIN }}" # change if tenant uses other domain as operations
            tenant_prefix: "berlin" # prefix for tenant K8s namespaces
    ...
    ```

    - network policy configuration file for this tenant must be named `vars/network_policy/berlin-realm.yml`

# Steps to create network policies

- Use Otterize Network Mapper to map connections in a namespace

- Create concept for network policy using Kubernetes docs [kubernetes.io - network-policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

- Add configurations to `vars/00_non_tenant.yml` for a component that is not a tenant, or add configurations to `vars/00_default_tenant.yml` (or custom file if using one configuration file per tenant) if component is a tenant

- Test config by running playbook

    ```sh
    ansible-playbook -i inventory.yml playbook.yml
    ```

## Configuration File

Structure of configuration file:

```yml
data:
  - stack_component: <name-of-component> # can be freely chosen
    stack_ns: <namespace-of-component>
    components:
      - name: <name-of-policy> # use name of workload here e.g. name shown in the traffic PNG
        pod_labels:
          - name: <name-of-label>
            value: <value-of-label>
        ingress:
          - ns_name: <source-namespace-of-traffic>
            pod_labels:
              name: <label-name-of-source-pod>
              value: <label-value-of-source-pod>
        egress:
          - ns_name: <destination-namespace-of-traffic>
            pod_labels:
              name: <label-name-of-destination-pod>
              value: <label-value-of-destination-pod>
```

- Notes:
    - `data[].components[].pod_labels[]` cannot be empty
    - `data[].components[].ingress[]` and `data[].components[].egress[]` may be empty, but not omitted

# Notes

- Playbook assumes `core-platform` is already deployed [gitlab.com - HTAG core-platform](https://gitlab.com/urban-dataspace-platform/core-platform/-/tree/master?ref_type=heads)

- Playbook can be easily integrated into `core-platform` playbook
