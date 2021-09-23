## Comprendre dbus
d-bus est un "nouveau" système d'IPC (la première version date de 2005) qui permet de facilement passer des messages entre applications. Ce framework est de plus en plus utilisé par les daemons systèmes et par les applications linux qui veulent fournir des services avec une API standardisée. Il est donc intéressant de connaître dbus pour écrire des applications qui modifient le système lui-même de façon fiable.
Linux fournit les mécanismes traditionnels Unix d'IPC (sockets, mémoire partagée, sémaphores) mais ces mécanismes sont des primitives bas-niveau qui ne constituent pas par elles-mêmes un système de communication complet. Dbus utilise ces primitives pour construire un bus de communication permettant de communiquer facilement avec les daemons et services systèmes ou de session.

dbus apporte ainsi un certain nombre de fonctionnalités indispensables pour un bus de communication moderne.

    - Modèle d'appel de fonction : les mécanismes standards ne permettent pas directement de faire des appels de fonction distants. Dbus fournit un standard de sérialisation des paramètres qui permet d'appeler de façon fiable une fonction distante.
    - Sécurité des communications : le daemon dbus (chargé du routage des messages) vérifie les autorisations avant de transmettre les informations. Il ajoute également des metadonnées aux messages qui permettent à l'appelé d'avoir des informations fiables sur l'appelant afin de prendre des décisions de sécurité.
    - Activation de services à la demande : le daemon dbus est configuré pour connaître certains noms fixes sur le bus et les associer à des exécutables. Il lancera l'exécutable lorsqu'un message pour ce nom fixe sera envoyé. Il est également possible (et très courant) de lier un nom sur le bus à un service systemd. Cela permet de lancer un service systemd à la demande et ainsi d'avoir un grand nombre de services disponibles sans avoir besoin de process résidents.
    - Découvrabilité et introspection : il est possible d'interroger le daemon dbus pour connaître la liste des services disponibles sur le bus et chaque service peut être interrogé pour connaître les méthodes qu'il fournit ainsi que la signature de ces méthodes.
    - Passage de descripteurs de fichiers :  il est possible pour un service de passer un descripteur de fichier ouvert à un autre service. Cela permet de facilement créer des canaux de communication entre un daemon et une application pour passer des quantités de données importantes.
Traditionnellement, de nombreux daemons systèmes étaient fournis avec un utilitaire ligne de commande pour communiquer avec le daemon une fois celui-ci lancé. La communication se faisait via des mécanismes ad-hoc basés sur les IPC unix (typiquement via des sockets dans /var/run). Cela rendait l'utilisation de ces services systèmes très délicate, chaque daemon ayant son propre système de communication et son propre format de donnée et protocole d'échange. Beaucoup d'applications appelaient directement les utilitaires en ligne de commande depuis le code C pour ne pas avoir à réimplémenter ces protocoles. Dbus permet d'éviter à avoir à faire ce genre de choses en fournissant un système d'appel vers les daemons plus simple à utiliser depuis le code, tout en fournissant une richesse de paramètres et de fonctionnalités bien supérieure à ce que peut décrire une ligne de commande.

Si dbus est intéressant pour ses performances, c'est surtout cette standardisation du protocole de communication et le grand nombre de services standards sur le bus qui font sa valeur.

## Exemples

### Polkit

De nombreux daemons systèmes fournissent des API nécessitant des droits d'accès spécifiques pour être utilisés. Polkit fournit un mécanisme générique pour permettre à un daemon privilégié de vérifier la validité d'une requête. Le daemon va transmettre des informations sur la requête au daemon polkit, celui-ci utilise des règles (dans /etc/polkit-1/rules.d et  /usr/share/polkit-1/rules.d) pour déterminer si la requête est valide. Polkit est également capable de retrouver un terminal ou une session X correspondant à l'utilisateur pour lui permettre de s'authentifier si nécessaire (via les notions de seat et session qui ne seront pas abordées dans cet article).

Polkit est donc un mécanisme de tiers de confiance facile à utiliser (des bibliothèques évitent de se préoccuper des détails de la communication avec le serveur) pour permettre à un utilisateur non-privilégié d'effectuer des actions privilégiées.

### udisks

Le daemon udisks surveille les différents périphériques de stockage du système et offre une API dbus pour interagir avec eux. Ce daemon est la façon la plus pratique d'interagir avec ces périphériques, que ce soit pour détecter l'arrivée de nouveaux périphériques, pour connaître les propriétés des disques (taille, ID, point de montage etc...) ou pour les reformater. L'outil en ligne de commande udisksctl permet d'interagir avec ce daemon de façon simplifiée.

udisks utilise polkit pour les permissions. Cette combinaison permet de contrôler finement quels utilisateurs peuvent monter quels disques. La documentation complète de udisks se trouve ici.

### Packagekit

Packagekit est un daemon qui fournit une API permettant d'installer des paquetages via dbus. Le gros intérêt de packagekit est de fournir une API indépendante de la distribution. Chaque distribution fournit des scripts à packagekit lui permettant de remplir son rôle quel que soit le backend utilisé.

Packagekit utilise polkit, il est donc possible d'utiliser ce mécanisme pour avoir des permissions fines sur la politique de gestion des paquetages du système (autoriser certains utilisateurs à faire des mises à jour mais pas à ajouter des paquetages, par exemple).

pkcon est un utilitaire ligne de commande permettant d'interagir avec packagekit.

### NetworkManager

NetworkManager est le daemon chargé des connexions réseau. Il détecte l'ajout et la suppression de cartes réseau, ajoute des routes, envoie les requêtes DHCP etc... Nous sommes habitués à le configurer via nmcli ou l'un des applet des différents gestionnaires de fenêtre, mais toutes ces méthodes ne font qu'interagir avec le daemon via dbus. Accéder ainsi au daemon permet de vérifier facilement si une connexion est active, d'activer ou désactiver une connexion et évidemment de paramétrer les interfaces. En particulier NetworkManager permet de lister et de sélectionner les points d'accès Wifi.

Source(s):
- https://www.linuxembedded.fr/2015/07/comprendre-dbus