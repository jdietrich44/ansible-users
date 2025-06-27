# Ansible User Management Playbook

This project demonstrates user and SSH key management using Ansible. It allows you to define, create, and remove users across different server groups, ensuring consistent and secure access control.

## Features

- Define and create users with custom shells, groups, and sudo privileges
- Deploy SSH authorized keys for users
- Remove users and clean up home directories and SSH keys
- Assign different users and groups to different server groups using Ansible's inventory and group variables
- **Dynamic privilege management:** Sudoers files are created or deleted automatically based on group variable settings
- **Automatic, safe cleanup:** Orphaned or outdated sudoers files are removed when privilege settings change
- **Variable-driven design:** All configuration is controlled through easy to edit group variables, making the playbook flexible and reusable
- **Clear documentation and logic matrix:** Includes a privilege matrix/table and thorough documentation for easy understanding and onboarding
- **Supplementary group management:** Assign users to one or more additional groups using the `groups` variable. Groups are created automatically if they do not exist

## Usage

1. Clone the repo

```bash
git clone <your-repo-url>
cd <your-repo-directory>
```

2. Edit your inventory file

  - Modify `inventory/inventory.ini` to define your hosts and their groups (e.g., `[web]`, `[local]`)

3. Configure group variables

  - Edit `group_vars/<group>.yml` (e.g., `group_vars/web.yml`, `group_vars/local.yml`) to define `user_groups`, `users`, and `deleted_users` for each group.

4. Run the playbook

```bash
ansible-playbook playbooks/users.yml
```

- To perform a dry run (no changes made):

```bash
ansible-playbook playbooks/users.yml -C
```

## Sudoers File Management Matrix

The `sudo` and `restricted_sudo` values referenced in this matrix are specified in your `group_vars/<group>.yml` files. This matrix can be used to understand the logic of how sudoers files get created or deleted based on these values.

| sudo  | restricted_sudo | Created? | Deleted? |
|-------|-----------------|----------|----------|
| true  | false           | Yes      | No       |
| false | true            | Yes      | No       |
| true  | true            | Yes      | No       |
| false | false           | No       | Yes      |
| (unset)| (unset)        | No       | Yes      |

## Groups logic for users

- If `groups` is set in `group_vars/<group>.yml` for a user, the user will be added to those groups (and the groups will be created if missing).
- If `groups` is not set, the user will only have their primary group.
- The playbook only sets `append:yes` when `groups` is defined, avoiding Ansible warnings or errors in newer versions.

| `groups` defined? | User added to supplementary groups? | Groups created if missing? |
|-------------------|-------------------------------------|---------------------------|
| Yes               | Yes                                 | Yes                       |
| No                | No                                  | N/A                       |

## Supplementary Groups Warning

For system services like Jenkins, let the installer create the `jenkins` group. Only add users to service groups like `jenkins` after Jenkins is installed, or ensure group creation does not conflict with the installer's expectations. Avoid pre-creating groups for system services unless you are certain it is safe.

- Avoid using service groups (like `jenkins`) as supplementary groups in your user definitions unless the service is already installed.
- If you want to add users to service groups, ensure the service group is created by the service installer first.
- When using supplementary groups, users will be added to new groups specified, but will not be removed from groups if those groups are removed from the configuration. To enforce exact group membership, remove append: yes, but be aware this will remove users from all supplementary groups not listed.

## Project Structure

```plaintext
ansible-users
├── README.md
├── ansible.cfg
├── inventory
│   └── inventory.ini
└── playbooks
    ├── group_vars
    │   ├── local.yml
    │   └── webserver.yml
    ├── roles
    │   └── users
    │       ├── files
    │       │   └── ssh_keys
    │       │       ├── jeff.pub
    │       │       └── julie.pub
    │       ├── tasks
    │       │   └── main.yml
    │       └── templates
    │           ├── group.j2
    │           └── user.j2
    └── users.yml
```

## Notes

- You can assign hosts to multiple groups in the inventory, but be aware of Ansible's variable precedence rules.
- Use `--limit <group>` to target specific groups during testing or deployment.
