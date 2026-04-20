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
Séquence 3 : Exercices  
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
Séquence 4 : Questions  
Difficulté : Moyenne (~45 minutes)
---------------------------------------------------
**Complétez et documentez ce fichier README.md** pour répondre aux questions des exercices.  
Faites preuve de pédagogie et soyez clair dans vos explications et procedures de travail.  

**Question 1 :**  
Pourquoi Ansible est-il qualifié d’outil "déclaratif" ?    
  
*..Ansible est qualifié d'outil déclaratif car l'utilisateur décrit l'état final souhaité du système (par exemple : "le paquet Nginx doit être présent", "le service doit être démarré"), plutôt que de fournir la liste exacte des commandes bash pour y parvenir. C'est Ansible qui se charge de vérifier l'état actuel du système et de déterminer les actions nécessaires (ou non) pour atteindre cet état cible. Cela garantit également l'idempotence (le fait de pouvoir relancer le script sans tout casser)...*

**Question 2 :**  
Pourquoi l’utilisation de variables est-elle essentielle dans un playbook ?  
  
*..Les variables rendent un playbook dynamique et réutilisable. Sans elles, les valeurs seraient écrites en "dur" dans le code (hardcodées). Grâce aux variables, on peut utiliser exactement le même playbook pour déployer des environnements très différents (comme le DEV et la PROD) simplement en modifiant le fichier de variables ou en passant des paramètres lors de l'exécution, sans jamais toucher à la logique du code source...*

**Question 3 :**  
En quoi Ansible facilite-t-il la gestion de plusieurs serveurs ?  
  
*..Ansible gère la multiplicité grâce à son fichier d'inventaire. Il suffit de lister les adresses IP ou les noms d'hôtes de plusieurs serveurs (en les regroupant par catégories, ex: [webservers], [dbservers]) pour qu'Ansible puisse s'y connecter simultanément via SSH. Une seule commande ansible-playbook permet de configurer 1, 10 ou 100 serveurs en parallèle, garantissant que tous auront exactement la même configuration, éliminant ainsi les erreurs humaines liées aux tâches manuelles répétitives...*

**Question 4 :**  
Quels sont les avantages et les limites d’Ansible dans un contexte DevOps ?   
  
**Avantages :** Agentless (sans agent) : Ne nécessite aucune installation de logiciel sur les serveurs cibles (utilise simplement SSH et Python).

Lisibilité : Utilise le format YAML, très facile à lire et à écrire pour des humains, ce qui facilite la collaboration entre développeurs et ops.

Idempotence : Applique uniquement les changements nécessaires.

**Limites :** Moins adapté que des outils comme Terraform pour la création pure d'infrastructure cloud (le provisioning de réseaux ou de machines virtuelles).

Peut devenir complexe à déboguer sur des erreurs très spécifiques.

Les performances peuvent être un défi sur de très grands parcs informatiques sans une configuration d'optimisation (tuning)...*
  
**Question 5 :**  
Quelle est la différence entre les modules copy et template dans Ansible ?   
  
**Le module copy** est utilisé pour transférer un fichier statique de la machine de contrôle (Ansible) vers le serveur cible, à l'octet près.

**Le module template** utilise le moteur Jinja2 pour lire le fichier source, l'analyser, et remplacer dynamiquement les variables (balises {{ variable }}) par leurs valeurs définies dans le playbook avant d'envoyer le résultat sur le serveur cible.

---------------------------------------------------
Séquence 5 : Atelier  
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

### Vous pouvez voir l'état de vos Codespace ici : https://github.com/codespaces  
  
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
