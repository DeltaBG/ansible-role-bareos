# ansible-role-bareos

Role for installing and managing BareOS director, storages and client.

## Requirements

This role requires you to have Apache and PostgreSQL (>=12.10) preinstalled on the director host. Installation and configuration of these pacakges is not subject of this role. Also, the hostgroup in your inventory file which contains the director should be named `bareos_director`.

## Role variables

All role variables are documented in role defaults or are self-explanatory.

## Example playbooks

### Configuring all components all at once (site.yml)

```yaml
---
- hosts: all
  roles: 
    - deltabg.bareos

```

Running the playbook: `ansible-playbook -i inventory_file.ini site.yml`

### Adding a new client (site.yml)

```yaml
---
- hosts: all
  roles: 
    - deltabg.bareos

```

Running the playbook: `ansible-playbook -i inventory_file.ini site.yml -t bareos-add-client --limit=<bareos_client>`

### Removing a client (remove-client.yml)

```yaml
- name: Remove a bareos client configuration and backups
  hosts: bareos_director
  handlers:
    - name: reload bareos-dir
      shell: "echo reload | bconsole"
      delegate_to: "{{ groups.bareos_director[0] }}"
  tasks:
    - name: Remove client configuration
      file:
        path: "{{ bareos_config_dir }}/bareos-dir.d/client/{{ bareos_remove_client }}.conf"
        state: absent
      notify: reload bareos-dir
    - name: Remove console configuration
      file:
        path: "{{ bareos_config_dir }}/bareos-dir.d/console/{{ bareos_remove_client }}.conf"
        state: absent
      notify: reload bareos-dir
    - name: Remove job configuration
      file:
        path: "{{ bareos_config_dir }}/bareos-dir.d/job/{{ bareos_remove_client }}.conf"
        state: absent
      notify: reload bareos-dir
    - name: Remove profile configuration
      file:
        path: "{{ bareos_config_dir }}/bareos-dir.d/profile/{{ bareos_remove_client }}.conf"
        state: absent
      notify: reload bareos-dir
    - name: Remove volumes
      shell: "for volume in $(echo list media | bconsole | grep {{ bareos_remove_client }} | awk '{print $4}' | grep -ve '^$'); do echo delete volume=$volume yes | bconsole; done"
    - name: Remove pools
      shell: "echo delete pool={{ bareos_remove_client }} yes | bconsole"

- hosts: bareos_storages
  tasks:
    - name: Physically remove pools from remote storage
      shell: "rm -f {{ bareos_storage_bareos_archive_device }}/{{ bareos_remove_client }}*"
```

Running the playbook: `ansible-playbook -i inventory_file.ini remove-client.yml -e 'bareos_remove_client=<client_from_inventory_to_remove>'`

Note: you will need to remove the client configuration from your inventory file and host_vars afterwards.

### Adding a new remote storage (site.yml)


```yaml
---
- hosts: all
  roles: 
    - deltabg.bareos

```

Running the playbook: `ansible-playbook -i inventory_file.ini site.yml -t bareos-add-client --limit=<bareos_storage_server>,bareos_director`

### Reconfiguring the director in case of new configuration (site.yml)

```yaml
---
- hosts: all
  roles: 
    - deltabg.bareos

```

Running the playbook: `ansible-playbook -i inventory_file.ini site.yml -t bareos-director-reconfigure --limit=bareos_director`

## License

See LICENSE for more information.

## Author

[Valentin Dzhorov](https://github.com/vdzhorov)
