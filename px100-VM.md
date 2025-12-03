# Guide : Pincher Px100
Le robot se connecte par USB et peut être contrôlé sur ROS1 ou ROS2 (ici on utilisera ROS2), cependant, il n'est pas nécessaire de posséder des connaissances sur ROS2 pour suivre ce TP. Sur Linux tout devrait fonctionner directement sans passer par une machine virtuelle. Sur Mac ou Windows, il faudra utiliser une machine virtuelle pour simuler Linux sur votre ordinateur. 

Remarque, Docker semble être une meilleure solution qu'une VM car plus simple à partager et à s'approprier, mais Docker ne donne pas les accès directs aux ports USB, donc pas possible de connecter le robot via Docker facilement. 

## Installation et Configuration de la machine virtuelle 
### Téléchargement de VirtualBox
VirtualBox est un logiciel de virtualisation gratuit qui permet de créer et exécuter des machines virtuelles sur un ordinateur.
Rendez-vous sur le site de [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

### Téléchargement de Linux Focal :

Pour simuler Linux sur votre ordinateur, il vous faut télécharger l'image correspondante de la bonne version de Linux souhaitée. Attention, cela diffère selon le processeur de votre machine hôte car vous ne pouvez pas virtualiser une machine qui possède une architecture de processeur différente de la vôtre. Voici les liens selon votre processeur (64 bits) :

Architecture x86-64 (AMD64) (MacOS Intel, autre PC avec processeur Intel ou AMD) : [Linux Focal AMD64](https://releases.ubuntu.com/focal/)  
Téléchargez ceci : `ubuntu-20.04.6-desktop-amd64.iso`

Architecture AArch64 (ARM64) (MacOS Silicon, Raspberry Pi) :  
[Linux Focal ARM64](https://cdimage.ubuntu.com/releases/focal/release/)  
Téléchargez ceci : `ubuntu-20.04.5-live-server-arm64.iso`

### Création et configuration de la machine virtuelle
1.  Création de la VM :

    Vous pouvez maintenant créer une nouvelle machine virtuelle en important bien l'image téléchargée. Renseignez le futur mot de passe de la VM également, allouez assez d'espace mémoire et de disque pour son bon fonctionnement.

    Remarque : Dans les réglages de VirtualBox, vous pouvez modifier la taille de la fenêtre dans 'Display' puis 'Scale Factor'.

2. Interface graphique (GNOME) :

    Si vous n'avez pas d'interface graphique (donc juste un terminal Linux sans le bureau et les autres applications), vous devez la télécharger maintenant.

    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install ubuntu-desktop
    ```

    Puis redémarrez votre VM (avec `reboot` ou en l’éteignant), vous devriez avoir l'interface graphique de Linux.

    Problème avec le terminal : Si vous avez une interface graphique mais qu'il est impossible d'ouvrir le terminal (depuis Ctrl+Alt+T ou dans le moteur de recherche), le plus simple est de télécharger un autre terminal comme `xterm` (depuis la console virtuelle Ctrl+Alt+F3) :

    ```bash
    sudo apt install xterm
    ```

    Problème avec 'sudo' : Par défaut, VirtualBox met votre compte utilisateur `vboxuser` (nom par défaut) dans la liste des sudoers. Si ce n'est pas le cas, suivez les instructions sur [ajouter-utilisateur-sudo.md](./ajouter-utilisateur-sudo.md).

3. Configuration de VirtualBox : Il faut impérativement donner l'accès aux ports USB à VirtualBox pour que les USB soient reconnus dans la VM. Branchez le robot sur votre ordinateur, allez dans les paramètres de VirtualBox puis cliquez sur 'ajouter un nouveau filtre USB'.  

    <img src="Images/usb.png" alt="Accès au port USB" width="600" />

    Jetez un coup d'œil aux droits et accès qu'a l'application VirtualBox sur votre ordinateur et autorisez ce qui pourrait compromettre l'accès USB (Sécurité et Confidentialité sur MacOS).

---

### Installation de ROS2 Galactic et des packages Interbotix 

1. Architecture x86-64 (AMD64) :
    ```bash
    sudo apt install curl
    curl 'https://raw.githubusercontent.com/Interbotix/interbotix_ros_manipulators/main/interbotix_ros_xsarms/install/amd64/xsarm_amd64_install.sh' > xsarm_amd64_install.sh
    chmod +x xsarm_amd64_install.sh
    bash ./xsarm_amd64_install.sh -d galactic
    ```

    <img src="Images/installation_interbotix.png" alt="install" width="600" />

2. Architecture AArch64 (ARM64) :
    ```bash
    sudo apt install curl
    curl 'https://raw.githubusercontent.com/Interbotix/interbotix_ros_manipulators/main/interbotix_ros_xsarms/install/rpi4/xsarm_rpi4_install.sh' > xsarm_rpi4_install.sh
    chmod +x xsarm_rpi4_install.sh
    bash ./xsarm_rpi4_install.sh -d galactic 
    ```
    
    <img src="Images/installation_interbotix2.png" alt="install" width="600" />

---

## Utilisation

Allez dans le dossier `interbotix_ws` qui fait office de workspace pour ROS :
```bash
cd interbotix_ws
```

Sourcez le workspace (à faire dans chaque terminal utilisé) :
```bash
source install/setup.bash
```

Tous les packages Interbotix sont téléchargés et ROS2 Galactic est prêt à l'emploi.

Lancez le robot virtuel avec :
```bash
ros2 launch interbotix_xsarm_descriptions xsarm_description.launch.py robot_model:=wx200 use_joint_pub_gui:=true
```

Lancez le robot avec :
```bash
ros2 launch interbotix_xsarm_control xsarm_control.launch.py robot_model:=vx250
```

Par défaut, tous les moteurs sont bloqués. Pour les débloquer :

**Attention :** cette commande fera s’effondrer le bras robotique (s’il n’est pas déjà en position de repos), donc tenez-le ou sécurisez-le manuellement avant de l’exécuter.
Sur un autre terminal (après l'avoir sourcé):
```bash
ros2 service call /vx250/torque_enable interbotix_xs_msgs/srv/TorqueEnable "{cmd_type: 'group', name: 'all', enable: false}"
```

Vous pouvez manipuler le robot, le mettre dans une position souhaitée et activer l’appel précédent avec `enable: true` pour bloquer les moteurs.  
**Attention**: vérifiez d'avoir bien avoir désactivé le couple moteur avant de manipuler le robot.


## Documentation

https://docs.trossenrobotics.com/interbotix_xsarms_docs/ros_interface/ros2.html
https://docs.trossenrobotics.com/interbotix_xsarms_docs/ros_interface/ros2/software_setup.html#raspberry-pi-4b-arm64-architecture
