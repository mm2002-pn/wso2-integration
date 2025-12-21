# Configuration WSO2 dans VS Code

## 🎯 Vue d'ensemble

Ce guide explique comment configurer et utiliser le projet WSO2 Micro Integrator dans Visual Studio Code.

## 📋 Prérequis

1. **Visual Studio Code** installé
2. **Java JDK 11 ou 17** installé
3. **Maven** installé
4. **WSO2 Micro Integrator** téléchargé

## 🔧 Installation

### 1. Extension VS Code

Dans VS Code :

1. **Ctrl+Shift+X** → Extensions
2. Chercher **"WSO2 Micro Integrator"**
3. Installer l'extension officielle WSO2

### 2. Configuration Java et Maven

Créer `.vscode/settings.json` :

```json
{
    "java.configuration.updateBuildConfiguration": "automatic",
    "java.home": "C:\\Program Files\\Java\\jdk-17",
    "maven.executable.path": "C:\\apache-maven-3.9.0\\bin\\mvn.cmd",
    "wso2mi.runtime.location": "C:\\wso2mi-4.2.0"
}
```

**Ajuster les chemins** selon votre installation !

### 3. Télécharger WSO2 MI

Si pas déjà fait :

```bash
# Télécharger depuis
https://wso2.com/integration/micro-integrator/

# Extraire dans
C:\wso2mi-4.2.0\
```

## 📁 Structure du projet

```
wso2-integration/
├── BillingIntegration/
│   ├── BillingIntegrationConfigs/
│   │   ├── src/
│   │   │   └── main/
│   │   │       └── synapse-config/
│   │   │           ├── api/
│   │   │           │   └── BillingGatewayAPI.xml
│   │   │           ├── endpoints/
│   │   │           │   ├── SOAPBackendEndpoint.xml
│   │   │           │   └── RESTBackendEndpoint.xml
│   │   │           └── sequences/
│   │   │               ├── JSONToSOAPSequence.xml
│   │   │               ├── SOAPToJSONSequence.xml
│   │   │               └── ErrorHandlerSequence.xml
│   │   ├── pom.xml
│   │   └── artifact.xml
│   │
│   ├── BillingIntegrationCompositeExporter/
│   │   ├── pom.xml
│   │   └── target/
│   │       └── BillingIntegration_1.0.0.car
│   │
│   └── pom.xml (parent)
│
└── .vscode/
    └── settings.json
```

## 🚀 Utilisation dans VS Code

### Option 1 : Avec l'extension WSO2 (Recommandé)

#### Créer le projet

1. **Ctrl+Shift+P** → "WSO2: Create New Integration Project"
2. Nom : `BillingIntegration`
3. Type : `Micro Integrator Project`

#### Copier les artefacts

```bash
# Copier les fichiers XML existants
cp src/api/* BillingIntegration/BillingIntegrationConfigs/src/main/synapse-config/api/
cp src/endpoints/* BillingIntegration/BillingIntegrationConfigs/src/main/synapse-config/endpoints/
cp src/sequences/* BillingIntegration/BillingIntegrationConfigs/src/main/synapse-config/sequences/
```

#### Build le projet

Dans VS Code :
1. Ouvrir **Terminal** (Ctrl+`)
2. Naviguer vers le projet :
   ```bash
   cd BillingIntegration
   ```
3. Build :
   ```bash
   mvn clean install
   ```

Cela génère le fichier **CAR** (Carbon Application) :
```
BillingIntegrationCompositeExporter/target/BillingIntegration_1.0.0.car
```

### Option 2 : Manuellement (sans extension)

#### Copier les fichiers directement

```bash
# Créer la structure WSO2
mkdir -p C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\api
mkdir -p C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\endpoints
mkdir -p C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\sequences

# Copier les artefacts
copy src\api\*.xml C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\api\
copy src\endpoints\*.xml C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\endpoints\
copy src\sequences\*.xml C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\sequences\
```

## ▶️ Démarrer WSO2 MI

### Depuis VS Code

#### Avec l'extension WSO2 :

1. **Ctrl+Shift+P** → "WSO2: Start Micro Integrator"
2. Sélectionner le runtime : `C:\wso2mi-4.2.0`

#### Manuellement dans le terminal VS Code :

```bash
# Terminal VS Code (Ctrl+`)
cd C:\wso2mi-4.2.0\bin
.\micro-integrator.bat
```

### Logs dans VS Code

Le terminal affichera :
```
[2025-01-15 10:30:00] INFO - Server started in 12345ms
[2025-01-15 10:30:00] INFO - BillingGatewayAPI deployed
[2025-01-15 10:30:00] INFO - Management Console URL: https://localhost:9164/dashboard
```

## 🧪 Tester depuis VS Code

### Terminal intégré (Ctrl+`)

