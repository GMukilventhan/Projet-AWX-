# Installation et configuration AWX - GitLab 

# AWX

## installation étape par étape pour Ansible AWX sur Ubuntu 20.4 avec Docker


## Prérequis :

- Privilèges sudo ou root pour SSH
- Min 4 Go de RAM, 2vCPU et 20 Go de stockage
- Docker et Docker Compose
- Node et NPM

## 1 - Installer Ansible

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
  
  ## 2 - Installer Docker
  
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

## 3 - Installer Node/NPM

```bash
sudo apt install -y nodejs npm
sudo npm install npm --global
```

## 4 - Installer Ansible AWX

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

Faites défiler la page et modifiez le champ secret_key. J'ai également activé le champ du mot de passe qui est normalement haché

## 5 - Déployer AWX

```bash
ansible-playbook -i inventory install.yml
```

Aucune tâche ne doit avoir échoué. Vérifiez que les conteneurs Docker sont en place avec les éléments suivants

```bash
docker ps
```

Output 

```bash
root@awx:/etc/awx-17.1.0/installer# docker ps
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS                                   NAMES
7a44440c353e   ansible/awx:17.1.0   "/usr/bin/tini -- /u…"   53 minutes ago   Up 53 minutes   8052/tcp                                awx_task
acdf1e1aa184   ansible/awx:17.1.0   "/usr/bin/tini -- /b…"   53 minutes ago   Up 53 minutes   0.0.0.0:80->8052/tcp, :::80->8052/tcp   awx_web
958e6fc63653   postgres:12          "docker-entrypoint.s…"   53 minutes ago   Up 53 minutes   5432/tcp                                awx_postgres
89b2aaed9bc2   redis                "docker-entrypoint.s…"   53 minutes ago   Up 53 minutes   6379/tcp                                awx_redis
```

Vous pouvez accéder à l'interface Web d'Ansible AWX via hostip ou hostname sur le port 80. Connectez-vous avec les informations d'identification définies dans le fichier d'inventaire.


# GitLab 

## Pour commencer

Avant d'installer GitLab, il est recommandé de mettre à jour et de mettre à niveau tous les paquets système vers la version actualisée. Vous mettez à jour tous les paquets à l'aide de la commande suivante.

```bash
apt update -y
apt upgrade -y
```
Une fois que tous les paquets ont été mis à jour, vous pouvez ajouter le dépôt GitLab.

## 1 - Ajouter un dépôt GitLab

Par défaut, le paquetage GitLab n'est pas inclus dans le dépôt par défaut d'Ubuntu. Vous devez donc ajouter le dépôt officiel de GitLab à APT.

Tout d'abord, installez toutes les dépendances nécessaires à l'aide de la commande suivante

```bash
apt install curl debian-archive-keyring lsb-release ca-certificates apt-transport-https software-properties-common -y
```

Une fois toutes les dépendances installées, ajoutez la clé GPG de GitLab avec la commande suivante

```bash
gpg_key_url="https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey"
curl -fsSL $gpg_key_url| gpg --dearmor -o /etc/apt/trusted.gpg.d/gitlab.gpg
```

Ensuite, ajoutez le dépôt GitLab à APT avec la commande suivante

```bash
nano /etc/apt/sources.list.d/gitlab_gitlab-ce.list****
```

Ajouter la ligne suivante

```bash
deb https://packages.gitlab.com/gitlab/gitlab-ce/ubuntu/ focal main
deb-src https://packages.gitlab.com/gitlab/gitlab-ce/ubuntu/ focal main
```

Enregistrez et fermez le fichier, puis mettez à jour le référentiel avec la commande suivante

```bash
apt update -y
```

## 2 - Installer GitLab

Vous pouvez maintenant installer GitLab en exécutant la commande suivante

```bash
apt install gitlab-ce -y
```

## 3 - Configurer GitLab

Ensuite, vous devez éditer le fichier de configuration de GitLab et définir l'URL de votre domaine pour y accéder via internet.

```bash
nano /etc/gitlab/gitlab.rb
```

Modifier la ligne suivante :

```bash
external_url 'http://gitlab.yourdomain.com'
```

Sauvegardez et fermez le fichier puis reconfigurez le GitLab avec la commande suivante.

```bash
gitlab-ctl reconfigure
```

Vous pouvez maintenant vérifier la configuration de GitLab à l'aide de la commande suivante.

```bash
gitlab-rake gitlab:check
```

## 4 - Accéder à GitLab

A ce stade, GitLab est installé et configuré. Pour accéder à l'interface web de GitLab, vous devrez récupérer le mot de passe root de GitLab pour vous connecter à GitLab.

```bash
cat /etc/gitlab/initial_root_password
```

Output
```bash
root@git:/etc/apt/sources.list.d# cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: Ym0cHhYK/ENJM8X/aMdo94SuNAhWMBVgb1C/U1fAUNY=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```
---------------------------------------------------------------------------------------------------------------------------------------------------------
sur la machine awx / copier la clés
```bash
cat ~/.ssh/id_rsa.pub
```

sur les machines clients coller la clés dans le repertoire 
```bash
nano ~/.ssh/authorized_keys
```


