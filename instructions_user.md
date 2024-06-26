

<h1>TP slurm - Instructions utilisateur</h1>

<br/><br/>
<h2>1. Généralités</h2>

Vous avez 3 alias définis dans votre environnement et utilisable pendant les TP.
Ceux-ci permettent l'utilisation d'un format de sortie personnalisé pour chaque commande.
```
alias sacct_='\sacct -D --format=jobid%-13,user%-12,jobname%-35,submit,timelimit,partition,qos,nnodes,start,end,elapsed,state,exitcode%-6,Derivedexitcode%-6,nodelist%-200 '
alias sinfo_='\sinfo --format="%100E %12U %19H %6t %N" '
alias squeue_='\squeue  -o "%.18i %.9P %.8j %.8u %.10a %.10q %.8T %.10M %.12l %.10E %.10A %.8C %.6D %R“’
```

Vous avez accès aux manpages dans les conteneurs : `man srun ; man sbatch ; man scontrol` etc…

Une fois connecté, privilégier l’utilisation de tmux, plus simple pour naviguer entre plusieurs fenêtre et permet au formateur de visualiser aisément votre fenêtre lorsque vous sollicitez son aide. Quelque commandes tmux : 

```
tmux new -s pasteur                # nouvelle session pasteur
tmux attach -t pasteur             # se connecter à la session pasteur existante
Ctrl+b then c                      # nouvelle fenêtre
Ctrl+b then n                      # fenêtre suivante
Ctrl+b then p                      # fenêtre précédente
Ctrl+b then d                      # se détacher de la session tmux
```

<br/><br/>
<h2>2. Administration du cluster slurm</h2>

Le TP est hébergé sur une Machine Virtuelle (VM) chez le fournisseur OVH et est accessible par ssh avec le login **almalinux**. Le mot de passe vous sera communiqué en séance. Exemple de connexion : 
```
ssh almalinux@141.94.106.28 
```
Le cluster slurm est accessible depuis le VM et fonctionne sur la base de conteneurs podman. Pour opérer le cluster, positionnez-vous dans le répertoire `/home/almalinux/TP_slurm_utilisateur` de votre machine virtuelle admin-X, et utiliser l’un des alias :
```
UP          # démarrage du cluster slurm 
DOWN        # arrêt du cluster slurm
RESTART     # redémarrage du cluster slurm
STATE       # statut du cluster slurm
```

Démmarrer le cluster
```
UP
```

Lorsque celui-ci est démarré, la commande STATE doit vous retourner les informations suivantes :
```
[almalinux@user-X TP_slurm_utilisateur]$ STATE
CONTAINER ID  IMAGE                             COMMAND     CREATED         STATUS         PORTS       NAMES
24709a5b010a  docker.io/library/mariadb:latest  mariadbd    15 seconds ago  Up 15 seconds              mariadb
ad972835acf8  localhost/slurm-23:latest                     13 seconds ago  Up 13 seconds              slurm
62f10538b34f  localhost/slurm-23:latest                     11 seconds ago  Up 11 seconds              c1
d1f8478a2f45  localhost/slurm-23:latest                     9 seconds ago   Up 9 seconds               c2
65fa8a7a5941  localhost/slurm-23:latest                     7 seconds ago   Up 7 seconds               c3
ae13f39c1c8b  localhost/slurm-23:latest                     5 seconds ago   Up 5 seconds               login
```

Le cluster slurm est composé de 6 conteneurs (nœuds)
```
mariadb      # la base de données mysql
slurm        # le nœud d’administration (slurmctld, slurmdbd)
c[1-3]       # les nœuds de calcul (slurmd puis slurmstepd pour les jobs)
login        # nœud de login, non intégré au cluster slurm mais depuis lequel on peut soumettre des travaux (jobs) slurm
```

<br/><br/>
<h2>3. Connexion au cluster</h2>

3 utilisateurs sont disponibles pour se connecter au cluster :
```
root        # administration du cluster slurm
bench1      # utilisateur principal pour le TP
bench2      # utilisateur facultatif pour des tests additionnels
```

