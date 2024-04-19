
<h1>Instructions TP slurm administrateur</h1>


Vous avez 3 alias définis dans votre environnement et utilisable pendant les TP :
```
alias sacct_='\sacct -D --format=jobid%-13,user%-12,jobname%-35,submit,timelimit,partition,qos,nnodes,start,end,elapsed,state,exitcode%-6,Derivedexitcode%-6,nodelist%-200 '
alias sinfo_='\sinfo --format="%100E %12U %19H %6t %N" '
alias squeue_='\squeue  -o "%.18i %.9P %.8j %.8u %.10a %.10q %.8T %.10M %.12l %.10E %.10A %.8C %.6D %R“’
```

Vous avez accès aux manpages dans les conteneurs : man srun ; man sbatch ; man slurm.conf etc…

Une fois connecté, privilégier l’utilisation de tmux, plus simple pour naviguer entre plusieurs fenêtre et permet au formateur de visualiser aisément votre fenêtre lorsque vous sollicitez son aide. Quelque commandes tmux : 

```
tmux new -s pasteur    		# créer une session tmux nommée pasteur
tmux attach -t pasteur		# se connecter à la session pasteur existante
Ctrl+c		                # nouvelle fenêtre
Ctrl+n  et Ctrl+p	        # fenêtre suivante et fenêtre précédent
Ctrl+d		                # se détacher de la session tmux
```

La VM contenant les TP est hébergée chez le fournisseur OVH et est accessible par ssh avec le login almalinux. Le mot de passe vous sera communiqué en séance. Exemple de connexion : 
```
ssh almalinux@141.94.106.28 
```
Le cluster slurm fonctionne sur la base de conteneurs podman. Pour opérer le cluster, positionnez-vous dans le répertoire TP_slurm_utilisateur, et utilisez l’un des alias :
```
UP          # démarrage
DOWN        # arrêt
RESTART     # redémarrage
STATE       # statut du cluster slurm
```

Lorsque le cluster est démarré, STATE doit vous retourner :
![image](https://github.com/trivinoc/TP_slurm_container_instructions/assets/115139596/0d55af76-4329-4e32-b2c9-c029641316a1)

Le cluster slurm est composé de 6 conteneurs (nœuds)
```
mariadb      # la base de données mysql
slurm        # le nœud d’administration (slurmctld, slurmdbd)
c[1-3]       # les nœuds de calcul (slurmd puis slurmstepd pour les jobs)
login        # nœud de login, non intégré au cluster slurm mais depuis lequel on peut soumettre des travaux (jobs) slurm
```

Il est possible de se connecter indépendamment sur chaque conteneur.

Si vous opérez des modifications dans la configuration slurm, privilégier un RESTART pour la prise en compte des modifications (redémarrage de tous les services).

3 utilisateurs sont disponibles pour se connecter au cluster :
```
root        # administration du cluster slurm
bench1      # utilisateur principal pour le TP
bench2      # utilisateur facultatif pour des tests additionnels
```

Pour vous connecter aux conteneurs, un script connect.sh est disponible dans le répertoire TP_slurm_utilisateur et prend une option obligatoire et une option facultative :
```
[almalinux@admin-1 TP_slurm_utilisateur]$ ./connect.sh 
Le nom de l'image est obligatoire.
Usage: ./connect.sh -n <nom_de_l_image> [-u <utilisateur>]
Options:
  -n, --name           Nom de l'image podman (obligatoire)
  -u, --user           Nom de l'utilisateur (facultatif: par défaut root)
```

Par exemple pour se connecter avec root au conteneur de management slurm :
```
[almalinux@admin-1 TP_slurm_utilisateur]$ ./connect.sh -n slurm
podman-compose version: 1.0.6
['podman', '--version', '']
using podman version: 4.6.1
podman exec --interactive --tty slurm bash
[root@slurm /]#
```

Pour se connecter avec l’utilisateur bench1 au noeud de login :
```
[almalinux@admin-1 TP_slurm_utilisateur]$ ./connect.sh -n login -u bench1
podman-compose version: 1.0.6
['podman', '--version', '']
using podman version: 4.6.1
podman exec --interactive --user bench1 --tty login bash
[bench1@login ~]$
```

Compléter les différents fichiers de configuration slurm avec les valeurs adéquates (mot clef A_REMPLACER)
```
/etc/slurm/slurm.conf      # configuration générale
/etc/slurm/slurmdbd.conf   # base de données
/etc/slurm/partition.conf  # nœuds et partitions
```

Rechercher les termes à compléter
```
[root@slurm /]# grep A_REMPLACER /etc/slurm/*.conf
scontrol show config
```

Aidez-vous :
```
du contenu du fichier podman-compose.yml    # description mariadb et users
de la commande lscpu                        # configuration CPU des nœuds de calcul
de la commande free –m                      # configuration mémoire
```

Une fois terminé vous devez voir les nœuds apparaître avec le statut idle
```
[root@slurm /]# sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
short*       up    1:00:00      2   idle c[1-2]
long         up   infinite      2   idle c[2-3]
```

