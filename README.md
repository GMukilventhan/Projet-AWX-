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
- Output
```bash
root@awx:/etc# ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.6 (main, Mar 10 2023, 10:55:28) [GCC 11.3.0]
  ```
  
  ## 2- Installer Docker
  
  - Installer les paquets nécessaires, importer la clé GPG et ajouter le dépôt aux sources APT

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg-agent
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

- Mettre à jour le nouveau dépôt et installer Docker depuis son dépôt

```bash
sudo apt update
apt-cache policy docker-ce
```

- Installer docker-ce

```bash
sudo apt install docker-ce
```

- Vérifier l'installation
```bash
root@awx:/etc# sudo systemctl status docker
```
- Output
```bash
root@awx:/etc# sudo systemctl status docker
docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-04-15 19:45:33 UTC; 6min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 962 (dockerd)
      Tasks: 19
     Memory: 109.4M
        CPU: 2.407s
     CGroup: /system.slice/docker.service
             └─962 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
  ```

## 3- Installer Node/NPM

```bash
sudo apt install -y nodejs npm
sudo npm install npm --global
```

## 4- Installer Ansible AWX

```bash
sudo apt install python3-pip git pwgen vim
sudo pip3 install requests==2.22.0
```

- Récupérer un module docker-compose pour Python qui correspond à votre version

```bash
root@awx:/etc# docker-compose version
docker-compose version 1.29.2, build unknown
docker-py version: <module 'docker.version' from '/usr/local/lib/python3.10/dist-packages/docker/version.py'>
CPython version: 3.10.6
OpenSSL version: OpenSSL 3.0.2 15 Mar 2022
```

- La version que j'ai installée est la 1.29.2

```bash
sudo pip3 install docker-compose==1.29.2
```

- Téléchargez le zip AWX depuis Github, décompressez, installez et générez le secret

```bash
wget https://github.com/ansible/awx/archive/17.1.0.zip
unzip 17.1.0.zip
cd awx-17.1.0/installer/
pwgen -N 1 -s 30
```


- Modifiez ensuite le fichier d'inventaire et ajoutez la clé secrète que vous venez de générer

```bash
sudo nano inventory
```
