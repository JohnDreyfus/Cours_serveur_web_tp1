# **TP 1 — Premier site avec Nginx**

## **Objectifs pédagogiques**

- Créer un **site statique simple (HTML + CSS)**
- Utiliser **Git et GitHub** pour versionner et déployer le projet
- Gérer la **connexion SSH** entre machine locale et serveur
- Déployer le projet sur une **VM avec Nginx**
- Configurer Nginx avec :
    - **HTTP → HTTPS** (certificat auto-signé)
    - **gzip** pour la compression
    - **logs dédiés**
---
## **Étape 1 — Créer le projet en local**
### **1.1 Créer l’arborescence**
```sh
premier-site/
├── index.html
└── assets/
    └── style.css
```
### **1.2 Contenu minimal**
**index.html**
```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Premier site</title>
  <link rel="stylesheet" href="assets/style.css">
</head>
<body>
  <header>
    <h1>Bienvenue sur mon premier site 🎉</h1>
    <p>Ce site est servi par <strong>Nginx</strong> avec HTTPS et Gzip.</p>
  </header>

  <main>
    <section>
      <h2>Présentation</h2>
      <p>Ceci est une simple page HTML + CSS déployée dans un mini-lab Nginx.</p>
    </section>

    <section>
      <h2>Bouton interactif</h2>
      <button onclick="document.getElementById('demo').textContent='👋 Bonjour, vous avez cliqué !';">
        Clique-moi
      </button>
      <p id="demo"></p>
    </section>
  </main>

  <footer>
    <p>&copy; 2025 - Mon Premier Site</p>
  </footer>
</body>
</html>
```

**assets/style.css**
```css
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  line-height: 1.6;
  background: #f4f4f4;
  color: #333;
}

header {
  background: #0077cc;
  color: white;
  padding: 20px;
  text-align: center;
}

h1 { margin: 0; }

main { padding: 20px; }

section {
  margin-bottom: 20px;
  background: white;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

button {
  background: #28a745;
  color: white;
  border: none;
  padding: 10px 15px;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
}
button:hover { background: #218838; }

footer {
  text-align: center;
  padding: 15px;
  background: #222;
  color: #bbb;
}
```

---
## **Étape 2 — Versionner avec Git & GitHub**
### **2.1 Générer une clé SSH (machine locale)**
```sh
ssh-keygen -t ed25519 -C "votre_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

Ajouter la clé publique dans GitHub :
_Settings → SSH and GPG Keys → New SSH key_.
### **2.2 Créer le dépôt local**
```sh
cd premier-site
git init
git add .
git commit -m "Initial commit - Premier site"
```
### **2.3 Créer le dépôt GitHub**
- Créez un dépôt vide sur GitHub (ex : premier-site).
### **2.4 Envoyer le projet**
```sh
git remote add origin git@github.com:<votre-compte>/premier-site.git
git branch -M main
git push -u origin main
```

---
## **Étape 3 — Préparer le serveur (VM)**
### **3.1 Générer une clé SSH (côté serveur)**
```sh
ssh-keygen -t ed25519 -C "votre_email@example.com"
cat ~/.ssh/id_ed25519.pub
```

Ajouter la clé publique à GitHub.

Tester :
```sh
ssh -T git@github.com
```
### **3.2 Cloner le projet**
```sh
cd /var/www
sudo git clone git@github.com:<votre-compte>/premier-site.git
sudo mv premier-site /var/www/premier-site
```
### **3.3 Permissions**
```sh
sudo chown -R root:www-data /var/www/premier-site
sudo find /var/www/premier-site -type d -exec chmod 755 {} \;
sudo find /var/www/premier-site -type f -exec chmod 644 {} \;
```

---
## **Étape 4 — Configurer Nginx**

### **4.1 Installer Nginx**
```sh
sudo apt update
sudo apt install -y nginx
```
### **4.2 Créer la config du site**
**/etc/nginx/sites-available/premier-site.conf**
```sh
server {
    listen 80;
    listen [::]:80;

    server_name localhost;

    root /var/www/premier-site;
    index index.html;

    access_log /var/log/nginx/premier-site.access.log;
    error_log  /var/log/nginx/premier-site.error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
### **4.3 Activer le site**
```sh
sudo ln -s /etc/nginx/sites-available/premier-site.conf /etc/nginx/sites-enabled/
```
### **4.4 Vérifier & recharger**
```sh
sudo nginx -t
sudo systemctl reload nginx
```
### **4.5 Tester**
Votre page doit s’afficher.
### **4.6 Vérifier les logs**
```sh
sudo tail -n 20 /var/log/nginx/premier-site.access.log
sudo tail -n 20 /var/log/nginx/premier-site.error.log
```

---
## **Étape 5 — Ajouter HTTPS (certificat auto-signé)**

### **5.1 Installer OpenSSL**
```sh
sudo apt install -y openssl
```
### **5.2 Créer le certificat**
```sh
sudo mkdir -p /etc/ssl/nginx
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/nginx/selfsigned.key \
  -out /etc/ssl/nginx/selfsigned.crt \
  -subj "/C=FR/ST=Local/L=Local/O=TP/OU=Nginx/CN=localhost"
```
### **5.3 Configurer le vhost HTTPS**
**/etc/nginx/sites-available/premier-site-ssl.conf**
```sh
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name localhost;

    ssl_certificate     /etc/ssl/nginx/selfsigned.crt;
    ssl_certificate_key /etc/ssl/nginx/selfsigned.key;

    root /var/www/premier-site;
    index index.html;

    access_log /var/log/nginx/premier-site-ssl.access.log;
    error_log  /var/log/nginx/premier-site-ssl.error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
### **5.4 Redirection HTTP → HTTPS**
Modifier **premier-site.conf** :
```sh
server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}
```
### **5.5 Activer et recharger**
```sh
sudo ln -s /etc/nginx/sites-available/premier-site-ssl.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---
## **Étape 6 — Activer Gzip**

**/etc/nginx/conf.d/gzip.conf**
```sh
gzip on;
gzip_comp_level 5;
gzip_min_length 256;
gzip_types
  text/plain
  text/css
  application/javascript
  application/json
  image/svg+xml;
```

Recharger Nginx :
```sh
sudo nginx -t
sudo systemctl reload nginx
```

---
## **Étape 7 — Tests finaux**

### **7.1 Redirection HTTP → HTTPS**
```sh
curl -I http://localhost
```
➡️ Doit renvoyer 301 Moved Permanently.
### **7.2 Page HTTPS**
```sh
curl -k https://localhost
```
➡️ Affiche le HTML de la page.
### **7.3 Vérifier Gzip**
```sh
curl -k -H "Accept-Encoding: gzip" -I https://192.168.1.165/assets/style.css
```
➡️ Doit contenir Content-Encoding: gzip.
### **7.4 Vérifier les logs**
```sh
sudo tail -n 20 /var/log/nginx/premier-site*.log
```
