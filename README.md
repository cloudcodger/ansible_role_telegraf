# Ansible role `telegraf` for Influxdata.

This is an Ansible role for the [Influxdata](https://www.influxdata.com/) [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) data collector.

Install and configure the `telegraf` data collector on a system for sending data to `influxdb`.

# Requirements

## Influxdata APT repository key

The installation instruction for `telegraf` include downloading a key file from a URL, validating its checksum, changing it to a binary format, and placing it in `/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg`. These steps require that `sudo` and `pgp` be installed on the system. Rather than put extra packages on systems that will only use it once, the PGP file must be downloaded and placed in the `files` directory, where we have already verified the checksum and converted it to the binary format. If the Influxdata APT repository ever changes this "key", it will need to be updated in this role.

The commands used to get `files/influxdata-archive.gpg`;

```bash
wget https://repos.influxdata.com/influxdata-archive.key
echo "943666881a1b8d9b849b74caebf02d3465d6beb716510d86a39f6c8e8dac7515  influxdata-archive.key" | sha256sum -c -
cat influxdata-archive.key | gpg --dearmor -o influxdata-archive.gpg
rm influxdata-archive.key # or this can be left there, if desired.
```

The instructions also say to place the `influxdata-archive.gpg`, in `/etc/apt/trusted.gpg.d`, where the [Debian Wiki](https://wiki.debian.org/DebianRepository/UseThirdParty) indicates that these should be placed in `/usr/share/keyrings` and is the location this role will use.

# Role Variables

Variables with default values that likely need to be set to the correct values for the calling playbook. The defaults may allow you to test what the role is doing when first using it.

- `telegraf_config_global_tags` - A multiline string containing all the desired tags.
- `telegraf_config_agent` - A multiline string containing all the desired `agent` settings.
- `telegraf_config_outputs` - A list of output plugins to configure (default: `[file]`).
- `telegraf_config_output_settings_<plugin>` - A multiline string containing all the desired settings for each `<plugin>`. Ignored if `telegraf_config_outputs` does not include `<plugin>`.
- `telegraf_config_inputs` - A list of input plugins to configure (default: `[cpu, disk, kernel, mem, net, processes, system]`).
- `telegraf_config_input_settings_<plugin>` - A multiline string containing all the desired settings for each `<plugin>`. Ignored if `telegraf_config_inputs` does not include `<plugin>`.

# Example Playbook

This example shows the output being changed from the `file` default, to `influxdb_v2` and the 

```yaml
---
- name: Install and configure 'telegraf'.
  become: true
  hosts: host_group

  roles:
    - role: telegraf
      telegraf_config_global_tags: |
        cloud = "MyColo"
        cluster = "{{ inventory_hostname }}"
        env = "prod"
        region = "us-west"
      telegraf_config_outputs: [influxdb_v2]
      telegraf_config_output_settings_influxdb_v2: |
        bucket = "My_bucket_name"
        organization = "My Org Name"
        token = "{{ lookup('file', '~/.secrets/influxdb-Founders.token') }}"
        urls = ["https://influxdb.example.com/"]
      telegraf_config_inputs: [kernel, mem, net, processes, system, cpu, disk, systemd_units]
      telegraf_config_input_settings_systemd_units: |
        timeout = "5s"
        unittype = "service"
        pattern = "service_prefix*"
```
