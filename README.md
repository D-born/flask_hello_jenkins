# TP Jenkins - Docker - Kubernetes

## Objectif

Mettre en place un pipeline CI/CD avec Jenkins, Docker et Kubernetes pour tester et deployer une application Flask.

## Etat actuel du projet

Ce projet contient deja:

- une application Flask
- des tests unitaires
- un `Dockerfile`
- un `Jenkinsfile`
- des manifests Kubernetes
- une branche `feature1` pour la partie TDD

## Arborescence utile

```text
flask_app/
├── app.py
├── test.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── values.yaml
├── kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
└── README.md
```

## Etapes du TP

### 1. Jenkins avec Docker

Dans `tp_jenkins/jenkins_simple`:

- creer `docker-compose.yml`
- creer `home/`
- creer `jenkins_data/`
- lancer Jenkins:

```powershell
cd C:\Users\kenny\Desktop\tp_jenkins\jenkins_simple
docker compose up -d
docker compose logs
```

- ouvrir Jenkins dans le navigateur
- initialiser Jenkins
- installer les plugins recommandes

### 2. Premier pipeline hello

Dans Jenkins:

- creer un pipeline `hello`
- coller ce script:

```groovy
pipeline {
  agent any
  stages {
    stage("Hello") {
      steps {
        echo 'Hello World'
      }
    }
  }
}
```

- lancer le build
- verifier `Console Output`

### 3. Application Flask

Dans `flask_app`:

- creer `app.py`
- creer `test.py`
- creer `requirements.txt`
- creer `.gitignore`

Lancer les tests localement:

```powershell
cd C:\Users\kenny\Desktop\tp_jenkins\flask_app
.\.venv\Scripts\python.exe test.py -v
```

Resultat attendu:

```text
Ran 3 tests ...
OK
```

### 4. Dockerisation

Construire l'image:

```powershell
cd C:\Users\kenny\Desktop\tp_jenkins\flask_app
docker build -t flask_hello .
```

Tester le conteneur:

```powershell
docker run --rm flask_hello ./test.py -v
```

### 5. Registry local

Lancer le registry:

```powershell
docker run -d -p 4000:5000 --name registry registry
```

Tagger et pousser:

```powershell
docker tag flask_hello localhost:4000/flask_hello
docker push localhost:4000/flask_hello
```

### 6. Git et GitHub

Initialiser Git:

```powershell
cd C:\Users\kenny\Desktop\tp_jenkins\flask_app
git init
git add .
git commit -m "Initial Flask app"
git branch -M main
git remote add origin git@github.com:D-born/flask_hello_jenkins.git
git push -u origin main
```

### 7. Jenkins sur Kubernetes

Activer Kubernetes dans Docker Desktop puis deployer Jenkins avec Helm.

Commandes utilisees:

```powershell
kubectl create namespace jenkins
kubectl config set-context --current --namespace=jenkins
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm show values jenkins/jenkins > values.yaml
```

Modifier `values.yaml`:

- utilisateur admin
- mot de passe admin
- exposition Jenkins

Installer Jenkins:

```powershell
helm install jenkins jenkins/jenkins -n jenkins -f values.yaml
```

Acces local:

```powershell
kubectl port-forward svc/jenkins 32000:8080 -n jenkins
```

Navigateur:

```text
http://localhost:32000
```

### 8. Pipeline Git avec Jenkinsfile

Le `Jenkinsfile` utilise un agent Kubernetes.

Le pipeline valide:

- checkout Git
- installation des dependances
- execution des tests

### 9. Deploiement Kubernetes

Les manifests sont dans:

- `kubernetes/deployment.yaml`
- `kubernetes/service.yaml`

Le pipeline appelle:

```sh
kubectl apply -f ./kubernetes/deployment.yaml
kubectl apply -f ./kubernetes/service.yaml
```

Verification:

```powershell
kubectl get deployment -n jenkins
kubectl get pods -n jenkins
kubectl get svc -n jenkins
```

### 10. Partie TDD

Creation de la branche:

```powershell
git checkout -b feature1
```

Ajout du test en echec dans `test.py`:

```python
    def test_new_route(self):
        name = 'Simon'
        rv = self.app.get(f'/feature/{name}')
        self.assertEqual(rv.status, '200 OK')
```

Push du test:

```powershell
git add test.py
git commit -m "Add failing feature route test"
git push -u origin feature1
```

Correction dans `app.py`:

```python
@app.route('/feature/<username>')
def feature_user(username):
    return 'Hello %s!\n' % username
```

Verifier localement:

```powershell
.\.venv\Scripts\python.exe test.py -v
```

Push de la correction:

```powershell
git add app.py
git commit -m "Implement feature route"
git push
```

### 11. Ajustement du deploiement

Le deploiement final utilise:

```yaml
image: pythontest:latest
imagePullPolicy: Never
```

Puis:

```powershell
docker tag flask_hello pythontest:latest
kubectl apply -f .\kubernetes\deployment.yaml -n jenkins
kubectl rollout restart deployment pythontest -n jenkins
```

## Verification finale

Verifier l'etat Git:

```powershell
git status
git branch -vv
git log --oneline -5
```

Verifier Kubernetes:

```powershell
kubectl get deployment -n jenkins
kubectl get pods -n jenkins
kubectl get svc -n jenkins
```

## Resultat attendu

Le projet final doit montrer:

- Jenkins operationnel
- pipeline de test
- application Flask testee
- image Docker construite
- deploiement Kubernetes
- branche `feature1` avec TDD
