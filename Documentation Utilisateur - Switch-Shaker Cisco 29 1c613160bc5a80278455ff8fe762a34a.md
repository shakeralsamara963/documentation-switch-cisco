# Documentation Utilisateur - Switch-Shaker Cisco 2960XR

## Configuration Réseau :

Switch-Shaker :

- VLAN 1 → Non actif
- VLAN 20 → LAN_Client ⇒ 172.20.0.1/24
    - Port 10
- VLAN 50 → LAN_Serveur ⇒ 192.168.50.1/24
    - Port 1

![](https://placeholder.com/wp-content/uploads/2018/10/placeholder.com-logo1.png)

## I. Connexion Avec Câble Console

Sur votre Ordinateur, ouvrir un terminal série (PuTTY, TeraTerm, etc.) et le Gestionnaire de Périphérique en même temps.

Dans le Gestionnaire de Périphérique, chercher sur quel port COM est branché votre câble console. Ensuite, ouvrir votre terminal et se connecter en "Serial" en utilisant les paramètres suivants :

- Port COM : celui identifié précédemment
- Vitesse : 9600 bauds
- Bits de données : 8
- Parité : Aucune
- Bits d'arrêt : 1
- Contrôle de flux : Aucun

Une fois sur votre interface terminal, vous devriez voir apparaitre les informations du switch (Nom, etc.)

## II. Commandes de Base

### Se connecter et modifier la configuration :

```
Switch> enable
Switch# configure terminal     "Entrer en mode configuration"
Switch(config)# exit           "Quitter les différents modes"
Switch#
```

### Pour renommer un switch :

```
Switch(config)# hostname Switch-Shaker
Switch-Shaker(config)#
```

### Pour voir une configuration existante :

```
Switch# show running-config
```

### Pour mettre un message d'accueil :

```
Switch(config)# banner motd #
Bienvenue sur le Switch-Shaker
#
```

### Pour enregistrer la configuration :

```
Switch# write memory
```

ou

```
Switch# copy running-config startup-config
```

### Création d'un utilisateur :

```
Switch(config)# username admin privilege 15 secret mot_de_passe
```

### Pour vérifier le statut des interfaces Ethernet :

```
Switch# show interfaces status
```

ou

```
Switch# show interfaces brief
```

### Pour attribuer une passerelle par défaut :

```
Switch(config)# ip default-gateway 192.168.1.254
```

## III. Supprimer une Configuration Existante

Pour supprimer une configuration existante :

```
Switch# erase startup-config
Switch# reload
```

Si vous connaissez le mot de passe et souhaitez juste le supprimer :

```
Switch# configure terminal
Switch(config)# no username admin
```

## IV. Configuration de VLANs

À SAVOIR :

- Trunk = Mode TRUNK chez Cisco (tagged chez HP)
- Access = Mode ACCESS chez Cisco (untagged chez HP)

### Pour créer un VLAN :

```
Switch> enable
Switch# configure terminal
Switch(config)# vlan 20
Switch(config-vlan)# name LAN_Client
Switch(config-vlan)# vlan 50
Switch(config-vlan)# name LAN_Serveur
Switch(config-vlan)# exit
```

### Attribuer une adresse IP à un VLAN :

```
Switch> enable
Switch# configure terminal
Switch(config)# interface vlan 20
Switch(config-if)# ip address 172.20.0.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# interface vlan 50
Switch(config-if)# ip address 192.168.50.1 255.255.255.0
Switch(config-if)# no shutdown
```

### Pour attribuer un VLAN à un port ou à un groupe de ports :

```
Switch(config)# interface gigabitethernet 2/0/10
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# exit
Switch(config)# interface gigabitethernet 2/0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 50
```

### Pour configurer un port en mode trunk :

```
Switch(config)# interface gigabitethernet 2/0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 20,50
```

### Pour supprimer un port d'un VLAN spécifique :

```
Switch(config)# interface gigabitethernet 2/0/10
Switch(config-if)# no switchport access vlan
Switch(config-if)# switchport access vlan 1
```

### Pour vérifier la configuration des VLANs :

```
Switch# show vlan brief
```

## V. Connexion SSH

Pour activer la connexion SSH, il vous faut obligatoirement configurer une adresse IP sur le switch et s'assurer que votre PC est dans le même réseau.

Pour savoir quelles interfaces sont associées à un VLAN :

```
Switch# show vlan id 20
```

Pour configurer SSH :

```
Switch(config)# ip domain-name example.com
Switch(config)# crypto key generate rsa
(Choisir 1024 bits minimum)
Switch(config)# ip ssh version 2
Switch(config)# ip ssh authentication-retries 3
Switch(config)# ip ssh time-out 60
Switch(config)# line vty 0 15
Switch(config-line)# login local
Switch(config-line)# transport input ssh
```

Ensuite, ouvrez un terminal comme PuTTY et connectez-vous en SSH en utilisant l'adresse IP du VLAN, votre nom d'utilisateur et votre mot de passe.

## VI. Liaisons Trunk

Pour établir une liaison trunk entre deux switches, configurez un port sur chaque switch en mode trunk.

Sur Switch-Shaker :

```
Switch> enable
Switch# configure terminal
Switch(config)# interface gigabitethernet 2/0/24
Switch(config-if)# description Liaison_Trunk
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 20,50
```

## VII. Routage Inter-VLANs

Pour activer le routage entre VLANs sur le Switch-Shaker :

```
Switch> enable
Switch# configure terminal
Switch(config)# ip routing
```

Vous pouvez vérifier les routes configurées :

```
Switch# show ip route
```

Après cette configuration, les appareils sur le VLAN 20 peuvent communiquer avec ceux sur le VLAN 50 en utilisant le switch comme routeur.

Exemple :

- Les PCs du VLAN 20 (172.20.0.0/24) peuvent ping les PCs du VLAN 50 (192.168.50.0/24)
- Les PCs du VLAN 50 peuvent ping les PCs du VLAN 20

## VIII. Configuration actuelle du Switch-Shaker

Le Switch-Shaker est actuellement configuré avec :

- Routage IP activé
- Serveur HTTP et HTTPS activés
- VLAN 1 désactivé
- VLAN 20 (172.20.0.1/24) assigné au port 10
- VLAN 50 (192.168.50.1/24) assigné au port 1
- STP en mode PVST