

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

Vous avez accès aux manpages dans les conteneurs : `man srun ; man sbatch ; man slurm.conf` etc…

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

Le TP est hébergé sur une Machine Virtuelle (VM) chez le fournisseur OVH et est accessible par ssh avec le login almalinux. Le mot de passe vous sera communiqué en séance. Exemple de connexion : 
```
ssh almalinux@141.94.106.28 
```
Le cluster slurm fonctionne sur la base de conteneurs podman. Pour opérer le cluster, positionnez-vous dans le répertoire `/home/almalinux/TP_slurm_utilisateur` de votre machine virtuelle user-X, et utiliser l’un des alias :
```
UP          # démarrage
DOWN        # arrêt
RESTART     # redémarrage
STATE       # statut du cluster slurm
```

Lorsque le cluster est démarré, STATE doit vous retourner :
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

Il est possible de se connecter indépendamment sur chaque conteneur.

Si vous opérez des modifications dans la configuration slurm, privilégiez un RESTART pour la prise en compte des modifications (redémarrage de tous les services).

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
[almalinux@user-X TP_slurm_utilisateur]$ ./connect.sh 
Le nom de l'image est obligatoire.
Usage: ./connect.sh -n <nom_de_l_image> [-u <utilisateur>]
Options:
  -n, --name           Nom de l'image podman (obligatoire)
  -u, --user           Nom de l'utilisateur (facultatif: par défaut root)
```

Par exemple pour se connecter avec root au conteneur de management slurm :
```
[almalinux@user-X TP_slurm_utilisateur]$ ./connect.sh -n slurm
podman-compose version: 1.0.6
['podman', '--version', '']
using podman version: 4.6.1
podman exec --interactive --tty slurm bash
[root@slurm /]#
```

Pour se connecter avec l’utilisateur bench1 au noeud de login :
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

Déplacez-vous dans le répertoire `/home/bench1/TP_job_simple` et soumettez le script job.slurm. Ecraser une ou plusieurs options sbatch en ligne de commande afin de constater la préponderance des options.
```
cd /home/bench1/TP_job_simple
sbatch  job.slurm
sbatch -N 2 –n 4 job.slurm
```

Consulter la sortie des jobs dans le répertoire de lancement `slurm-<jobid>.out`
Modifier le fichier job.slurm pour séparer les sorties erreur et standard puis relancer
```
#SBATCH -o slurm-%j.out
#SBATCH -e slurm-%j.err
```

Exporter une variable dans votre environnement et évaluer le comportement avec l'utilisation de l'option `--export`
```
srun --partition=short /bin/env | grep DAKAR
export DAKAR=1
srun --partition=short /bin/env | grep DAKAR
srun --partition=short --export=NONE /bin/env | grep DAKAR
srun --partition=short --export=DAKAR=2 /bin/env | grep DAKAR
```

<br/><br/>
<h2>7. Comprendre la configuration du cluster et les limitations posées</h2>

Soumettre quelques commandes simples avec srun depuis le nœud de login avec l’utilisateur bench1
```
srun --nodes=1 --ntasks=2 --time=00:01:00 --partition=short hostname
srun --nodes=2 --ntasks=2 --time=00:01:00 --partition=short hostname
srun --nodes=3 --ntasks=3 --time=00:01:00 --partition=short hostname
```
Pourquoi cette dernière commande ne passe pas ?

Consulter la configuration des comptes utilisateurs
```
sacctmgr  show assoc
sacctmgr  show assoc where user=bench1
```

Maitenant, consulter les configurations des QOS (Quality Of Service). Observer les différentes limitations positionnées. Consulter le manpage si necessaire `man sacctmgr`
```
sacctmgr show qos
```

Soumettre les commandes suivantes et consulter le statut des jobs avec l’alias squeue_
```
sbatch -A eqb --qos=padawan -N 1 -n 1 --wrap="hostname ; sleep 1m"
squeue_
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
<h2>9. Utilisation d'un répertoire temporaire</h2>

Il s'agit d'un cas test utilisant un répertoire créé temporairement par un script de prolog slurm ajouté par l'administrateur.
Ce répertoire est supprimé à la fin du job par un script epilog.
Placez-vous dans le répertoire `/home/bench1/TP_tmpdir`.
Consulter le fichier et lancer le job.
```
sbatch job.slurm
```
Analyser le fichier de sortie.

<br/><br/>
<h2>10. Utilisation de la fonctionnalité de job array</h2>

Un "job array" dans SLURM permet d'exécuter de multiples tâches similaires avec une seule commande, simplifiant ainsi la gestion des travaux par lot.

Déplacez-vous dans le répertoire `/home/bench1/TP_job_array`.
Consluter le fichier de soumission job.slurm et exécuter le job.
```
sbatch job.slurm
Analyser les fichiers de sortie.
```
<br/><br/>
<h2>11. Utilisation de la fonctionnalité de job dependency</h2>

La fonctionnalité de dépendance de tâches dans SLURM permet de spécifier des relations entre les travaux, déterminant ainsi l'ordre d'exécution en fonction de l'état des travaux précédents.

Placez-vous dans le répertoire `/home/bench1/TP_job_dependency`.
Consulter les fichiers et lancer les jobs avec dépendance. Consulter le statut des jobs avec l’alias squeue_, en particulier les colonnes DEPENDENCY et NODELIST(REASON).
```
./lancer
squeue_
```

<br/><br/>
<h2>12. Tests additionnels MPI - hello world & NPB</h2>

<h3>12.1 MPI hello world</h3>

Placez-vous dans le répertoire `/home/bench1/TP_hello_world_mpi`. Compiler et lancer le job
```
make
sbatch job.slurm
```
Faire varier les paramètres (nombre de noeuds, tasks …)

<br/><br/>
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

