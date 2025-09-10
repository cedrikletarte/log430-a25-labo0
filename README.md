# Labo 00 – Infrastructure (Git, Docker, CI/CD)
<img src="https://upload.wikimedia.org/wikipedia/commons/2/2a/Ets_quebec_logo.png" width="250">    
ÉTS - LOG430 - Architecture logicielle - Chargé de laboratoire: Gabriel C. Ullmann, Automne 2025.    

## 🎯 Objectifs d’apprentissage
- Comprendre comment utiliser des conteneurs avec **Docker**.
- Apprendre à écrire et exécuter des tests automatisés avec **pytest**.
- Mettre en place un pipeline **CI/CD** avec **GitLab** et **Docker**.
- Accéder à un serveur via **SSH** et vérifier la disponibilité des ressources computationnelles (CPU, RAM, espace disque)
- Savoir combiner les outils de développement modernes (VS Code, **Git**, **Docker**) pour lancer un cycle de développement logiciel.

---

## ⚙️ Setup
Dans ce laboratoire, vous travaillerez sur une application calculatrice. Cette calculatrice est volontairement très simple afin que nous puissions nous concentrer sur la configuration du projet, ainsi que sur la création et la maintenance d'un pipeline CI/CD. 

Dans les prochains laboratoires, nous verrons des architectures plus complexes et nous travaillerons avec une variété d'outils logiciels et de concepts architecturaux.

> ⚠️ IMPORTANT : Avant de commencer le setup et les activités, veuillez lire la documentation architecturale dans le répertoire `/docs/arc42/docs.pdf`.

### 1. Faites un fork et clonez le dépôt GitLab

```bash
git clone https://github.com/guteacher/log430-a25-labo0
cd log430-a25-labo0
```

### 2. Créez le conteneur Docker
Construisez le conteneur Docker `labo0-calculator` et lancez-le de manière itérative.
```bash
docker build -t labo0-calculator .
docker run -it labo0-calculator 
```

Dans un autre instance du terminal, vous pouvez vérifier si le conteneur a été correctement démarré. 
```bash
docker ps
```

