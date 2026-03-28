------------------------------------------------------------------------------------------------------
ATELIER ANSIBLE
------------------------------------------------------------------------------------------------------
L’idée en 30 secondes : Dans cet atelier, vous allez apprendre à **automatiser le déploiement d’un serveur web avec Ansible**, directement depuis GitHub **Codespaces**, sans infrastructure complexe. L’objectif est de comprendre comment **décrire un état cible** (installer, configurer, déployer) et laisser l’outil l’appliquer automatiquement, de manière reproductible et idempotente. On passe ainsi d’une logique manuelle à une logique DevOps industrialisée.
  
**Architecture cible :** Ci-dessous, voici l'architecture cible souhaitée.   
  
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/8064eb95-da73-4bdd-9ef2-a7cebbbd71c8" />
  
  
-------------------------------------------------------------------------------------------------------
Séquence 1 : Codespace de Github
-------------------------------------------------------------------------------------------------------
Objectif : Création d'un Codespace Github  
Difficulté : Très facile (~5 minutes)
-------------------------------------------------------------------------------------------------------
**Faites un Fork de ce projet**. Si besoin, voici une vidéo d'accompagnement pour vous aider à "Forker" un Repository Github : [Forker ce projet](https://youtu.be/p33-7XQ29zQ) 
  
Ensuite depuis l'onglet **[CODE]** de votre nouveau Repository, **ouvrez un Codespace Github**.
  
---------------------------------------------------
Séquence 2 : Création du votre environnement de travail
---------------------------------------------------
Objectif : Créer votre environnement de travail  
Difficulté : Simple (~10 minutes)
---------------------------------------------------
Vous allez dans cette séquence mettre en place votre environnement et les logiciels Ansible. Depuis le terminal de votre Codespace copier/coller les codes ci-dessous étape par étape :  

**Installation de Ansible et Nginx**  
```
sudo apt update
sudo apt install -y ansible curl
```
**Vérifier l’environnement**  
```
ansible --version
curl --version
```
**Tester la cible locale**  
```
ansible -i inventory.ini local -m ping
```
**Exécuter le playbook**  
```
ansible-playbook -i inventory.ini playbook.yml
```
**Vérifier le résultat**  
```
curl http://localhost
```  
---------------------------------------------------  
**Réccupération de l'URL de votre serveur Nginx**. Votre serveur Nginx (et sa page Web) est déployé dans Codespace. Pour obtenir votre URL cliquez sur l'onglet **[PORTS]** dans votre Codespace (à coté de Terminal), ouvrez le port 80 et rendez public votre port (Visibilité du port). Ouvrez l'URL dans votre navigateur et c'est terminé.  
  
---------------------------------------------------
Séquence 3 : Déploiement de l'infrastructure
---------------------------------------------------
Objectif : Déployer l'infrastructure sur le cluster Kubernetes
Difficulté : Facile (~15 minutes)
---------------------------------------------------  
Nous allons à présent déployer notre infrastructure sur Kubernetes. C'est à dire, créér l'image Docker de notre application Flask avec Packer, déposer l'image dans le cluster Kubernetes et enfin déployer l'infratructure avec Ansible (Création du pod, création des PVC et les scripts des sauvegardes aututomatiques).  

**Création de l'image Docker avec Packer**  
```
packer init .
packer build -var "image_tag=1.0" .
docker images | head
```
  
**Import de l'image Docker dans le cluster Kubernetes**  
```
k3d image import pra/flask-sqlite:1.0 -c pra
```
  
**Déploiment de l'infrastructure dans Kubernetes**  
```
ansible-playbook ansible/playbook.yml
```
  
**Forward du port 8080 qui est le port d'exposition de votre application Flask**  
```
kubectl -n pra port-forward svc/flask 8080:80 >/tmp/web.log 2>&1 &
```
  
---------------------------------------------------  
**Réccupération de l'URL de votre application Flask**. Votre application Flask est déployée sur le cluster K3d. Pour obtenir votre URL cliquez sur l'onglet **[PORTS]** dans votre Codespace (à coté de Terminal) et rendez public votre port 8080 (Visibilité du port). Ouvrez l'URL dans votre navigateur et c'est terminé.  

**Les routes** à votre disposition sont les suivantes :  
1. https://...**/** affichera dans votre navigateur "Bonjour tout le monde !".
2. https://...**/health** pour voir l'état de santé de votre application.
3. https://...**/add?message=test** pour ajouter un message dans votre base de données SQLite.
4. https://...**/count** pour afficher le nombre de messages stockés dans votre base de données SQLite.
5. https://...**/consultation** pour afficher les messages stockés dans votre base de données.
  
---------------------------------------------------  
### Processus de sauvegarde de la BDD SQLite

