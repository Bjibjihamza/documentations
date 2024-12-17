## Étapes pour automatiser le déploiement avec des scripts Bash
### 1.  Créer un script Bash pour le déploiement (par exemple, `deploy.sh`)
Dans ce script, vous allez inclure toutes les étapes nécessaires à l'installation et à la mise en ligne 
de votre application.Le script pourra être exécuté sur votre serveur à chaque mise à jour.


Variables (personnalisez ces variables selon votre projet)
```bash
FRONTEND_DIR="/var/www/frontend"
BACKEND_DIR="/var/www/backend"
GIT_REPO_BACKEND="git@github.com:your-username/backend-repo.git"
GIT_REPO_FRONTEND="git@github.com:your-username/frontend-repo.git"
NODE_VERSION="14"  # Exemple : version de Node.js
PORT=4000  # Port de l'API backend (modifiez selon votre configuration)
```
Étape 1 : Mise à jour du système et installation des dépendances
```bash
echo "Mise à jour du système..."
sudo apt update -y
sudo apt upgrade -y
```
Étape 2 : Installation de Node.js (si non déjà installé)
```bash
echo "Installation de Node.js..."
curl -fsSL https://deb.nodesource.com/setup_$NODE_VERSION.x | sudo -E bash -
sudo apt install -y nodejs
```
Vérification de l'installation de Node.js
```bash 
node -v
npm -v
```
Étape 3 : Installation de MongoDB (si non déjà installé)
```bash
echo "Installation de MongoDB..."
sudo apt install -y mongodb
```
Vérification de l'installation de MongoDB
```bash
sudo systemctl start mongodb
sudo systemctl enable mongodb
```

