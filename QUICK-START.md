# 🚀 Guide de Démarrage Rapide - WSO2 dans VS Code

## ⚡ En 5 minutes

### 1️⃣ Ouvrir dans VS Code

```bash
# Ouvrir le dossier wso2-integration
cd C:\Users\hp\IdeaProjects\wso2-integration
code .
```

### 2️⃣ Configuration initiale (première fois uniquement)

#### A. Installer les extensions

Dans VS Code, appuyer sur **Ctrl+Shift+X**, puis installer :

- ✅ **WSO2 Micro Integrator** (by WSO2)
- ✅ **Extension Pack for Java** (by Microsoft)
- ✅ **XML** (by Red Hat)
- ✅ **REST Client** (by Huachao Mao) - optionnel

#### B. Configurer les chemins

Éditer `.vscode/settings.json` et ajuster ces chemins selon votre installation :

```json
{
    "java.home": "C:\\Program Files\\Java\\jdk-17",
    "wso2mi.runtime.location": "C:\\wso2mi-4.2.0"
}
```

### 3️⃣ Copier les artefacts WSO2

#### Option A : Drag & Drop (plus simple)

Dans l'explorateur VS Code :

1. Naviguer vers `src/api/`
2. **Drag & drop** `BillingGatewayAPI.xml` → `BillingIntegration/BillingIntegrationConfigs/src/main/synapse-config/api/`

3. Répéter pour :
   - `src/endpoints/*.xml` → `.../endpoints/`
   - `src/sequences/*.xml` → `.../sequences/`

#### Option B : Terminal VS Code (Ctrl+`)

```powershell
# Créer les dossiers si nécessaire
New-Item -ItemType Directory -Path "BillingIntegration\BillingIntegrationConfigs\src\main\synapse-config\api" -Force
New-Item -ItemType Directory -Path "BillingIntegration\BillingIntegrationConfigs\src\main\synapse-config\endpoints" -Force
New-Item -ItemType Directory -Path "BillingIntegration\BillingIntegrationConfigs\src\main\synapse-config\sequences" -Force

# Copier les fichiers
Copy-Item "src\api\*.xml" "BillingIntegration\BillingIntegrationConfigs\src\main\synapse-config\api\"
Copy-Item "src\endpoints\*.xml" "BillingIntegration\BillingIntegrationConfigs\src\main\synapse-config\endpoints\"
Copy-Item "src\sequences\*.xml" "BillingIntegration\BillingIntegrationConfigs\src\main\synapse-config\sequences\"
```

### 4️⃣ Démarrer les services

#### A. Démarrer Billing Service (dans IntelliJ)

Dans IntelliJ IDEA :
```bash
# Ouvrir billing-service
# Run → Run 'BillingServiceApplication'
# Ou terminal : mvn spring-boot:run
```

Vérifier : http://localhost:8081/billing/health

#### B. Démarrer WSO2 MI (dans VS Code)

**Méthode 1 - Avec l'extension** :

1. **Ctrl+Shift+P**
2. Taper : `WSO2: Start Micro Integrator`
3. Sélectionner le runtime

**Méthode 2 - Terminal** :

```powershell
# Dans terminal VS Code
cd C:\wso2mi-4.2.0\bin
.\micro-integrator.bat
```

Attendre le message :
```
WSO2 Micro Integrator started in 15 seconds
```

### 5️⃣ Tester l'API Gateway

