![image](https://github.com/trivinoc/TP_slurm_container_instructions/assets/115139596/d6e9de9b-2099-4c16-b775-a019b1a41412)Vous avez 3 alias définis dans votre environnement et utilisable pendant les TP :
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
UP		: démarrage
DOWN	: arrêt
RESTART	: redémarrage
STATE	: statut du cluster slurm

Lorsque le cluster est démarré, STATE doit vous retourner :
![image](https://github.com/trivinoc/TP_slurm_container_instructions/assets/115139596/0d55af76-4329-4e32-b2c9-c029641316a1)


