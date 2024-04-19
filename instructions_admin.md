
<h1>TP slurm - Instructions administrateur</h1>

<br/><br/>
<h2>1. Généralités</h2>

Vous avez 3 alias définis dans votre environnement et utilisable pendant les TP.
Ceux-ci permettent l'utilisation d'un format de sortie personnalisé pour chaque commande.
```
alias sacct_='\sacct -D --format=jobid%-13,user%-12,jobname%-35,submit,timelimit,partition,qos,nnodes,start,end,elapsed,state,exitcode%-6,Derivedexitcode%-6,nodelist%-200 '
alias sinfo_='\sinfo --format="%100E %12U %19H %6t %N" '
alias squeue_='\squeue  -o "%.18i %.9P %.8j %.8u %.10a %.10q %.8T %.10M %.12l %.10E %.10A %.8C %.6D %R“’
```

Vous avez accès aux manpages dans les conteneurs : `man srun ; man sbatch ; man slurm.conf` etc…

Une fois connecté, privilégier l’utilisation de tmux, plus simple pour naviguer entre plusieurs fenêtre et permet au formateur de visualiser aisément votre fenêtre lorsque vous sollicitez son aide. Quelque commandes tmux : 

```
tmux new -s pasteur                # nouvelle session pasteur
tmux attach -t pasteur             # se connecter à la session pasteur existante
Ctrl+b then c                      # nouvelle fenêtre
Ctrl+b then n  et Ctrl+b then p    # fenêtre suivante et fenêtre précédent
Ctrl+b then d                      # se détacher de la session tmux
```

<br/><br/>
<h2>2. Administration du cluster slurm</h2>

La VM contenant les TP est hébergée chez le fournisseur OVH et est accessible par ssh avec le login almalinux. Le mot de passe vous sera communiqué en séance. Exemple de connexion : 
```
ssh almalinux@141.94.106.28 
```
Le cluster slurm fonctionne sur la base de conteneurs podman. Pour opérer le cluster, positionnez-vous dans le répertoire `/home/bench1/TP_slurm_utilisateur`, et utilisez l’un des alias :
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

<br/><br/>
<h2>3. Connexion au cluster</h2>

3 utilisateurs sont disponibles pour se connecter au cluster :
```
root        # administration du cluster slurm
bench1      # utilisateur principal pour le TP
bench2      # utilisateur facultatif pour des tests additionnels
```

Pour vous connecter aux conteneurs, un script connect.sh est disponible dans le répertoire `/home/bench1/TP_slurm_utilisateur` et prend une option obligatoire et une option facultative :
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

<br/><br/>
<h2>4. Configuration du cluster</h2>

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

<br/><br/>
<h2>5. Les partitions et accounts</h2>

Créer une nouvelle partition avec 2 nœuds et autoriser l'utilisation uniquement au groupe unix bench1 (attention de reporter la partition dans les fichiers de configuration si vous souhaitez qu’elle soit persistante)
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

<br/><br/>
<h2>6. Commandes d'allocation et de soumission de jobs</h2>

Utiliser salloc pour obtenir une allocation de ressources de slurm
```
salloc --nodes=1 --ntasks=2 --partition=short 
```

Exécuter ces deux commandes et comparer le résultat
```
hostname
srun hostname
```

Vérifier les variables SLURM_* dans l'environnement du job
```
env | grep SLURM
```

Vérifier le statut du job avec et sans l’alias squeue 
```
squeue -j $SLURM_JOBID
squeue_ -j $SLURM_JOBID
```

Libérer les ressources allouées par slurm
> Ctrl+d  OR  exit

Soumettre quelques commandes simples avec srun depuis le nœud de login avec l’utilisateur bench1
```
srun --nodes=1 --ntasks=2 --time=00:01:00 --partition=short hostname
srun --nodes=2 --ntasks=2 --time=00:01:00 --partition=short hostname
srun --nodes=3 --ntasks=3 --time=00:01:00 --partition=short hostname
```
Pourquoi cette dernière commande ne passe pas ?

Soumettre les commandes suivantes et consulter le statut des jobs avec l’aias squeue_
```
sbatch -A eqb --qos=padawan -N 1 -n 1 --wrap="hostname ; sleep 1m"
```

Soumettre une série de job
```
for i in {1..3}
do
  sbatch -A eqb --qos=padawan -N 1 -n 1 --wrap="hostname ; sleep 30s"
done
```
Pourquoi le message d’erreur QOSMaxSubmitJobPerUserLimit après quelques soumissions ?
Utiliser `scancel <jobid>` si besoin.

Tester différents paramétrages :
```
srun -A bench1 --qos=padawan -N 2 -n 4 --cpus-per-task=1 hostname
srun -A eqa --qos=jedi -N 2 -n 4  hostname
srun -A eqa --qos=jedi -N 3 -n 3 -p exclusive --cpus-per-task=1 hostname
srun -A chef --qos=normal -N 3 -n 3 -p exclusive --cpus-per-task=1 hostname
```

