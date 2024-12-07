# smb
pour le quete wsc partage de fichiers avec windows 


## Partage de fichiers (Windows)

### Sur un Windows Server 2022 avec le rôle ADDS déployé sur une VM :

#### -  Installe le rôle Serveur de fichiers

Pour l'installation du rôle de partage de fichiers :
Dans le Server Manager => Manage => Add Roles and features => Next(x2) => Dans Server Roles => File and Storage Services => On déroule => Dans File and isCSI Services => On coche File Server => Next (x2) => Install
J'ai aussi coché AD-DS sachant que j'en aurait aussi besoin pour les groupes et Users ainsi que pour le domaine => Création du domaine "Totoenterprise.lan"
=>Création de 3 groupes : "RH", "Compta" et "Direction"
=>Création de 3 Users : "totodirlo", "totocompta" & "totoRH"
Mis chaque utilisateur dans son groupe. Pour ça, dans AD Users and computers toujours : Clique droirt sur l'utilisateur en question => Add to a group => recherche du groupe et ajout.

#### -  Crée un dossier "Documents_Entreprise" à la racine du disque C:
#### -  Crée trois sous-dossiers : "RH", "Comptabilité" et "Direction"
Dans l'explorateur de fichiers Windows, on crée le dossier "Documents_Entreprise" à la racine C:\ ainsi que ses sous dossiers "RH, "Direction" et "Comptabilité".

#### -  Configure un partage nommé "Docs" pour ce dossier

Dans Le Server Manager, sur la gauche, on clique sur "File and Storage Services", puis "Shares" 
Ensuite sur le milieu, dans la partie "Shares" en haut à droite, on clique sur Tasks, puis "New Share", on sélectionne ensuite "SMB - Quick Share" => Next => En bas on sélectionne "Type a custom Path" et "Browse" et on sélectionne le dossier créé précédemment (C:\Documents_Entreprise) => Select folder => Next
Dans "Share name", on nomme le partage "Docs". => Next
Dans permissions, j'ai cliqué sur "disable inheritance" pour pouvoir supprimer Users (qui éaient sur read & write) puis je les ai remis sur read seulement, Administrators et Adlministrator sur Full control. J'ai également mis le groupe "Direction" sur Read & Write.

#### -  Configure les permissions NTFS et de partage pour que :
##### -  Le groupe "RH" ait un accès en lecture/écriture au dossier "RH"
##### -  Le groupe "Comptabilité" ait un accès en lecture/écriture au dossier "Comptabilité"
##### -  Le groupe "Direction" ait un accès en lecture/écriture à tous les dossiers
##### -  Tous les utilisateurs du domaine aient un accès en lecture seule au dossier "Documents_Entreprise"

Pour ca je suis allé dans l'explorateur de fichiers  
Pour les droits de la direction =>Déjà fait lorque j'ai partagé le dossier à tous les utilisateurs du domaine (j'avais ajouté un read & write au groupe Direction)  
Pour les droits de la compta => clique droit sur le sous-dossier "Comptabilité" => Properties => Onglet Sharing => Share => En haut on déroule avec la petite flèche => Find peaple => On cherche et sélectionne l'objet "Compta" Pour les droits de la RH => pareil mais avec le groupe RH sur le dossier RH
  
Ainsi Direction a les droits en lecture / écriture sur le dossier parent donc sur tous les sous-dossiers.
RH uniquement sur le dossier RH
Compta sur le groupe Comptabilité.

#### -  Utilise PowerShell pour lister tous les partages sur le serveur

```
PS C:\Users\Administrator> Get-SmbShare

Name     ScopeName Path                                                 Description
----     --------- ----                                                 -----------
ADMIN$   *         C:\Windows                                           Remote Admin
C$       *         C:\                                                  Default share
Docs     *         C:\Documents_Entreprise
IPC$     *                                                              Remote IPC
NETLOGON *         C:\Windows\SYSVOL\sysvol\totoenterprise.lan\SCRIPTS Logon server share
SYSVOL   *         C:\Windows\SYSVOL\sysvol                             Logon server share
``` 

#### -  Sur un poste client Windows 10, configure un lecteur réseau pointant vers ce partage via PowerShell

Dans le poste client, j'ai d'abord rejoins le domaine totoenterprise.lan.
Pour ça : Configuration du client sur le même réseau privé que le serveur. (leserveur est en 192.168.0.1, le client en 192.168.0.2 avec en dns 192.168.0.1)
Ensuite => Démarrer => Paramètres => Système => A propos de => renommer ce PC (avancé) => Là on sélectionne domaine au lieu de workgroup et on saisi "totoenterprise.lan"
Ensuite on saisi les identifiants d'un user définit dans l'ADDS su serveur, j'ai saisi totodirlo avec son mdp.


#### -  Teste l'accès depuis le poste client avec différents comptes utilisateurs

Je me suis donc connecté avec totodirlo.  
Ouverture du dossier partagé sur le serveur :  
Dans l'explorateur de fichiers en haut j'ai saisi l'adresse du dossier partagé : dans mon cas "\\DC1\Docs"  
Puis j'ai ouvert le dossier et créé un fichier texte nommé test dans chaque dossier en rentrant un texte dedans et en l'enregistrant afin de vérifier que la groupe direction avait bien les droits en lecture / écriture. J'ai pu les créer, les enregister donc c'est bon.  
Avec le groupe comptabilité maintenant.  
J'ai fait la même chose, connecté sur le client avec totocompta et constaté qu'il pouvait bien lire tous les fichiers créé par totodirlo mais ne pouvait modifier et n'enregister que le fichier contenu dans le sous-dossier comptabilité.  
Encore pareil avec le totoRH du groupe RH, c'était bon aussi il n'a pu enregister un ajout de texte que dans le fichier test du dossier RH.  
  
Après je suis retourné sur le serveur et j'ai retrouvé tous mes fichiers créés sur le client.