> 📝 **NOTE** : Si vous exécutez des conteneurs sur votre ordinateur de développement, vous pouvez utiliser [Docker Desktop](https://www.docker.com/products/docker-desktop/) pour faciliter la gestion des conteneurs. Lorsque vous déployez sur un serveur, vous devrez utiliser l'interface de ligne de commande. Il existe des outils avancés de gestion Docker pour les serveurs, tels que [Portainer](https://www.portainer.io/), mais nous ne les aborderons pas ici. 

### 3. Créez un environnement virtuel Python sur votre ordinateur

#### Sur Linux/Mac
```bash
python -m venv .venv/labo0
source .venv/labo0/bin/activate
```

#### Sur Windows
```bash
python -m venv .venv/labo0
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser # Si nécessaire
.venv\labo0\Scripts\activate.ps1
```

### 4. Installez les dépendances Python

```bash
pip install -r requirements.txt
```

### 5. Lancez l’application

```bash
cd src
python calculator.py
```

---

## 🧪 Activités

### 1. Écrivez les tests

Dans le fichier `test_calculator.py`, écrivez des tests pour les fonctions définies dans `calculator.py`.

```python
def test_addition():
    assert addition(2, 3) == 5
```
Pour lancer les tests localement:

```bash
pytest
```

Si cela ne marche pas dans votre environnement, vous pouvez essayer:
```bash
python3 -m pytest
```

> 💡 **Question 1** : Si l’un des tests échoue à cause d’un bug, comment pytest signale-t-il l’erreur et aide-t-il à la localiser ? Rédigez un test qui provoque volontairement une erreur, puis montrez la sortie du terminal obtenue.

Pytest affiche le test ayant échoué (nom du test, fichier et ligne), la trace d'appel et le message d'assertion qui montre la valeur attendue et la valeur obtenue (ou un diff).

====================================================================== FAILURES ======================================================================= 
_________________________________________________________________ test_addition_error _________________________________________________________________ 

    def test_addition_error():
        my_calculator = Calculator()
>       assert my_calculator.addition(2, 3) == 6  # This test is supposed to fail
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
E       assert 5 == 6
E        +  where 5 = addition(2, 3)
E        +    where addition = <calculator.Calculator object at 0x000001EF82360550>.addition

tests\test_calculator.py:19: AssertionError
=============================================================== short test summary info =============================================================== 
FAILED tests/test_calculator.py::test_addition_error - assert 5 == 6
1 failed, 5 passed in 0.41s


### 2. Ajoutez une étape à la pipeline CI (intégration continue)

Ajoutez une étape (step) dans `.github/workflows/.gitlab-ci.yml` pour que GitLab exécute les tests automatiquement à chaque push. 

### 3. Versionnez votre code

Si tous les tests passent :

```bash
git add .
git commit -m "Tests pour calculator.py"
git push
```

Gitlab éxecutera les tests dans son serveur, et ils devront passer également si ils sont corrects.

> 💡 **Question 2** :  Que fait GitLab pendant les étapes de « setup » et « checkout » ? Veuillez inclure la sortie du terminal Gitlab CI dans votre réponse.

#### Setup
prépare l’environnement d’exécution du runner : provisionnement de la VM/conteneur, installation des outils demandés (ex. Python) et téléchargement/caching des actions ou dépendances. Il configure aussi les variables et secrets, crée le répertoire de travail et initialise le dépôt pour le job.

Current runner version: '2.328.0'
Runner Image Provisioner
Operating System
Runner Image
GITHUB_TOKEN Permissions
Secret source: Actions
Prepare workflow directory
Prepare all required actions
Getting action download info
Download action repository 'actions/checkout@v3' (SHA:f43a0e5ff2bd294095638e18286ca9a3d1956744)
Download action repository 'actions/setup-python@v4' (SHA:7f4fc3e22c37d6ff65e88745f38bd3157c663f7c)
Complete job name: build

#### Checkout
Pendant le checkout, le runner clone ou synchronise le dépôt dans le répertoire de travail, configure Git (safe.directory), initialise les sous-modules/LFS si nécessaire et met en place l’authentification pour accéder au repo. Ensuite il récupère la révision demandée et positionne la branche/commit en HEAD, prêt pour l’exécution du job.

Run actions/checkout@v3
Syncing repository: cedrikletarte/log430-a25-labo0
Getting Git version info
Temporarily overriding HOME='/home/runner/work/_temp/e6b98f4e-413f-4bf3-a591-f6a1fbc28d20' before making global git config changes
Adding repository directory to the temporary git global config as a safe directory
/usr/bin/git config --global --add safe.directory /home/runner/work/log430-a25-labo0/log430-a25-labo0
Deleting the contents of '/home/runner/work/log430-a25-labo0/log430-a25-labo0'
Initializing the repository
Disabling automatic garbage collection
Setting up auth
Fetching the repository
Determining the checkout info
Checking out the ref
/usr/bin/git log -1 --format='%H'
'6b135160d70aa4b3b73909fd482981d0f4b57392'

### 4. Automatiser déploiement continu (CD)
Après l’exécution des tests, déployez l’application dans un serveur ou machine virtuelle via SSH manuellement:

```bash
ssh $my_username@$my_hostname
git clone https://github.com/guteacher/log430-a25-labo0
cd log430-a25-labo0
```

>  📝 **NOTE** : N'oubliez pas d'installer Python, Docker et toutes les dépendances nécessaires sur le serveur de déploiement.

Procédez ensuite à la mise en place de l’automatisation du déploiement continu (CD) dans la machine virtuelle à l’aide de GitLab et de scripts Bash. Les approches les plus courantes pour implémenter le CD consistent à effectuer le déploiement via SSH ou à utiliser un webhook. Si votre SSH est protégé par mot de passe, vous devrez peut-être créer aussi un environnement et des secrets. Vous trouverez ci-dessous quelques liens de réference :

SSH:
- https://www.cyberciti.biz/faq/noninteractive-shell-script-ssh-password-provider/ 

GitLab:
- https://docs.gitlab.com/user/project/integrations/webhooks/#create-a-webhook
- https://docs.gitlab.com/user/project/integrations/webhook_events/#job-events

GitHub:
- https://docs.github.com/en/webhooks
- https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments#creating-an-environment
- https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets#creating-secrets-for-an-environment

> ⚠️ IMPORTANT : Il n'est pas recommandé d'écrire les noms d'utilisateur et les mots de passe en « plain text » dans un script CI/CD. Veuiller utiliser les secrets dans GitHub/GitLab.


> 💡 **Question 3** : Quel approache et quelles commandes avez-vous exécutées pour automatiser le déploiement continu de l'application dans la machine virtuelle ? Veuillez inclure les sorties du terminal et les scripts bash dans votre réponse.

Quelques commandes utiles pour vérifier l’état des ressources :
```bash
free -h   # Vérifier l’utilisation de la RAM
top       # Vérifier l’utilisation du CPU et les processus en cours
df -h     # Vérifier l’espace disque disponible
```

> 💡 **Question 4** : Quel type d'informations pouvez-vous obtenir via la commande « top » ? Veuillez inclure la sortie du terminal dans votre réponse.
---

## 📦 Livrables

- Code compressé en `.zip` contenant **l'ensemble du code source** du projet Labo 00.
- Rapport `.pdf` répondant aux 4 questions presentées dans ce fichier. Il est **obligatoire** d'ajouter du code ou des sorties de terminal pour illustrer chacune de vos réponses.