<br/><br/>
<h2>7. Réservations</h2>

Créer dès maintenant une réservation d'une durée de 3 heures avec votre nodeset et pour l'utilisateur bench2 uniquement
```
scontrol  create reservation=pasteur nodes=c3 user=bench1 start=now duration=03:00:00
```

Vérifier la reservation
```
sinfo -T
scontrol show reservation=pasteur
```

Mettre la réservation à jour afin d’ajouter le flag DAILY (la réservation sera répétée tous les jours à la même heure). Vérifier à nouveau la réservation
```
scontrol  update reservation=pasteur Flags+=DAILY
```

Soumettre un job dans la reservation en tant qu’utilisateur bench1
```
srun -p long --ntasks=4 --reservation=pasteur hostname
```

Supprimer la réservation
```
scontrol delete reservation=pasteur
```

<br/><br/>
<h2>8. Accounting utilisateur</h2>

Vérifier l’accounting slurm des 5 derniers jobs
```
sacct -X | tail -5
```

Consulter les jobs exécutés depuis une certaine date pour l'utilisateur bench1
```
sacct -X -u bench1 -S 2024-04-24
```

Lancer la même commande avec un format optimisé exécutant l'alias sacct_
```
sacct_ -X -u bench1 -S 2024-04-24
```

Obtenir les jobs été exécutés sur c[1-2] entre des dates spécifiques
```
sacct_ -X -N c[1-2] -S 2024-04-22T10:30:00  -E 2024-04-22T12:42:00
```

Obtenir les jobs qui ont été exécutés sur 2 nœuds au cours du mois d’avril
```
sacct_ -X -S 2024-04-01T00:00:01  -E 2024-04-30T23:59:59 --nnodes=2
```

<br/><br/>
<h2>9. Modifier le statut des noeuds dans slurm</h2>

Vérifiez le status des noeuds 
```
sinfo -n c[1-3]
sinfo -R -n c[1-3]
```

Modifier le statut des noeuds dans slurm 
```
scontrol  update state=drain node=c[1-2] reason="redemarrage requis"
sinfo_ -R 
scontrol  update state=idle node=c[1-2]
```

<br/><br/>
<h2>10. Activer le Node Health Check (NHC)</h2>

Activer nhc dans la configuration slurm
```
grep –i HealthCheck /etc/slurm/slurm.conf
```
Ajouter quelques règles de contrôle nhc (fichier de configuration `/etc/nhc/nhc.conf`)
Voir le lien (https://github.com/mej/nhc#installation)
> * || check_hw_physmem XXgb XXgb 5%
> * || check_hw_cpuinfo 4 X X


Run nhc explicitly. Connectez vous sur le noeud c1 en root et exécuter :
```
[root@c1 /]# nhc –v
```
Puis consulter la fin du fichier `/var/log/nhc.log`
Et pour une sortie plus verbeuse (debug) :
```
[root@c1 /]# nhc –d
```

<br/><br/>
<h2>11. Activer l'utilisation des scripts epilog/prolog</h2>

Ajouter un script de prologue afin de
 - créer un répertoire temporaire pour chaque job dans l’espace de travail partagé /workdir : `/workdir/$SLURM_USER.$SLURM_JOBID`
 - positionner les droits d’accès et owner du répertoire créé

Ajouter un script d’epilog afin de
 - supprimer le répertoire temporaire du job et son contenu (l’utilisateur doit copier les données qu’il souhaite conserver à la fin du job).
 - synchroniser les écritures en cache sur le stockage `sync`

Lancer un job utilisateur afin de vérifier la présence de la variable d’environnement et l’existence du répertoire temporaire
Pour cela, placez-vous dans le répertoire `/home/bench1/TP_tmpdir` de l’utilisateur bench1. Consulter le fichier et lancer le job.
```
sbatch job.slurm
```
Analyser le fichier de sortie.

<br/><br/>
<h2>12. Tests additionnels MPI - hello world & NPB</h2>

<h3>12.1 MPI hello world</h3>

Placez-vous dans le répertoire `/home/bench1/TP_hello_world_mpi`. Compiler et lancer le job
```
make
sbatch job.slurm
```
Faire varier les paramètres (nombre de noeuds, tasks …)

<h3>12.2 NPB benchmark</h3>

Placez-vous dans le répertoire `/home/bench1/TP_NPB3.3-MPI`. Compiler le noyau CFD BT (Block Tri-diagonal solver). Il s’agit d’un benchmark utilisant MPI.
Une fois compilé, placez-vous dans le répertoire `/home/bench1/TP_NPB3.3-MPI/run` et lancer le job
```
make BT CLASS=A
cd run/
sbatch job.slurm
```
Faire varier les paramètres (nombre de noeuds, tasks …)
<br/><br/>
<h2>FIN</h2>

