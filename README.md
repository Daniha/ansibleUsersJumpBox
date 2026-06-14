# Ansible VM Configuration

This Ansible playbook configures a VM at `192.168.162.129` with the following setup:

## Prerequisites

- Ansible user with sudo privileges (already in wheel group)
- SSH access to the VM
- Python 3 installed on the target VM
- If firewalld is running, port 22 (SSH) will be automatically opened

## Configuration Details

The playbook will:

1. **Check and configure firewall**: If firewalld service is active, open port 22 using:
   ```bash
   firewall-cmd --permanent --add-service=ssh
   firewall-cmd --reload
   ```

2. **Create users**: danha, msilva, corazoncito
   - All users will be added to the wheel group for sudo privileges

3. **Generate SSH keys**: Ed25519 SSH keys for each user at `~/.ssh/id_ed25519`

4. **Create initial passwords**: A random password is generated for each user and expires on first login. The playbook displays each user's initial password at runtime and saves them to `initial_user_passwords.txt` on the controller.

5. **Create SSH config**: Each user will have `~/.ssh/config` configured with:
   ```
   Host myserver
       HostName 192.168.162.128
       User <username>
       IdentityFile ~/.ssh/id_ed25519
   ```

## Running the Playbook

### 1. Ensure SSH connectivity to the VM:
```bash
ssh ansible@192.168.162.129
```

### 2. Run the playbook:
```bash
ansible-playbook configure_vm.yml -k
```

### 3. Verify the configuration:
Connect to the VM and check:
```bash
ssh ansible@192.168.162.129

# As ansible user, check the created users
id danha
id msilva
id corazoncito

# Verify SSH keys
ls -la /home/danha/.ssh/
cat /home/danha/.ssh/config
```

## Directory Structure

```
ansibleJumpBox/
├── ansible.cfg              # Ansible configuration
├── configure_vm.yml         # Main playbook
├── inventory/
│   └── hosts               # Inventory file with VM IP
├── group_vars/
│   └── all.yml            # Group variables (users list)
└── roles/
    └── vm_setup/
        ├── tasks/
        │   └── main.yml   # Main tasks
        └── templates/
            └── ssh_config.j2  # SSH config template
```

## Troubleshooting

- **SSH key authentication issues**: Ensure the ansible user has valid SSH credentials
- **Permission denied for sudo**: Verify the ansible user is in the wheel group
- **Firewall issues**: Manually run the firewall commands if automation fails

## Notes

- The SSH config sets `HostName 192.168.162.128` (note: different from the VM IP 192.168.162.129)
- Each user gets their own SSH key and config file
- All SSH files are created with proper permissions (0600 for keys, 0700 for .ssh directory)
