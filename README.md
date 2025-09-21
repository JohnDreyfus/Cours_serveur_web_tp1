# TP 1 - premier site
## **Objectif du TP**
Mettre en place un **premier site web statique (HTML + CSS)** et le déployer sur un serveur **Nginx**, avec gestion via **Git/GitHub** et connexion sécurisée en **SSH**.

À la fin du TP, votre site devra être accessible depuis votre VM via **Nginx**.

---
## **Étape 1 — Préparer le projet en local**

1. Créez un dossier nommé **premier-site/** avec l’arborescence suivante :
```
premier-site/
├── index.html
└── assets/
    └── style.css
```

2. Créez le fichier index.html contenant une structure simple de page HTML (titre, en-tête, paragraphe, bouton interactif, etc.).
    
3. Créez le fichier assets/style.css pour mettre en forme la page (mise en page, couleurs, styles du bouton, etc.).
    
4. Vérifiez en ouvrant index.html dans un navigateur local que la page s’affiche correctement.

---
## **Étape 2 — Gérer le projet avec Git & GitHub**
### **Côté client (votre machine locale)**
1. Configurez une **nouvelle paire de clés SSH** pour GitHub.
2. Ajoutez la clé publique à votre compte GitHub (voir documentation GitHub).
	https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
3. Initialisez un dépôt Git dans votre dossier premier-site/.
4. Faites un premier commit avec vos fichiers.
5. Créez un dépôt vide sur GitHub (sans README ni fichiers automatiques).
6. Poussez votre projet local vers GitHub en utilisant **l’URL SSH**.

---
## **Étape 3 — Préparer le serveur (VM)**
### **Côté serveur (votre VM)**
1. Générez une **nouvelle paire de clés SSH** sur la VM et configurez-la également sur votre compte GitHub.
2. Depuis la VM, testez la connexion SSH avec GitHub.
3. Clonez votre projet GitHub dans le répertoire /var/www/ (ou un autre dossier prévu pour Nginx).
4. Vérifiez les droits et permissions de vos fichiers pour qu’ils soient lisibles par Nginx.

---
## **Étape 4 — Configurer Nginx**
1. Installez Nginx si ce n’est pas déjà fait.
2. Modifiez ou créez une configuration de serveur virtuel (server block) pour Nginx afin d’afficher votre site statique.
    - Le serveur doit répondre sur le port **80 (HTTP)**.
    - Le **document root** doit pointer vers le dossier de votre projet (/var/www/premier-site).
    
3. Activez la configuration et rechargez Nginx.
4. Vérifiez dans votre navigateur ou avec curl que le site s’affiche correctement.

---
## **Résultats attendus**
- Votre projet **premier-site** est versionné avec Git et disponible sur GitHub.
- Votre serveur VM est connecté à GitHub via SSH et peut cloner/puller le projet.
- Nginx affiche correctement votre site statique à l’adresse de la VM.
