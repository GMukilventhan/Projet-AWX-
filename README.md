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
sudo apt update && sudo apt install ansible -y
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
🚨n'oubliez pas d'avoir le fichier vault ou sinon la créer 

--------------------------------------------------------------------------------------------------------------------------------------------------------
Le projet se trouve sur gitlab: 
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 23 39" src="https://user-images.githubusercontent.com/90500004/232628086-f164d6d2-7d6f-4a5a-9262-ffc030f4139e.png">

<img width="1904" alt="Capture d’écran 2023-04-18 à 00 50 32" src="https://user-images.githubusercontent.com/90500004/232627161-ad493f33-e90d-4e3f-9b39-32a7fdcce0fc.png">


Ajouts des élements sur awx:
Inventories:
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 53 11" src="https://user-images.githubusercontent.com/90500004/232628171-d622d5db-ccf1-44b9-9ad7-f0d61b6dc9c6.png">
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 53 45" src="https://user-images.githubusercontent.com/90500004/232628606-a4987b71-07cc-46d7-8b58-47d59f3cb66c.png">
Groups:
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 54 09" src="https://user-images.githubusercontent.com/90500004/232628226-fc12ff47-2ba6-4726-ad12-54f3c6e2b5c9.png">

Hosts:
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 54 52" src="https://user-images.githubusercontent.com/90500004/232628245-40efc291-fa4c-4a4c-986a-71ebdb8b2a70.png">
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 55 06" src="https://user-images.githubusercontent.com/90500004/232628276-bdabb606-6df7-4d5d-8d77-24b4467b00c0.png">

Projects
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 55 25" src="https://user-images.githubusercontent.com/90500004/232628313-7d34f5cd-8882-43a3-96c0-11d2c1f7f773.png">

Credentials
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 55 34" src="https://user-images.githubusercontent.com/90500004/232628329-0c62ee07-76ff-430c-bf7d-6a318c1ba67f.png">

Templates:
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 55 55" src="https://user-images.githubusercontent.com/90500004/232628364-1a1c9f68-5170-4632-b024-fce5068c3dd7.png">

Notification:
<img width="1904" alt="Capture d’écran 2023-04-18 à 00 56 28" src="https://user-images.githubusercontent.com/90500004/232628409-2c9a5a1f-dfb9-43cf-8643-d6c328b54c89.png">

<img width="1904" alt="Capture d’écran 2023-04-18 à 00 56 58" src="https://user-images.githubusercontent.com/90500004/232628422-1769afcc-c2df-4c58-9ce5-74bb034ca88f.png">

<img width="1904" alt="Capture d’écran 2023-04-18 à 00 57 21" src="https://user-images.githubusercontent.com/90500004/232628663-e522a536-2f5c-49ab-bea0-bfe53823e619.png">

<img width="1904" alt="Capture d’écran 2023-04-18 à 01 05 31" src="https://user-images.githubusercontent.com/90500004/232628879-faa719ce-cb7e-4962-ba00-6ebc2106ec7a.png">


<img width="1904" alt="Capture d’écran 2023-04-18 à 00 58 46" src="https://user-images.githubusercontent.com/90500004/232628679-f979c795-1be9-4b0e-9695-e03037a9771b.png">