# Étape 4 : Cloner les dépôts Git (backend et frontend)
```bash
echo "Clonage du backend..."
if [ -d "$BACKEND_DIR" ]; then
  echo "Répertoire backend déjà existant, mise à jour..."
  cd $BACKEND_DIR && git pull origin main
else
  git clone $GIT_REPO_BACKEND $BACKEND_DIR
fi

echo "Clonage du frontend..."
if [ -d "$FRONTEND_DIR" ]; then
  echo "Répertoire frontend déjà existant, mise à jour..."
  cd $FRONTEND_DIR && git pull origin main
else
  git clone $GIT_REPO_FRONTEND $FRONTEND_DIR
fi

```
Étape 5 : Installer les dépendances Node.js pour le backend
```bash
echo "Installation des dépendances backend..."
cd $BACKEND_DIR
npm install
```
Étape 6 : Installer les dépendances Node.js pour le frontend (build React)
```bash
echo "Installation des dépendances frontend..."
cd $FRONTEND_DIR
npm install
```
Étape 7 : Build du frontend (React)
```bash
echo "Build du frontend React..."
cd $FRONTEND_DIR
npm run build
```
Étape 8 : Transfert des fichiers build du frontend vers le serveur web (Apache2)
```bash
echo "Transfert du frontend vers Apache2..."
sudo cp -r $FRONTEND_DIR/build/* /var/www/html/
```
Étape 9 : Démarrer le backend avec PM2
```bash
echo "Démarrage du backend avec PM2..."
cd $BACKEND_DIR
pm2 start server.js --name "mern-backend"
```
Étape 10 : Assurer que PM2 démarre au démarrage
```bash
pm2 startup
pm2 save
```
Étape 11 : Vérification des services
```bash
echo "Vérification de l'état du serveur Node.js..."
pm2 status
echo "Déploiement terminé avec succès!"
```
tout ca daans le deploy.sh 
```bash
#!/bin/bash

# Variables (personnalisez ces variables selon votre projet)
FRONTEND_DIR="/var/www/my-app"
BACKEND_DIR="/var/www/my-backend"
GIT_REPO_BACKEND="git@github.com:your-username/your-backend-repo.git"
GIT_REPO_FRONTEND="git@github.com:your-username/your-frontend-repo.git"
NODE_VERSION="14"  # Exemple : version de Node.js
PORT=5000  # Port de l'API backend (modifiez selon votre configuration)

# Étape 1 : Mise à jour du système et installation des dépendances
echo "Mise à jour du système..."
sudo apt update -y
sudo apt upgrade -y

# Étape 2 : Installation de Node.js (si non déjà installé)
echo "Installation de Node.js..."
curl -fsSL https://deb.nodesource.com/setup_$NODE_VERSION.x | sudo -E bash -
sudo apt install -y nodejs

# Vérification de l'installation de Node.js
node -v
npm -v

# Étape 3 : Installation de MongoDB (si non déjà installé)
echo "Installation de MongoDB..."
sudo apt install -y mongodb

# Vérification de l'installation de MongoDB
sudo systemctl start mongodb
sudo systemctl enable mongodb

# Étape 4 : Cloner les dépôts Git (backend et frontend)
echo "Clonage du backend..."
if [ -d "$BACKEND_DIR" ]; then
  echo "Répertoire backend déjà existant, mise à jour..."
  cd $BACKEND_DIR && git pull origin main
else
  git clone $GIT_REPO_BACKEND $BACKEND_DIR
fi

echo "Clonage du frontend..."
if [ -d "$FRONTEND_DIR" ]; then
  echo "Répertoire frontend déjà existant, mise à jour..."
  cd $FRONTEND_DIR && git pull origin main
else
  git clone $GIT_REPO_FRONTEND $FRONTEND_DIR
fi

# Étape 5 : Installer les dépendances Node.js pour le backend
echo "Installation des dépendances backend..."
cd $BACKEND_DIR
npm install

# Étape 6 : Installer les dépendances Node.js pour le frontend (build React)
echo "Installation des dépendances frontend..."
cd $FRONTEND_DIR
npm install

# Étape 7 : Build du frontend (React)
echo "Build du frontend React..."
cd $FRONTEND_DIR
npm run build

# Étape 8 : Transfert des fichiers build du frontend vers le serveur web ( Apache2)
echo "Transfert du frontend vers Apache2..."
sudo cp -r $FRONTEND_DIR/build/* /var/www/html/


# Étape 9 : Démarrer le backend avec PM2
echo "Démarrage du backend avec PM2..."
cd $BACKEND_DIR
pm2 start server.js --name "mern-backend"

# Étape 10 : Assurer que PM2 démarre au démarrage
pm2 startup
pm2 save

# Étape 11 : Vérification des services
echo "Vérification de l'état du serveur Node.js..."
pm2 status

echo "Déploiement terminé avec succès!"

```
Explication des différentes étapes dans le script
  - Mise à jour du système : Vous mettez à jour le système d'exploitation pour vous assurer que toutes les dépendances sont à jour.
  - Installation de Node.js et MongoDB : Si Node.js ou MongoDB n'est pas déjà installé sur votre serveur, ces commandes permettent de les installer.
  - Clonage des dépôts Git : Le script clone les dépôts backend et frontend depuis GitHub (ou tout autre dépôt git que vous utilisez) vers le serveur. Si les dépôts existent déjà, il effectue un git pull pour récupérer les dernières mises à jour.
  - Installation des dépendances : Chaque répertoire (frontend et backend) a ses propres dépendances, qui sont installées avec npm install.
  - Build du frontend : Le frontend React est construit avec npm run build pour générer les fichiers statiques. Ces fichiers sont ensuite copiés dans le répertoire où  Apache2 peut les servir.
  - Démarrage du backend avec PM2 : PM2 est utilisé pour démarrer le backend en mode production et pour s'assurer que le serveur Node.js tourne en arrière-plan.
  - Démarrage automatique avec PM2 : pm2 startup configure le serveur pour que PM2 redémarre l'application automatiquement après un redémarrage du serveur.
Vérification du statut de PM2 : Le script termine par une vérification pour s'assurer que le backend fonctionne correctement.
#### 3. Rendre le script executable et l'executer 
Rendre le script exécutable
```bash
chmod +x deploy.sh
```
```bash
./deploy.sh
```

### 4.  Automatisation avec Cron (Optionnel)
Si vous souhaitez que ce déploiement soit effectué automatiquement à intervalles réguliers (par exemple, chaque fois qu'il y a une mise à jour dans votre dépôt Git), vous pouvez utiliser cron jobs.
Exemple de cron job pour exécuter le script tous les jours à minuit :
 #### 4.1 Ouvrez votre crontab : 
 ```bash
crontab -e
```

#### 4.2 Ajoutez la ligne suivante pour exécuter le script à minuit chaque jour :
```bash
0 0 * * * /path/to/deploy.sh
```

#### Surveillance du déploiment 
Vous pouvez ajouter des alertes de logs pour vérifier si le déploiement a échoué ou réussi. Par exemple, dans votre script, vous pouvez rediriger la sortie vers un fichier log :

```bash
./deploy.sh > deploy.log 2>&1
```
