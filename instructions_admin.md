
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

Consulter la version de slurm installée de différente manière
```
sinfo --version	
sbatch --version
```

Consulter les partitions existantes et les nœuds déclarés
```
sinfo
sinfo -s
sinfo -N    # node-oriented format
```

Consulter les caractéristiques d’un noeud
```
scontrol show node=c1
```

Vérifier dans quelle partition un nœud est associé
```
sinfo -n c1
```

créer une nouvelle partition avec 2 nœuds et autoriser l'utilisation uniquement au groupe unix bench1 (attention de reporter la partition dans les fichiers de configuration si vous souhaitez qu’elle soit persistante)
```
scontrol create PartitionName=dakar MaxTime=INFINITE State=UP Nodes=c[1,2] AllowGroups=bench1
```

Modifier la nouvelle partition pour fixer le paramètre MaxTime sur 10 heures et vérifiez la partition
```
scontrol  update PartitionName=dakar MaxTime=10:00:00
scontrol  show partition=dakar
```

Connectez-vous sur le nœud de login avec l’utilisateur bench1 puis soumettre la commande suivante avec srun. Pourquoi cela échoue ?
```
srun --partition=dakar hostname
```

Créer l’account depuis le noeud slurm
```
sacctmgr create account name=bench1
sacctmgr create user name=bench1 account=bench1 partition=Dakar
sacctmgr  show assoc
```

Relancer la commande srun
```
srun --partition=dakar hostname
```

Tenter de soumettre le même job avec un délai de 24 heures. Consulter le statut du job avec squeue
```
srun  --time=24:00:00 --partition=dakar hostname & 
```

Modifier la partition pour définir l'état sur DRAIN et vérifier l'état avec la commande sinfo
```
scontrol  update PartitionName=dakar State=drain
sinfo -p dakar
```

Soumettre un autre job avec la commande srun
```
srun --nodes=1 --ntasks=1 --time=00:01:00 --partition=dakar hostname
```

Supprimer la partition
```
scontrol delete PartitionName=dakar 
```

Supprimer les accounts
```
sacctmgr delete user bench1 cluster=my_cluster account=bench1
sacctmgr -i delete account bench1
sacctmgr show assoc
```

Vérifier la configuration qos actuelle
```
sacctmgr  show qos
```

Créer deux qos avec des limitations différentes
```
sacctmgr create qos padawan Priority=100 MaxJobs=1 MaxSubmit=2 MaxTRES=cpu=2
sacctmgr create qos jedi Priority=10000 MaxJobs=4 MaxSubmit=8 MaxTRES=cpu=8
sacctmgr  show qos
```

Créer deux accounts génériques avec une qos par défaut associée
```
sacctmgr create account name=eqa set qos=jedi
sacctmgr create account name=eqb set qos=padawan
sacctmgr show assoc
```

Associer les utilisateurs bench1 et bench2 aux deux accounts créés ci-dessus
```
sacctmgr  create user name=bench1 account=eqa
sacctmgr  create user name=bench1 account=eqb
sacctmgr  create user name=bench2 account=eqb
sacctmgr show assoc
```

Ajouter la qos jedi à l'association bench1 / account eqb
```
sacctmgr  modify user bench1 where account=eqb set qos+=jedi
```

Afficher les caractéristiques de l'association
```
sacctmgr  show assoc where user=bench1
```

Soumettre un job avec la qos avec l’utilisateur bench2 depuis le nœud de login
```
srun --nodes=3 --ntasks-per-node=3 --time=00:01:00 --qos=jedi hostname
```

Ajouter un dernier account chef et associé bench1 à celui-ci. La qos sélectionnée est normal
```
sacctmgr  create account name=chef qos=normal
sacctmgr  create user name=bench1 account=chef qos=normal
```

Ajouter / supprimer une qos
```
sacctmgr create qos qostp 
sacctmgr delete qos qostp
```




