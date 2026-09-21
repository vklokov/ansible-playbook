### Install

apt update && apt install ansible

# sudo apt install pipx
# pipx install --include-deps ansible

### Add plugins
ansible-galaxy collection install ansible.posix

### Run playbook
ansible-playbook playbook.yml