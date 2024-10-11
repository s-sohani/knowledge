In Ansible, there are several built-in variables and keywords that are available for use in playbooks, templates, and tasks. These keywords give access to information about the hosts, the environment, the inventory, and more. Below is a list of commonly used Ansible keywords like `inventory_hostname` or `ansible_host`, categorized by their purpose.

### 1. **Host and Inventory Variables**

These variables are related to inventory and the hosts being managed:

- **`inventory_hostname`**: The name of the current host as listed in the Ansible inventory.
- **`inventory_hostname_short`**: The short version of the current hostname (everything before the first dot).
- **`ansible_host`**: The actual IP address or hostname Ansible uses to connect to the host (if different from `inventory_hostname`).
- **`groups`**: A dictionary of all groups in the inventory, where each key is a group name and each value is a list of hosts in that group.
- **`group_names`**: A list of all the groups the current host belongs to.
- **`hostvars`**: A dictionary of all the hosts in the inventory and their associated variables, which allows you to access information about any host.
- **`play_hosts`**: The list of hosts in the play.

### 2. **Facts (Gathered Information about Hosts)**

When facts gathering is enabled, Ansible collects a wealth of information about the host (e.g., IP addresses, OS, hardware). These facts can be accessed using the `ansible_facts` or individual fact variables:

- **`ansible_facts`**: A dictionary containing all the facts gathered about the host.
- **`ansible_default_ipv4.address`**: The default IPv4 address of the host.
- **`ansible_default_ipv6.address`**: The default IPv6 address of the host.
- **`ansible_distribution`**: The name of the operating system distribution (e.g., `Ubuntu`, `CentOS`).
- **`ansible_os_family`**: The family of the OS (e.g., `Debian`, `RedHat`).
- **`ansible_kernel`**: The kernel version of the host.
- **`ansible_hostname`**: The short hostname of the host.
- **`ansible_fqdn`**: The fully qualified domain name of the host.
- **`ansible_memory_mb`**: Information about the memory of the host in megabytes.
- **`ansible_processor`**: Information about the host’s processor.

### 3. **Playbook and Task Context Variables**

These variables provide context about the current playbook or task:

- **`ansible_playbook_dir`**: The path of the directory where the playbook is located.
- **`ansible_play_hosts`**: A list of all the hosts in the current play.
- **`ansible_play_batch`**: The set of hosts currently being executed in a batch (when using `serial`).
- **`playbook_dir`**: The directory where the playbook is located (similar to `ansible_playbook_dir`).
- **`ansible_role_names`**: A list of the names of the roles used in the current play.
- **`ansible_check_mode`**: Indicates whether the playbook is running in check mode (`true` or `false`).
- **`ansible_diff_mode`**: Indicates whether the playbook is running in diff mode (`true` or `false`).

### 4. **Connection Variables**

These variables control how Ansible connects to hosts:

- **`ansible_user`**: The username Ansible uses to connect to the remote host.
- **`ansible_ssh_pass`**: The SSH password used to connect to the remote host (if applicable).
- **`ansible_port`**: The SSH port used to connect to the host.
- **`ansible_ssh_private_key_file`**: The private key file used for SSH authentication.

### 5. **Loop Variables**

These variables are used when working with loops in Ansible:

- **`item`**: The current item in a loop.
- **`ansible_loop`**: Provides context about the current iteration in a loop (e.g., `index`, `index0`, `first`, `last`).

### 6. **Special Variables**

These are special variables that control certain behaviors or are used for debugging:

- **`hostvars`**: A dictionary containing variables of all hosts in the inventory.
- **`vars`**: The variables defined for the current host, play, or task.
- **`omit`**: A special variable used to indicate that a parameter should be omitted (useful for optional arguments).
- **`ansible_failed_task`**: Information about the last failed task in case of a failure.
- **`ansible_facts`**: A dictionary of gathered facts about the host.
- **`ansible_version`**: The version of Ansible being used.

### 7. **Variables for Ansible Playbook Structure**

- **`role_path`**: The full path to the currently executing role.
- **`inventory_dir`**: The directory of the inventory file being used.
- **`inventory_file`**: The path to the inventory file.

### 8. **Delegation and Host-Specific Variables**

When dealing with delegated tasks (e.g., using `delegate_to`), there are specific variables:

- **`ansible_delegated_vars`**: Contains variables related to delegation.
- **`delegate_to`**: Specifies the host to which a task is delegated.

### Example Playbook Using Some of These Variables:
```yaml
---
- hosts: all
  tasks:
    - name: Show the current host and its IP address
      debug:
        msg: "Host: {{ inventory_hostname }} with IP: {{ ansible_host }}"

    - name: Show the default IPv4 address
      debug:
        msg: "Default IPv4 address: {{ ansible_default_ipv4.address }}"

    - name: Show the current playbook directory
      debug:
        msg: "Playbook directory: {{ ansible_playbook_dir }}"

    - name: Show all hosts in the webservers group
      debug:
        msg: "Hosts in webservers group: {{ groups['webservers'] }}"

```

This list is not exhaustive, but it covers the most commonly used keywords and variables in Ansible. You can explore more through Ansible documentation on variables.