```bash
# Test health check
curl http://localhost:8290/api/health

# Test Legacy SOAP → JSON
curl -X POST http://localhost:8290/api/legacy/billing/invoices \
  -H "Content-Type: application/json" \
  -d "{\"clientId\":1001,\"amount\":1500.00,\"description\":\"Test VS Code\"}"

# Test REST v2
# 1. Login
curl -X POST http://localhost:8290/api/v2/billing/auth/login \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"admin\",\"password\":\"password\"}"

# 2. Utiliser le token...
```

### Avec extension REST Client (optionnel)

Installer **REST Client** extension, puis créer `test-api.http` :

```http
### Health Check
GET http://localhost:8290/api/health

### Login
POST http://localhost:8290/api/v2/billing/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password"
}

### Create Invoice (Legacy)
POST http://localhost:8290/api/legacy/billing/invoices
Content-Type: application/json

{
  "clientId": 1001,
  "amount": 1500.00,
  "description": "Test from VS Code"
}

### Create Invoice (REST v2)
POST http://localhost:8290/api/v2/billing/invoices
Authorization: Bearer {{token}}
Content-Type: application/json

{
  "clientId": 1001,
  "amount": 2500.00,
  "description": "Test REST v2 from VS Code"
}
```

Cliquer sur **"Send Request"** au-dessus de chaque requête.

## 🔄 Workflow de développement

### 1. Modifier les artefacts

Dans VS Code, éditer les fichiers XML :
```
BillingIntegrationConfigs/src/main/synapse-config/api/BillingGatewayAPI.xml
```

### 2. Hot Deploy (développement)

WSO2 MI détecte automatiquement les changements :

```bash
# Copier le fichier modifié
copy BillingIntegrationConfigs\src\main\synapse-config\api\BillingGatewayAPI.xml ^
     C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\api\

# WSO2 recharge automatiquement (logs dans terminal)
```

### 3. Build pour production

```bash
# Dans VS Code terminal
cd BillingIntegration
mvn clean install

# Déployer le CAR
copy BillingIntegrationCompositeExporter\target\BillingIntegration_1.0.0.car ^
     C:\wso2mi-4.2.0\repository\deployment\server\carbonapps\
```

## 🐛 Debug dans VS Code

### Activer les logs debug

1. Modifier `C:\wso2mi-4.2.0\conf\log4j2.properties` :
```properties
logger.synapse-api.level = DEBUG
```

2. Voir les logs en temps réel dans le terminal VS Code

### Breakpoints (avancé)

Pour débugger avec des breakpoints, utiliser **WSO2 Integration Studio** (Eclipse-based) au lieu de VS Code.

## 📊 Monitoring depuis VS Code

### Dashboard WSO2

Ouvrir dans navigateur depuis VS Code :
```
http://localhost:9164/dashboard
```

Credentials : `admin` / `admin`

### Logs en temps réel

Dans terminal VS Code :
```bash
# Windows
Get-Content C:\wso2mi-4.2.0\repository\logs\wso2carbon.log -Wait

# Ou utiliser extension "Log File Highlighter"
```

## 📝 Checklist de démarrage

- [ ] Extension WSO2 installée dans VS Code
- [ ] Java et Maven configurés (settings.json)
- [ ] WSO2 MI téléchargé et chemin configuré
- [ ] Projet BillingIntegration créé
- [ ] Artefacts XML copiés
- [ ] Billing Service démarré (IntelliJ - port 8081)
- [ ] WSO2 MI démarré (VS Code - port 8290)
- [ ] Tests API réussis ✅

## 🔗 Intégration avec IntelliJ

### Workflow complet

```
1. IntelliJ IDEA (billing-service)
   └─ Démarrer : mvn spring-boot:run
   └─ Port 8081

2. VS Code (wso2-integration)
   └─ Éditer : BillingGatewayAPI.xml
   └─ Démarrer : WSO2 MI
   └─ Port 8290

3. Tests
   └─ VS Code terminal : curl http://localhost:8290/...
```

### Tips

- Garder les deux IDE ouverts côte à côte
- Terminal IntelliJ → Billing Service
- Terminal VS Code → WSO2 MI + Tests
- Hot reload activé des deux côtés

## 🎯 Résumé rapide

```bash
# Dans VS Code

# 1. Ouvrir le dossier
cd C:\Users\hp\IdeaProjects\wso2-integration
code .

# 2. Créer projet WSO2 (Ctrl+Shift+P)
WSO2: Create New Integration Project → BillingIntegration

# 3. Copier les artefacts
# (Utiliser explorateur VS Code pour drag & drop les XML)

# 4. Build
cd BillingIntegration
mvn clean install

# 5. Démarrer WSO2 (Ctrl+Shift+P)
WSO2: Start Micro Integrator

# 6. Tester (terminal)
curl http://localhost:8290/api/health
```

---

**Vous êtes prêt !** 🚀