```bash
Identity added: /tmp/bwrap_131_hoooi_b1/awx_131_t657dtwv/artifacts/131/ssh_key_data (root@ubuntu03)
SSH password: 
Vault password: 

PLAY [CLIENT] ******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [apache2 : Installation Apache2] ******************************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [show result_apache2] *****************************************************
ok: [172.20.20.12] => {
    "result_apache2": {
        "cache_update_time": 1681718990,
        "cache_updated": false,
        "changed": false,
        "failed": false
    }
}
ok: [172.20.20.14] => {
    "result_apache2": {
        "cache_update_time": 1681738976,
        "cache_updated": false,
        "changed": false,
        "failed": false
    }
}

TASK [apache2 : Already installed] *********************************************
ok: [172.20.20.12] => {
    "msg": "Apache2 is already installed"
}
ok: [172.20.20.14] => {
    "msg": "Apache2 is already installed"
}

TASK [apache2 : Restart Apache2] ***********************************************
skipping: [172.20.20.12]
skipping: [172.20.20.14]

TASK [apache2 : Check that the URL responds] ***********************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [apache2 : debug message] *************************************************
ok: [172.20.20.12] => {
    "msg": "The URL answers"
}
ok: [172.20.20.14] => {
    "msg": "The URL answers"
}

TASK [apache2 : Error message] *************************************************
skipping: [172.20.20.12]
skipping: [172.20.20.14]

TASK [apache2 : Restart service after URL ko] **********************************
skipping: [172.20.20.12]
skipping: [172.20.20.14]

TASK [copy : Copy the esgi.jpg image to the /var/www/html folder] **************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [copy : Copy index.j2] ****************************************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [Installation of ntp] *****************************************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [show result_ntp] *********************************************************
ok: [172.20.20.12] => {
    "result_ntp": {
        "cache_update_time": 1681718990,
        "cache_updated": false,
        "changed": false,
        "failed": false
    }
}
ok: [172.20.20.14] => {
    "result_ntp": {
        "cache_update_time": 1681738976,
        "cache_updated": false,
        "changed": false,
        "failed": false
    }
}

TASK [ntp : Already installed] *************************************************
ok: [172.20.20.12] => {
    "msg": "ntp is already installed"
}
ok: [172.20.20.14] => {
    "msg": "ntp is already installed"
}

TASK [ntp : check file exist] **************************************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [Creation of the ntp file] ************************************************
skipping: [172.20.20.12]
skipping: [172.20.20.14]

TASK [ntp : Add NTP servers with a list and loop in the file] ******************
ok: [172.20.20.12] => (item=30.30.30.30)
ok: [172.20.20.14] => (item=30.30.30.30)
ok: [172.20.20.12] => (item=14.14.14.14)
ok: [172.20.20.14] => (item=14.14.14.14)
ok: [172.20.20.12] => (item=5.5.5.5)
ok: [172.20.20.14] => (item=5.5.5.5)
ok: [172.20.20.12] => (item=8.8.8.8)
ok: [172.20.20.14] => (item=8.8.8.8)

TASK [ntp : check  file update] ************************************************
skipping: [172.20.20.12] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 30.30.30.30', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '30.30.30.30', 'ansible_loop_var': 'item'}) 
skipping: [172.20.20.12] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 14.14.14.14', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '14.14.14.14', 'ansible_loop_var': 'item'}) 
skipping: [172.20.20.12] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 5.5.5.5', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '5.5.5.5', 'ansible_loop_var': 'item'}) 
skipping: [172.20.20.12] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 8.8.8.8', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '8.8.8.8', 'ansible_loop_var': 'item'}) 
skipping: [172.20.20.14] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 30.30.30.30', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '30.30.30.30', 'ansible_loop_var': 'item'}) 
skipping: [172.20.20.14] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 14.14.14.14', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '14.14.14.14', 'ansible_loop_var': 'item'}) 
skipping: [172.20.20.14] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 5.5.5.5', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '5.5.5.5', 'ansible_loop_var': 'item'}) 
skipping: [172.20.20.14] => (item={'changed': False, 'msg': '', 'backup': '', 'diff': [{'before': '', 'after': '', 'before_header': '/etc/ntp.conf (content)', 'after_header': '/etc/ntp.conf (content)'}, {'before_header': '/etc/ntp.conf (file attributes)', 'after_header': '/etc/ntp.conf (file attributes)'}], 'invocation': {'module_args': {'path': '/etc/ntp.conf', 'line': 'server 8.8.8.8', 'state': 'present', 'backrefs': False, 'create': False, 'backup': False, 'firstmatch': False, 'follow': False, 'regexp': None, 'insertafter': None, 'insertbefore': None, 'validate': None, 'mode': None, 'owner': None, 'group': None, 'seuser': None, 'serole': None, 'selevel': None, 'setype': None, 'attributes': None, 'src': None, 'force': None, 'content': None, 'remote_src': None, 'delimiter': None, 'directory_mode': None, 'unsafe_writes': None}}, 'failed': False, 'item': '8.8.8.8', 'ansible_loop_var': 'item'}) 

TASK [Restart ntp] *************************************************************
skipping: [172.20.20.12]
skipping: [172.20.20.14]

TASK [firewall : Installation of iptables] *************************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [firewall : show result_iptables] *****************************************
ok: [172.20.20.12] => {
    "result_iptables": {
        "cache_update_time": 1681718990,
        "cache_updated": false,
        "changed": false,
        "failed": false
    }
}
ok: [172.20.20.14] => {
    "result_iptables": {
        "cache_update_time": 1681738976,
        "cache_updated": false,
        "changed": false,
        "failed": false
    }
}

TASK [firewall : Already installed] ********************************************
ok: [172.20.20.12] => {
    "msg": "iptables is already installed"
}
ok: [172.20.20.14] => {
    "msg": "iptables is already installed"
}

TASK [Allow port 80 on the firewall] *******************************************
ok: [172.20.20.12]
ok: [172.20.20.14]

TASK [Create users] ************************************************************
ok: [172.20.20.12] => (item=None)
ok: [172.20.20.14] => (item=None)
ok: [172.20.20.12] => (item=None)
ok: [172.20.20.14] => (item=None)
ok: [172.20.20.12] => (item=None)
ok: [172.20.20.14] => (item=None)
ok: [172.20.20.12] => (item=None)
ok: [172.20.20.12]
ok: [172.20.20.14] => (item=None)
ok: [172.20.20.14]

PLAY RECAP *********************************************************************
172.20.20.12               : ok=18   changed=0    unreachable=0    failed=0    skipped=6    rescued=0    ignored=0   
172.20.20.14               : ok=18   changed=0    unreachable=0    failed=0    skipped=6    rescued=0    ignored=0   

```






Machine |IP           |Fonction | 
------  | -------    | -----   
ubuntu03|172.20.20.13| AWX / Ansible| 
ubuntu05|172.20.20.15| Gitlab       | 
ubuntu02|172.20.20.12| Client1      | 
ubuntu04|172.20.20.14| Client3      | 