Pour vous connecter aux conteneurs, un script connect.sh est disponible dans le répertoire `/home/almalinux/TP_slurm_utilisateur` de la VM admin-X et prend une option obligatoire ainsi qu'une option facultative :
```
[almalinux@admin-X TP_slurm_utilisateur]$ ./connect.sh 
Le nom de l'image est obligatoire.
Usage: ./connect.sh -n <nom_de_l_image> [-u <utilisateur>]
Options:
  -n, --name           Nom de l'image podman (obligatoire)                    # colonne NAMES de la sortie de la commande STATE
  -u, --user           Nom de l'utilisateur (facultatif: par défaut root)     # bench1, bench2, root  
```

Pour le déroulement de ce TP, vous pouvez simplement vous connecter au conteneur **login** avec l'utilisateur **bench1** comme ceci :
```
[almalinux@user-X TP_slurm_utilisateur]$ ./connect.sh -n login -u bench1
podman-compose version: 1.0.6
['podman', '--version', '']
using podman version: 4.6.1
podman exec --interactive --user bench1 --tty login bash
[bench1@login ~]$
```

<br/><br/>
<h2>4. Configuration du cluster</h2>

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
<h2>5. Commandes d'allocation et de soumission de jobs</h2>

<h3>5.1 Différence de l'exécution d'une commande avec ou sans srun</h3>

Déplacez-vous dans le répertoire `/home/bench1/TP_job_simple`. Consulter le contenu et soumettre le script **job.slurm**.
Observer le résultat des commandes exécutées dans le fichier de sortie **slurm-SLURM_JOBID.out**
```
cd /home/bench1/TP_job_simple
sbatch  job.slurm
cat slurm-*.out
```

Modifier les valeurs des options par `-N 2` et `--ntasks-per-node=3` dans le fichier **job.slurm** puis relancer le job.
Constater les différences dans les résultats.

<h3>5.2 Prépondérance des options </h3>

Ecraser une ou plusieurs options sbatch en ligne de commande afin de constater l'ordre de prise en compte des options (l'option passée en ligne de commande est prépondérante).
```
cd /home/bench1/TP_job_simple
sbatch  job.slurm
sbatch -N 2 --ntasks-per-node=4 job.slurm
```

<h3>5.3 Redirection stdout et stderr </h3>

Modifier le fichier job.slurm pour séparer les sorties erreur et standard dans deux fichiers distincts puis relancer
```
#SBATCH -o slurm-%j.out
#SBATCH -e slurm-%j.err
```

<h3>5.4 Interrompre des jobs </h3>

Lancer une série de job puis les interrompres
```
for i in {1..5}; do sbatch job_long.slurm ; done
squeue
scancel -u bench1 -t PD                            # interrompre les jobs de l'utilsiteur bench1 en statut PENDING (PD)
squeue
scancel -u bench1 -t R --name=test                 # interrompre les jobs de l'utilsiteur bench1 en statut RUNNING (R) dont le nom est test
squeue
```

<h3>5.5 Gestion de l'environnement </h3>

Exporter une variable dans votre environnement et évaluer le comportement avec l'utilisation de l'option `--export`
```
srun --partition=short /bin/env | grep DAKAR
export DAKAR=1
srun --partition=short /bin/env | grep DAKAR
srun --partition=short --export=NONE /bin/env | grep DAKAR
srun --partition=short --export=DAKAR=2 /bin/env | grep DAKAR
```

<h3>5.6 Distribution des processus (tasks) </h3>

Remplacer dans le script **job.slurm** la ligne `srun hostname` par celle-ci
```
srun --label --distribution=block hostname | sort
```
Consulter la documentation `man srun` pour connaitre le rôle des options `--label` et `--distribution`.
Changer le mode de distribution et observer la différence dans la sortie
```
srun --label --distribution=cyclic hostname | sort
```

<br/><br/>
<h2>6. Comprendre la configuration du cluster et les limitations posées</h2>

<h3>6.1 Limite des partitions </h3>

Soumettre quelques commandes simples avec srun depuis le nœud de login avec l’utilisateur bench1
```
srun --nodes=1 --ntasks=2 --time=00:01:00 --partition=short hostname
srun --nodes=2 --ntasks=2 --time=00:01:00 --partition=short hostname
srun --nodes=3 --ntasks=3 --time=00:01:00 --partition=short hostname &
squeue
```
Pourquoi cette dernière commande ne passe pas ? (voir la colonne NODELIST(REASON) de `squeue`)

