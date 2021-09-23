## Une introduction à UDEV

udev est le daemon chargé de gérer les device-nodes. Nous allons commencer par nous intéresser à ceux-ci pour comprendre le problème qu'udev résout.

Un système Unix va présenter certains outils informatiques (en particulier les périphériques matériels) sous forme de pseudo-fichiers. Ils sont visibles dans le système de fichiers et peuvent être manipulés via les outils habituels : cat, grep, open(), read(), write() etc...

Les systèmes Linux ont donc des fichiers pour permettre l'accès aux périphériques. Ceux-ci sont traditionnellement stockés dans /dev et sont appelés device nodes. Un device node est un point d'entrée vers le noyau caractérisé par un type (bloc ou char) et deux nombres: le major et le minor. Ce triplé définit de façon unique quel périphérique matériel est accédé via ce fichier. Le majeur permet au noyau de savoir quel driver doit gérer le périphérique et le mineur permet au driver de savoir quel périphérique parmi ceux qu'il gère est utilisé.

 Le noyau n'a pas de code spécifique à udev, et udev est un programme purement user-space relativement simple. Mais ce point de vue oublie un certain nombre d'évolutions du noyau qui sont apparues entre temps et sur lesquels s'appuie udev pour fonctionner.

- Le noyau exporte un grand nombre d'informations sur ses périphériques dans /sys. Une source d'information extrêmement complète. Cela permet de retrouver les majors et minors des périphériques (donc de ne pas les coder en dur ==> MAKEDEV) ainsi qu'un nom probable pour le fichier /dev correspondant.
- Le noyau fournit un lien netlink permettant à n'importe quel programme user-space d'être prévenu de l'arrivée ou du départ de n'importe quel périphérique. L'interface netlink fournit (entre autre) la position du périphérique dans /sys, ce qui permet de récupérer toutes les informations disponibles sur le périphérique.

udev est donc un daemon relativement simple qui récupère les événements noyau, les compare à une base de règles venant de fichiers de configuration et crée les entrées udev correspondantes. Il peut également appeler des commandes externes pour obtenir des informations supplémentaires sur le périphérique (comme une base de données des identifiants USB ou PCI) ou pour permettre à d'autres programmes de réagir (en envoyant un message d-bus par exemple).

## Les événements noyau

udev réagit donc à des événements envoyés par le noyau. Pour comprendre un peu plus ce qui se passe (et avant d'étudier la syntaxe des règles udev) nous allons étudier ces événements.

udev est capable d'afficher les événement noyau au fur et à mesure de leur arrivée. Pour cela, lancez la commande suivante :

```
udevadm monitor -k
```

puis branchez une souris USB. Vous devriez voir apparaître les lignes suivantes:

monitor will print the received events for:
```
KERNEL - the kernel uevent

KERNEL[117993.564897] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1 (usb)
KERNEL[117993.565452] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1:1.0 (usb)
KERNEL[117993.567809] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1:1.0/0003:15D9:0A4D.0009 (hid)
KERNEL[117993.568072] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1:1.0/0003:15D9:0A4D.0009/input/input28 (input)
KERNEL[117993.568443] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1:1.0/0003:15D9:0A4D.0009/input/input28/mouse0 (input)
KERNEL[117993.568713] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1:1.0/0003:15D9:0A4D.0009/input/input28/event1 (input)
KERNEL[117993.568763] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1:1.0/0003:15D9:0A4D.0009/hidraw/hidraw0 (hidraw)
```

Chaque ligne correspond à un événement envoyé par le noyau vers udev. On y voit apparaître :

- Le périphérique USB lui-même,
- L'unique interface sur ce périphérique,
- le périphérique USB en tant que périphérique HID,
- le périphérique en tant qu'input-device.

Le noyau est donc assez verbeux dans ses événements et chaque périphérique est vu plusieurs fois selon ses différentes fonctionnalités (une clé USB génère facilement une douzaine d'événements, plus si elle comporte un grand nombre de partitions).

Le noyau envoie un certain nombre de propriétés avec chaque événement. Nous pouvons demander à udev de nous afficher ces propriétés avec la commande suivante:

```
udevadm monitor -k -p
```
Le premier événement de branchement de notre souris affiche alors les propriétés suivantes:

```
KERNEL[118369.168889] add /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1 (usb)
ACTION=add
BUSNUM=002
DEVNAME=/dev/bus/usb/002/036
DEVNUM=036
DEVPATH=/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1
DEVTYPE=usb_device
MAJOR=189
MINOR=163
PRODUCT=15d9/a4d/100
SEQNUM=2678
SUBSYSTEM=usb
TYPE=0/0/0
```

Les propriétés dépendent du type exact de périphérique mais certaines propriétés sont toujours présentes:

- ACTION : Le type d'événement à traiter,
- MAJOR, MINOR : Les numéro majeur et mineur du périphérique concerné,
- SEQNUM : un numéro croissant pour ordonner les événements,
- SUBSYSTEM : Le sous-système noyau ayant causé l'événement,
- DEVPATH : Le fichier dans /sys correspondant au périphérique.

Source(s):
- https://www.linuxembedded.fr/2015/05/une-introduction-a-udev