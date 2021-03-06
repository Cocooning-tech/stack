# Installation de l'OS

## Installation d'ubuntu server armhf

### Gravure et insertion de la SD

Graver une version de l'image disque sur SD card (Balena)

* version __ubuntu-20.04-server__
* version __ubuntu-18.04-server__

Ejecter et rebrancher le lecteur/graveur USB

## Première connection en SSH

Utiliser Putty (déteminer IP sur port 22)  
login : __ubuntu__  
password : __ubuntu__  
Change le mot de passe et se reconnecter  

```bash
sudo su
apt update 
apt full-upgrade
timedatectl set-timezone Europe/Paris
apt-get install zip docker.io docker-compose nfs-kernel-server
# mkdir /home/root
wget https://codeload.github.com/Cocooning-tech/cocooning/zip/master
unzip master
mv cocooning-master /apps
rm master
chmod 600 /apps/traefik/acme.json
## install docker
sudo curl
# sudo systemctl enable docker
```

> Sous la version 20.04 il peut être nécessaire de rebooter entre update et upgrade

### Activer le Wifi

Configurer Netplan  

```bash
sudo bash -c "echo 'network: {config: disabled}' > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg"
sudo nano /etc/netplan/10-my-config.yaml
```

Ajouter le code  
>Pas de tabulation dans ce fichier mais des "espaces"

```bash
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      dhcp6: no
      accept-ra: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
  wifis:
    wlan0:
      dhcp4: no
      dhcp6: no
      accept-ra: no
      addresses: [192.168.1.99/24]
      gateway4: 192.168.1.1
      access-points:
        "Livebox-XXXX":
          password: "XXXXX"
```

> Changer l'adresse IP selon la box et le node
Appliquer la configuration  

```bash
sudo netplan generate
sudo netplan apply
```

### Changer le hostname

```bash
nano /etc/hostname
```

Changer le hostname en fonction du type de noeud

```bash
cl-1-master-1
```

```bash
cl-1-worker-1
```

>Les hostname de chaque noeud du cluster doivent être différents

```bash
reboot
```

## Installation d'un server NFS

```bash
sudo su
apt-get install nfs-kernel-server
```

Créez une table d'export NFS

```bash
nano /etc/exports
```

Copier coller les chemins ci-dessous

```bash
/apps/hassio 192.168.1.100(rw,no_root_squash,aync,no_subtree_check)
```  

> Créer autant de ligne que de répertoire à partager  
Mettre à jour la table nfs

```bash
exportfs -ra
```

Ouvrez les ports utilisés par NFS.
Pour savoir quels ports NFS utilise, entrez la commande suivante:

```bash
rpcinfo -p | grep nfs
```

Ouvrez les ports générés par la commande précédente.

```bash
sudo ufw allow 2049
```

Relancer le service

```bash
sudo service nfs-kernel-server reload
```

## Installation de Docker

### Installation d'un master node

```bash
sudo su
docker swarm init --advertise-addr 192.168.1.100
```

### Installation d'un worker node

```bash
sudo su
docker swarm join --token SWMTKN-1-4r733pcmjoem5er2r8v8bml7jhxlsphibhe3wftcycy5gw5z5i-drfkkg3p41na68fltnjtc0kjj 192.168.1.100:2377
```

Créer le network de type overlay (traefik,hassio...) : cocooning-network

## Installation de k3s  

Modifier le fichier

* __cmdline.txt__ sur __ubuntu-20.04-server__
* __nobtcmd.txt__ sur __ubuntu-18.04-server__
Ajouter en fin de ligne

```bash
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```

### Mode High Availability with Embedded DB (Experimental) avec etcd
