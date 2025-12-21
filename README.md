# WSO2 Micro Integrator - Billing Service Integration

Configuration WSO2 MI pour intégrer et exposer les services de facturation (SOAP legacy + REST v2).

## 🎯 Objectifs

1. **API Gateway unifié** - Point d'entrée unique pour tous les services
2. **Exposition SOAP legacy** - `/api/legacy/billing` (SOAP → REST/JSON)
3. **Exposition REST v2** - `/api/v2/billing` (passthrough)
4. **Médiation SOAP/XML → JSON** - Conversion automatique
5. **Routing intelligent** - Selon version et type de requête

## 📁 Structure

```
wso2-integration/
├── src/
│   ├── api/
│   │   ├── BillingGatewayAPI.xml          # API Gateway principal
│   │   ├── BillingLegacyAPI.xml           # API pour SOAP legacy
│   │   └── BillingRestAPI.xml             # API pour REST v2
│   ├── endpoints/
│   │   ├── SOAPBackendEndpoint.xml        # Endpoint SOAP legacy
│   │   └── RESTBackendEndpoint.xml        # Endpoint REST v2
│   ├── sequences/
│   │   ├── SOAPToJSONSequence.xml         # Médiation SOAP → JSON
│   │   ├── JSONToSOAPSequence.xml         # Médiation JSON → SOAP
│   │   └── ErrorHandlerSequence.xml       # Gestion erreurs
│   └── local-entries/
│       └── SOAPTemplates.xml              # Templates SOAP
├── deployment.toml                         # Configuration MI
├── docker-compose.yml                      # Déploiement Docker
└── README.md
```

## 🚀 Déploiement

### Avec WSO2 Micro Integrator

1. **Installation WSO2 MI** :
   ```bash
   # Télécharger depuis https://wso2.com/integration/micro-integrator/
   # Extraire dans un répertoire
   ```

2. **Copier les artefacts** :
   ```bash
   cp -r src/* <WSO2_MI_HOME>/repository/deployment/server/synapse-configs/default/
   ```

3. **Démarrer WSO2 MI** :
   ```bash
   cd <WSO2_MI_HOME>/bin
   ./micro-integrator.sh  # Linux/Mac
   micro-integrator.bat   # Windows
   ```

### Avec Docker

```bash
# Build et démarrer
docker-compose up -d

# Vérifier les logs
docker-compose logs -f wso2mi

# Arrêter
docker-compose down
```

## 🌐 Endpoints exposés

### API Gateway (Port 8290)

**Base URL** : `http://localhost:8290`

#### SOAP Legacy (converti en REST/JSON)

```bash
# Créer une facture (SOAP → JSON)
POST http://localhost:8290/api/legacy/billing/invoices
Content-Type: application/json

{
  "clientId": 1001,
  "amount": 1500.00,
  "description": "Service legacy"
}

# Récupérer une facture
GET http://localhost:8290/api/legacy/billing/invoices/1

# Payer une facture
PUT http://localhost:8290/api/legacy/billing/invoices/1/pay
{
  "paymentMethod": "TRANSFER"
}
```

#### REST v2 (passthrough)

```bash
# Authentification
POST http://localhost:8290/api/v2/billing/auth/login
{
  "username": "admin",
  "password": "password"
}

# Créer une facture
POST http://localhost:8290/api/v2/billing/invoices
Authorization: Bearer <token>
{
  "clientId": 1001,
  "amount": 1500.00,
  "description": "Service moderne"
}

# Récupérer une facture
GET http://localhost:8290/api/v2/billing/invoices/1
Authorization: Bearer <token>

# Générer PDF
GET http://localhost:8290/api/v2/billing/invoices/1/pdf
Authorization: Bearer <token>
```

## 🔄 Flux de médiation

### SOAP Legacy → REST/JSON

```
Client (JSON)
    ↓
WSO2 Gateway (/api/legacy/billing)
    ↓
JSONToSOAPSequence (conversion JSON → SOAP)
    ↓
SOAP Backend (http://localhost:8081/billing/ws)
    ↓
SOAPToJSONSequence (conversion SOAP → JSON)
    ↓
Client (JSON)
```

### REST v2 (passthrough)

```
Client (JSON + JWT)
    ↓
WSO2 Gateway (/api/v2/billing)
    ↓
REST Backend (http://localhost:8081/billing/api/v2)
    ↓
Client (JSON)
```

## 🧪 Tests

### Test SOAP Legacy via Gateway

```bash
# Via WSO2 (SOAP → JSON)
curl -X POST http://localhost:8290/api/legacy/billing/invoices \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": 1001,
    "amount": 2500.00,
    "description": "Test via WSO2"
  }'
```

### Test REST v2 via Gateway

```bash
# 1. Login
TOKEN=$(curl -X POST http://localhost:8290/api/v2/billing/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}' \
  | jq -r '.data.token')

# 2. Créer facture
curl -X POST http://localhost:8290/api/v2/billing/invoices \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": 1001,
    "amount": 3500.00,
    "description": "Test REST v2 via WSO2"
  }'
```

## 📊 Monitoring

- **WSO2 MI Dashboard** : http://localhost:9164/dashboard
- **Health Check** : http://localhost:8290/health

## 🔧 Configuration

### Backend Services

Les backends doivent être démarrés avant WSO2 :

```bash
# 1. Billing Service (SOAP + REST)
cd billing-service
mvn spring-boot:run
# → http://localhost:8081/billing

# 2. WSO2 MI
cd wso2-integration
docker-compose up -d
# → http://localhost:8290
```

### Variables d'environnement

`.env` :
```env
BILLING_SOAP_BACKEND=http://localhost:8081/billing/ws
BILLING_REST_BACKEND=http://localhost:8081/billing/api/v2
WSO2_PORT=8290
```

## 🔐 Sécurité

- **SOAP Legacy** : Pas d'authentification (legacy)
- **REST v2** : JWT requis (passthrough du header)
- **WSO2** : Peut ajouter OAuth2, API Key, etc.

## 📝 Notes importantes

1. **SOAP Backend** : Doit être accessible sur `http://localhost:8081/billing/ws`
2. **REST Backend** : Doit être accessible sur `http://localhost:8081/billing/api/v2`
3. **Conversion** : SOAP/XML automatiquement converti en JSON
4. **Routing** : Basé sur le préfixe `/api/legacy/` ou `/api/v2/`

---

**Version** : 1.0.0
**Auteur** : Tech Solutions
"# wso2-integration" 