<h3>6.2 Configurations des accounts slurm </h3>

Consulter la configuration des comptes utilisateurs
```
sacctmgr  show assoc
sacctmgr  show assoc where user=bench1
```

<h3>6.3 Configurations des QOS </h3>

Maitenant, consulter les configurations des QOS (Quality Of Service). Observer les différentes limitations positionnées. Consulter le manpage si necessaire `man sacctmgr`
```
sacctmgr show qos
```

Soumettre la commande suivante et consulter le statut des jobs avec l’alias squeue_
```
sbatch -A eqb --qos=padawan -N 1 -n 1 --wrap="hostname ; sleep 1m"
squeue_                                                               # colomne PARTITION, USER, ACCOUNT, QOS ...
```

<h3>6.4 Limite des QOS </h3>

Soumettre une série de job
```
for i in {1..3}
do
  sbatch -A eqb --qos=padawan -N 1 -n 1 --wrap="hostname ; sleep 30s"
done
```
Pourquoi le message d’erreur QOSMaxSubmitJobPerUserLimit après quelques soumissions ?
Utiliser `scancel SLURM_JOBID` si besoin.

<h3>6.4 Tests accounts / QOS </h3>

Tester différents paramétrages :
```
srun -A bench1 --qos=padawan -N 2 -n 4 --cpus-per-task=1 hostname
srun -A eqa --qos=jedi -N 2 -n 4  hostname
srun -A eqa --qos=jedi -N 3 -n 3 -p exclusive --cpus-per-task=1 hostname
srun -A chef --qos=normal -N 3 -n 3 -p exclusive --cpus-per-task=1 hostname
```

<br/><br/>
<h2>7. Accounting utilisateur</h2>

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
sacct_ -X -N c[1-2] -S 2024-04-24T10:30:00  -E 2024-04-24T12:42:00
```

Obtenir les jobs qui ont été exécutés sur 2 nœuds au cours du mois d’avril
```
sacct_ -X -S 2024-04-01T00:00:01  -E 2024-04-30T23:59:59 --nnodes=2
```

Obtenir les jobs exécutés avec l'account eqb à partir du 2024-04-22
```
sacct_ -A eqb -S 2024-04-24
```

<br/><br/>
<h2>8. Utilisation d'un répertoire temporaire</h2>

Il s'agit d'un cas test utilisant un répertoire créé temporairement par un script de prolog slurm ajouté par l'administrateur.
Ce répertoire est supprimé à la fin du job par un script epilog.
Placez-vous dans le répertoire `/home/bench1/TP_tmpdir`.
Consulter le fichier **job.slurm** et lancer le job.
```
sbatch job.slurm
```
Analyser le fichier de sortie.

<br/><br/>
<h2>9. Utilisation de la fonctionnalité de job array</h2>

Un "job array" dans SLURM permet d'exécuter de multiples tâches similaires avec une seule commande, simplifiant ainsi la gestion des travaux par lot.

Déplacez-vous dans le répertoire `/home/bench1/TP_job_array`.
Consluter le fichier de soumission job.slurm et exécuter le job.
```
sbatch job.slurm
Analyser les fichiers de sortie.
```
<br/><br/>
<h2>10. Utilisation de la fonctionnalité de job dependency</h2>

La fonctionnalité de dépendance de tâches dans SLURM permet de spécifier des relations entre les travaux, déterminant ainsi l'ordre d'exécution en fonction de l'état des travaux précédents.

Placez-vous dans le répertoire `/home/bench1/TP_job_dependency`.
Consulter les fichiers et lancer les jobs avec dépendance. Consulter le statut des jobs avec l’alias squeue_, en particulier les colonnes DEPENDENCY et NODELIST(REASON).
```
./lancer
squeue_
```

<br/><br/>
<h2>11. Tests MPI - hello world & NPB</h2>

<h3>11.1 MPI hello world</h3>

Placez-vous dans le répertoire `/home/bench1/TP_hello_world_mpi`. Compiler et lancer le job
```
make
sbatch job.slurm
```
Faire varier les paramètres (nombre de noeuds, tasks …)

<br/><br/>
<h3>11.2 NPB benchmark</h3>

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