Dans le terminal VS Code (Ctrl+`), ou utiliser **REST Client** :

```bash
# Health check
curl http://localhost:8290/api/health

# Test Legacy (SOAP → JSON)
curl -X POST http://localhost:8290/api/legacy/billing/invoices `
  -H "Content-Type: application/json" `
  -d '{\"clientId\":1001,\"amount\":1500.00,\"description\":\"Test\"}'

# Test REST v2
# 1. Login
$response = Invoke-RestMethod -Uri "http://localhost:8290/api/v2/billing/auth/login" `
  -Method Post -ContentType "application/json" `
  -Body '{"username":"admin","password":"password"}'

$token = $response.data.token
Write-Host "Token: $token"

# 2. Créer facture
Invoke-RestMethod -Uri "http://localhost:8290/api/v2/billing/invoices" `
  -Method Post -ContentType "application/json" `
  -Headers @{Authorization="Bearer $token"} `
  -Body '{"clientId":1001,"amount":2500.00,"description":"Test REST v2"}'
```

---

## 📊 Workflow quotidien

```
┌─────────────────────┐         ┌─────────────────────┐
│   IntelliJ IDEA     │         │      VS Code        │
│                     │         │                     │
│  billing-service/   │         │  wso2-integration/  │
│  ├─ src/main/java   │         │  ├─ src/api/        │
│  ├─ pom.xml         │         │  ├─ src/endpoints/  │
│  └─ Run ▶️          │         │  └─ src/sequences/  │
│     :8081           │         │                     │
└──────────┬──────────┘         └──────────┬──────────┘
           │                               │
           │  1. Modifier Java             │  2. Modifier XML
           │  2. Hot reload auto           │  3. Copier vers MI
           │                               │  4. WSO2 reload
           │                               │
           └───────────────┬───────────────┘
                          │
                    3. Tester via
                    curl/REST Client
                    :8290
```

### Scénario 1 : Modifier le backend Java

1. **IntelliJ** → Modifier `InvoiceRestController.java`
2. **IntelliJ** → Spring Boot auto-reload (quelques secondes)
3. **VS Code** → Tester via `curl http://localhost:8290/...`

### Scénario 2 : Modifier la médiation WSO2

1. **VS Code** → Modifier `JSONToSOAPSequence.xml`
2. **VS Code** → Copier vers WSO2 MI :
   ```powershell
   Copy-Item "src\sequences\JSONToSOAPSequence.xml" `
     "C:\wso2mi-4.2.0\repository\deployment\server\synapse-configs\default\sequences\"
   ```
3. WSO2 recharge automatiquement (voir logs)
4. **VS Code** → Tester

### Scénario 3 : Ajouter un nouveau endpoint

1. **VS Code** → Créer `NewServiceEndpoint.xml` dans `src/endpoints/`
2. **VS Code** → Copier vers WSO2 MI
3. **VS Code** → Modifier `BillingGatewayAPI.xml` pour utiliser le endpoint
4. Tester

---

## 🎯 Raccourcis VS Code utiles

| Raccourci | Action |
|-----------|--------|
| `Ctrl+Shift+P` | Command Palette (WSO2 commands) |
| ``Ctrl+` `` | Ouvrir/fermer terminal |
| `Ctrl+B` | Toggle sidebar |
| `Ctrl+Shift+E` | Explorateur de fichiers |
| `Ctrl+K Ctrl+S` | Keyboard shortcuts |
| `Ctrl+Shift+F` | Recherche globale |
| `Alt+Click` | Multi-curseur |

---

## 🐛 Problèmes fréquents

### ❌ "WSO2 runtime not found"

**Solution** : Vérifier `.vscode/settings.json` :
```json
"wso2mi.runtime.location": "C:\\wso2mi-4.2.0"
```

### ❌ "Port 8290 already in use"

**Solution** :
```powershell
# Trouver le processus
netstat -ano | findstr :8290

# Tuer le processus
taskkill /PID <PID> /F
```

### ❌ "Backend connection refused"

**Solution** : Vérifier que billing-service est démarré :
```bash
curl http://localhost:8081/billing/health
```

### ❌ "Cannot find Java"

**Solution** : Installer Java 17 et configurer :
```json
"java.home": "C:\\Program Files\\Java\\jdk-17"
```

---

## ✅ Checklist de vérification

Avant de tester, vérifier que :

- [ ] Java 17+ installé
- [ ] Maven installé
- [ ] WSO2 MI téléchargé et extrait
- [ ] Extensions VS Code installées
- [ ] `.vscode/settings.json` configuré
- [ ] Billing Service démarré (:8081) ✅
- [ ] WSO2 MI démarré (:8290) ✅
- [ ] Artefacts XML copiés ✅

**Si tout est vert, vous êtes prêt !** 🎉

---

## 📚 Ressources

- **VSCODE-SETUP.md** - Guide détaillé
- **INTEGRATION-GUIDE.md** - Guide d'intégration complet
- **README.md** - Documentation générale
- **WSO2 Docs** - https://wso2.com/integration/micro-integrator/

---

**Bonne intégration !** 🚀
