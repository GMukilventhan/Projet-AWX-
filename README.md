# AWX - GitLab 

# AWX

## installation étape par étape pour Ansible AWX sur Ubuntu 20.4 avec Docker


## Prérequis :

- Privilèges sudo ou root pour SSH
- Min 4 Go de RAM, 2vCPU et 20 Go de stockage
- Docker et Docker Compose
- Node et NPM

## 1- Installer Ansible

```bash
sudo apt update
sudo apt install ansible -y
```

- Vérifier l'installation
```bash
ansible --version
```
- Ouput
```bash
root@awx:/etc# ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.6 (main, Mar 10 2023, 10:55:28) [GCC 11.3.0]
  ```