Grâce à une tâche CRON déployée par Ansible sur le cluster Kubernetes (un CronJob), toutes les minutes une sauvegarde de la BDD SQLite est faite depuis le PVC pra-data vers le PCV pra-backup dans Kubernetes.  

Pour visualiser les sauvegardes périodiques déposées dans le PVC pra-backup, coller les commandes suivantes dans votre terminal Codespace :  

```
kubectl -n pra run debug-backup \
  --rm -it \
  --image=alpine \
  --overrides='
{
  "spec": {
    "containers": [{
      "name": "debug",
      "image": "alpine",
      "command": ["sh"],
      "stdin": true,
      "tty": true,
      "volumeMounts": [{
        "name": "backup",
        "mountPath": "/backup"
      }]
    }],
    "volumes": [{
      "name": "backup",
      "persistentVolumeClaim": {
        "claimName": "pra-backup"
      }
    }]
  }
}'
```
```
ls -lh /backup
```
**Pour sortir du cluster et revenir dans le terminal**
```
exit
```

---------------------------------------------------
Séquence 4 : Exercices  
Difficulté : Facile (~30 minutes)
---------------------------------------------------
### Exercice 1 : Customisation de la page d'accueil 
Customisez la page d'accueil de votre site Web en ajoutant la ligne suivante dans votre fichier index.html  
```
<h1>Déploiement réalisé par [Votre Nom]</h1>
```

### Exercice 2 : Paramétrer dynamiquement votre serveur avec Ansible 
Voici le contenu (contenu imposé) de votre nouveau fichier index.html    
```
<!DOCTYPE html>
<html>
<head>
  <title>{{ page_title }}</title>
</head>
<body>
  <h1>{{ page_title }}</h1>
  <p>Déployé par : {{ author }}</p>
  <p>Utilisateur système : {{ app_user }}</p>
</body>
</html>
```
Modifier votre playbook afin de :  
* Créer un title contenant le texte suivant : "Serveur déployé avec Ansible"
* L'auteur sera : "Votre nom"
* L'utilisateur sera un **utilisateur Linux** : "Votre prénom"
  
---------------------------------------------------
Séquence 5 : Questions  
Difficulté : Moyenne (~45 minutes)
---------------------------------------------------
**Complétez et documentez ce fichier README.md** pour répondre aux questions des exercices.  
Faites preuve de pédagogie et soyez clair dans vos explications et procedures de travail.  

**Question 1 :**  
Pourquoi Ansible est-il qualifié d’outil "déclaratif" ?    
  
*..Répondez à cet exercice ici..*

**Question 2 :**  
Pourquoi l’utilisation de variables est-elle essentielle dans un playbook ?  
  
*..Répondez à cet exercice ici..*

**Question 3 :**  
En quoi Ansible facilite-t-il la gestion de plusieurs serveurs ?  
  
*..Répondez à cet exercice ici..*

**Question 4 :**  
Quels sont les avantages et les limites d’Ansible dans un contexte DevOps ?   
  
*..Répondez à cet exercice ici..*
  
**Question 5 :**  
Quelle est la différence entre les modules copy et template dans Ansible ?   
  
*..Répondez à cet exercice ici..*

---------------------------------------------------
Séquence 6 : Atelier  
Difficulté : Moyenne (~1 heure)
---------------------------------------------------
### Structurer votre déploiement Ansible afin de pouvoir choisir entre un rôle DEV ou un rôle PROD  
Modifier votre fichier playbook.yml afin de pouvoir choisir entre un rôle DEV ou un rôle PROD.  

**Resultats attendus**  
* Déploiement en DEV  
```
ansible-playbook -i inventory.ini playbook.yml --limit dev
```
```
app_name: "Application DEV"
env: "dev"
author: "Etudiant DEV"
```
* Déploiement en PROD
```
ansible-playbook -i inventory.ini playbook.yml --limit prod
```
```
app_name: "Application PROD"
env: "prod"
author: "Equipe Ops""
```
---------------------------------------------------
### Zoom sur votre environnement Codepace 
Codepace est un outil proposer par GitHub soumit à quota (4$/mois). Si vous dépasser votre quota mensuel vous ne serez plus en mesure de pouvoir utiliser Codespace. C'est pourquoi à **la fin de votre atelier, pensez à suprimer votre Codespace après avoir mis à jour votre GitHub**.  

---------------------------------------------------
Evaluation
---------------------------------------------------
Cet atelier ANSIBLE, **noté sur 20 points**, est évalué sur la base du barème suivant :  
- Mise en oeuvre (2 points)
- Exercice N°1 - Customisation de la page d'accueil (2 points)
- Exercice N°2 - Paramétrer dynamiquement votre serveur avec Ansible (2 points)
- Questions + Qualité du Readme (lisibilité, erreur, ...) (5 points)
- Atelier - Rôles DEV et PROD (6 points)
- Processus travail (quantité de commits, cohérence globale, interventions externes, ...) (3 points) 
