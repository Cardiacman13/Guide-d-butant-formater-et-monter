# Guide débutant pour formater et monter au démarrage ses disques / SSD internes sur Linux

Pour ceux qui préfèrent une vidéo, [cliquez ici](https://www.youtube.com/watch?v=8mNHfQJ3-oo)

> [!WARNING]
>Je vous déconseille de partager vos jeux entre Linux et Windows. Il finira pas arriver une catastrohpe et si vous êtes sur cette page vous devez être débutants, vous ne vous en sortirez pas facilement. Donc chacun chez soit, Linux d'un coté, Windows de l'autre et moins ils communiquent, mieux c'est. Même à l'installation idéalement chacun son EFI, mettre Linux dans le EFI de 100mo de Windows, à la longue [un des deux risque fortement de casser l'autre](https://www.zdnet.fr/actualites/une-mise-a-jour-de-windows-casse-le-dual-boot-de-linux-mais-il-existe-une-solution-pour-certains-utilisateurs-395939.htm). **N'UTILISEZ PAS LE NTFS EXFAT FAT32 pour vos jeux sur Linux**. Les devs Bazzite font [ces mêmes recommendations](https://docs.bazzite.gg/Advanced/Auto-Mounting_Secondary_Drives/) et [Kyle le leader du projet est un ingénieur de chez Microsoft](https://github.com/KyleGospo). Néanmoins, si vraiment vous voulez tenter l'aventure NTFS sur Linux pour vos jeux, il vous faudra suivre [cette documentation de Valve](https://github.com/ValveSoftware/Proton/wiki/Using-a-NTFS-disk-with-Linux-and-Windows#preventing-ntfs-read-errors) à la lettre.
>
>À noter également que une erreur peut vous causer la perte de vos données ou empécher votre système de démarrer.

## Formater

Pour cette tâche, je vous recommande d'utiliser Gnome Disks, accessible facilement depuis la logithèque de votre distribution. Il fonctionne aussi bien sur Gnome que sur KDE ou autres DE.

1. **Installer Gnome Disques**: Cet outil facilite la gestion de vos disques. Vous le trouverez en cherchant "disques" dans la logithèque de votre distribution. Il est généralement près installé sur les distributions utilisants Gnome.

   ![Ouvrir Gnome Disks](images/1.png)

2. **Supprimer les anciennes partitions**: Si vous souhaitez effacer des partitions existantes, cliquez sur le bouton ![image](images/2-1.png)
situé à droite de la partition concernée dans l'interface de Gnome Disks.

   ![Supprimer les anciennes partitions](images/2.png)

3. **Formatage de la partition**: Cliquez sur les petits engrenages pour ouvrir le menu d'options, puis choisissez "Formater la partition". Une boîte de dialogue de formatage s'ouvrira.

   ![Formatage de la partition](images/3.png)

4. **Choisir le système de fichiers**: Il est fortement conseillé d'éviter les systèmes de fichiers NTFS ou exFAT sur Linux, en particulier si vous prévoyez de partager la partition avec Windows, car cela peut causer des problèmes. Nommez votre volume et sélectionnez Ext4 comme système de fichiers pour une compatibilité optimale avec Linux.

   ![Choisir le système de fichiers](images/4.png)

5. **Finaliser le formatage**: Assurez-vous de n'avoir commis aucune erreur avant de procéder. Cliquer sur "Formater" lancera le processus de formatage.

   ![Finaliser le formatage](images/5.png)

## Monter au démarrage

Pour que votre disque ou SSD soit automatiquement monté au démarrage :

1. **Modifier les options de montage**: Recliquez sur les petits engrenages, mais cette fois-ci, allez dans "Modifier les options de montage".

   ![Modifier les options de montage](images/6.png)

2. **Configurer le montage automatique** :
    - Cochez "Monter au démarrage".
    - Mettez les options `defaults` et, pour les SSD, ajoutez `noatime` pour réduire l'usure et améliorer la performance. Attention, toute erreur dans ces options peut empêcher le volume de se monter correctement pour éviter au maximum ce genre de problèmes, je vous recommande d'ajouter également l'option `nofail`.
    - Montez dans `/media`, `/mnt`, `/home` selon vos préférences.
    - Montez dans `/media` assure que vos disques soient faciles à trouver quelque soit l'application. (résutat [/mnt](https://codeberg.org/Gaming-Linux-FR/guide-formater-monter/src/branch/main/images/mnt.png) [résultat /media](https://codeberg.org/Gaming-Linux-FR/guide-formater-monter/src/branch/main/images/media.png))
    - Utilisez l'identification par UUID, qui est la méthode la plus fiable.
    - Laissez le type de système de fichiers sur "auto" ou spécifiez "ext4" si c'est votre choix lors du formatage.

   ![Configurer le montage automatique](images/7.png)

### Explications des options `defaults,nofail,noatime,x-gvfs-show`

- **`defaults`** offre un ensemble d'options de montage par défaut, incluant les permissions de lecture/écriture pour l'utilisateur, entre autres, convenant à la majorité des usages.
- **`nofail`** empêche le système de se bloquer si le point de montage échoue lors du démarrage. Cette option est particulièrement utile pour les disques externes ou les partages réseau qui ne sont pas toujours connectés ou disponibles. Avec `nofail`, le système continue de démarrer normalement, même si le périphérique spécifié n'est pas accessible au moment du montage.
- **`noatime`** évite la mise à jour de l'heure d'accès des fichiers à chaque lecture, réduisant ainsi les écritures sur le disque. C'est particulièrement bénéfique pour les SSD, où cela peut augmenter la performance et la durée de vie du disque.
- **`x-gvfs-show`** indique que le point de montage doit apparaître dans les gestionnaires de fichiers GNOME, KDE etc (par exemple Nautilus, Dolphin) , facilitant ainsi son accès et sa gestion via l’interface graphique. En d’autres termes, il permet de monter, démonter et parcourir le périphérique directement dans l’explorateur de fichiers.

### Vérifiez avant de redémarrer

La moindre erreur peut empécher de redémarrer et vous devrez depanner en ligne de commande (sauf si vous avez correctement entré l'option nofail), avant de redémarrer vous pouvez faire ces commandes : 

```bash
systemctl daemon-reload 
sudo mount -a
```
Si `sudo mount -a` ne renvoie rien, normalement tout est OK. Si cette commande renvoie une erreur il faudra la fixer avant de redémarrer.

---

### Fonctionnement

**Gnome Disques** ne fait en réalité qu’ajouter une ligne dans un fichier texte nommé **`fstab`**, lu par le système au démarrage. Ce fichier indique quelles partitions monter, où les monter et avec quelles options. Vous pouvez l’ouvrir et le modifier (à vos risques et périls) avec la commande :

```
sudo nano /etc/fstab
```

Si vous avez suivi ce tutoriel, vous avez normalement ajouté une ligne ressemblant par exemple à ceci :

```
UUID=f23c194d-2f48-4f47-a16a-1fd0aae4ca35   /media/jeux    ext4   defaults,nofail,noatime,x-gvfs-show           0 0
```

Vous auriez pu tout à fait réaliser cette configuration vous-même, sans passer par Gnome Disques, en récupérant l’UUID de la partition via la commande :

```
sudo blkid
```

puis en ajoutant la ligne appropriée dans le fichier **`/etc/fstab`**. 

```UUID=uuiddelapartition     /le/chemin     lesystèmedefichier    les,options     0 0``` 

> **Important** : Si vous formatez la partition ou si vous remplacez le disque, pensez à supprimer ou adapter la ligne correspondante dans **`/etc/fstab`** parce que l'uuid ne sera plus le même. 

Dans le cas contraire :

> - Si **l’option `nofail` n’est pas activée**, le système ne vas pas démarrer, car il attendra la partition manquante. Il faudra dépaner en ligne de commande le fstab.
> - Si **l’option `nofail` est activée**, le démarrage se fera, mais un message d’erreur sera consigné dans le journal, indiquant l’échec du montage de la partition.

---

### Bonus

Pour encore plus d'infos voir :

[Documentation de Adrien sur le fstab](https://www.linuxtricks.fr/wiki/fstab-explications-sur-le-fichier-et-sa-structure)
