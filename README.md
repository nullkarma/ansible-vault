# Role Name

Manage HashiCorp Vault with Ansible

## Capabilities

### Auto initialize and unseal Vault

Setting `vault_auto_init: true` will instruct Ansible to perform a `vault operator init` and `vault operator unseal` through the Vault API on the very first install. Once Vault is unsealed, it will create a KV secret called `root`, store the generated unseal keys and root token in it, and use the root token to create all defined policies.
If `vault_token` is set and `vault_auto_init` is enabled, vault will ignore the user provided `vault_token` during the very first run. Every consecutive `ansible-playbook` run will honor `vault_token`.
This operation is performed only on `initialized: false` ([sys/seal-status](https://www.vaultproject.io/api/system/seal-status.html)). See `tasks/api/init.yml` for more details.

### Policy management

Ansible will deploy all policies defined in `vault_policies`. `vault_token` is required.
Format is JSON compatible HCL written in YAML.

```yaml
vault_policies:
- name: mypolicy
  policies:
    path:
    - "foo":
        capabilities:
        - read
    - "bar/*":
        capabilities:
        - create
        - read
        - update
        - delete
```

# Role Variables

|Parameter|Default|Description|
|---------|-------|-----------|
|vault_version|`0.11.5`||
|vault_base_url|`https://releases.hashicorp.com/vault/{{ vault_version }}`||
|vault_user|`vault`||
|vault_group|`vault`||
|vault_home_dir|`/opt/vault`||
|vault_conf_dir|`/etc/vault`||
|vault_data_dir|`/opt/vault/data`||
|vault_manage_service|`true`|Manage Service start and restart|
|vault_restart_on_change|`true`|Service restart on config change|
|vault_host|`{{ ansible_fqdn }}`||
|vault_auto_init|`true`|Let Ansible initialize and unseal vault. Unseal keys will be writte into `root` kv secret|
|vault_config|see `defaults/main.yml`|JSON config written in yaml. Consult Vault documentation for parameters|
|vault_policies|see example for syntax|JSON Vault Policies|
|vault_init_parameters||`secret_shares: 5`<br>`secret_threshold: 3`||
|vault_token|nil|If set, Ansible will use the token to create policies|


# Dependencies

None

# Example Playbook

```yaml
- hosts: vault
  tasks:
  - import_role:
      name: vault
  vars:
    vault_config:
      default_lease_ttl: 168h
      max_lease_ttl: 720h
      disable_mlock: false
      ui: true
      listener:
        tcp:
          address: 0.0.0.0:8200
          tls_disable: true
      storage:
        file:
          path: "{{ vault_data_dir }}"
    vault_policies:
    - name: root-secret-read
      policies:
        path:
        - "root/*":
            capabilities:
            - read
```

# License

BSD

# Author Information

see `meta/main.yml`
