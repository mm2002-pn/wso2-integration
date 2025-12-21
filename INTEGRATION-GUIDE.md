# Guide d'Intégration WSO2 Micro Integrator

## 🎯 Vue d'ensemble

Ce guide explique comment WSO2 MI intègre les services de facturation :

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Applications                       │
└────────────────┬───────────────────────┬────────────────────┘
                 │                       │
      /api/legacy/billing     /api/v2/billing
                 │                       │
                 ▼                       ▼
┌─────────────────────────────────────────────────────────────┐
│              WSO2 Micro Integrator (Port 8290)              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          BillingGatewayAPI (API Gateway)            │   │
│  └─────┬──────────────────────────────────┬────────────┘   │
│        │                                  │                │
│   ┌────▼────┐                        ┌────▼────┐          │
│   │ Legacy  │                        │  REST   │          │
│   │ Branch  │                        │ Branch  │          │
│   └────┬────┘                        └────┬────┘          │
│        │                                  │                │
│ ┌──────▼────────┐                         │                │
│ │ JSONToSOAP    │                         │                │
│ │   Sequence    │                         │                │
│ └──────┬────────┘                         │                │
│        │                                  │                │
│ ┌──────▼────────────┐            ┌────────▼────────┐      │
│ │ SOAP Backend      │            │ REST Backend    │      │
│ │ Endpoint          │            │ Endpoint        │      │
│ └──────┬────────────┘            └────────┬────────┘      │
└────────┼──────────────────────────────────┼───────────────┘
         │                                  │
         ▼                                  ▼
┌─────────────────────┐          ┌──────────────────────┐
│  Billing Service    │          │  Billing Service     │
│  SOAP (Legacy)      │          │  REST v2 (Modern)    │
│  :8081/billing/ws   │          │  :8081/billing/api/v2│
└─────────────────────┘          └──────────────────────┘
```

## 📊 Flux de données détaillé

### 1️⃣ Requête Legacy (SOAP → REST)

**Client envoie JSON** → **WSO2 convertit en SOAP** → **Backend SOAP** → **WSO2 convertit en JSON** → **Client reçoit JSON**

```bash
# Client appelle
POST http://localhost:8290/api/legacy/billing/invoices
{
  "clientId": 1001,
  "amount": 1500.00,
  "description": "Test"
}

↓ JSONToSOAPSequence

# WSO2 envoie à SOAP
<soap:CreateInvoiceRequest>
  <clientId>1001</clientId>
  <amount>1500.00</amount>
  <description>Test</description>
</soap:CreateInvoiceRequest>

↓ SOAP Backend traite

<soap:CreateInvoiceResponse>
  <invoiceId>123</invoiceId>
  <status>SUCCESS</status>
</soap:CreateInvoiceResponse>

↓ SOAPToJSONSequence

# Client reçoit
{
  "success": true,
  "data": {
    "invoiceId": 123
  }
}
```

### 2️⃣ Requête REST v2 (passthrough)

**Client envoie JSON** → **WSO2 passthrough** → **Backend REST** → **Client reçoit JSON**

```bash
# Client appelle
POST http://localhost:8290/api/v2/billing/invoices
Authorization: Bearer <token>
{
  "clientId": 1001,
  "amount": 1500.00
}

↓ WSO2 passthrough (+ JWT)

# Backend REST traite
POST http://localhost:8081/billing/api/v2/invoices
Authorization: Bearer <token>

↓ Backend répond

# Client reçoit
{
  "success": true,
  "data": {
    "id": 123,
    "status": "PENDING"
  }
}
```

## 🔄 Mapping des opérations

### Legacy SOAP → REST JSON

| HTTP Method | Path | SOAP Operation | Description |
|-------------|------|----------------|-------------|
| POST | /api/legacy/billing/invoices | CreateInvoice | Créer facture |
| GET | /api/legacy/billing/invoices/{id} | GetInvoice | Récupérer facture |
| GET | /api/legacy/billing/clients/{id}/invoices | GetInvoicesByClient | Liste factures client |
| PUT | /api/legacy/billing/invoices/{id}/pay | PayInvoice | Payer facture |

### REST v2 (passthrough)

| HTTP Method | Path | Description |
|-------------|------|-------------|
| POST | /api/v2/billing/auth/login | Login JWT |
| POST | /api/v2/billing/invoices | Créer facture |
| GET | /api/v2/billing/invoices/{id} | Récupérer facture |
| PUT | /api/v2/billing/invoices/{id}/pay | Payer facture |
| GET | /api/v2/billing/invoices/{id}/pdf | Générer PDF |
| DELETE | /api/v2/billing/invoices/{id} | Supprimer facture |

## 🚀 Déploiement

### Option 1 : Docker Compose (Recommandé)

```bash
# 1. Démarrer tous les services
cd wso2-integration
docker-compose up -d

# 2. Vérifier les services
docker-compose ps

# 3. Vérifier les logs
docker-compose logs -f wso2mi

# 4. Tester le gateway
curl http://localhost:8290/api/health
```

### Option 2 : Local (sans Docker)

```bash
# 1. Démarrer Billing Service
cd billing-service
mvn spring-boot:run
# → http://localhost:8081

# 2. Télécharger WSO2 MI
# https://wso2.com/integration/micro-integrator/

# 3. Copier les artefacts
cp -r wso2-integration/src/* <WSO2_MI>/repository/deployment/server/synapse-configs/default/

# 4. Démarrer WSO2 MI
cd <WSO2_MI>/bin
./micro-integrator.sh

# → http://localhost:8290
```

## 🧪 Tests complets

### Test 1 : SOAP Legacy via WSO2

```bash
# Créer une facture (JSON → SOAP → JSON)
curl -X POST http://localhost:8290/api/legacy/billing/invoices \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": 1001,
    "amount": 2500.00,
    "description": "Service via WSO2 Legacy"
  }'

# Réponse attendue
{
  "success": true,
  "message": "Invoice created successfully",
  "data": {
    "invoiceId": 1
  }
}

# Récupérer la facture
curl http://localhost:8290/api/legacy/billing/invoices/1

# Payer la facture
curl -X PUT http://localhost:8290/api/legacy/billing/invoices/1/pay \
  -H "Content-Type: application/json" \
  -d '{"paymentMethod": "TRANSFER"}'
```

### Test 2 : REST v2 via WSO2

```bash
# 1. Login
TOKEN=$(curl -X POST http://localhost:8290/api/v2/billing/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}' \
  | jq -r '.data.token')

echo "Token: $TOKEN"

# 2. Créer facture
curl -X POST http://localhost:8290/api/v2/billing/invoices \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": 1001,
    "amount": 3500.00,
    "description": "Service via WSO2 REST v2"
  }'

# 3. Récupérer facture
curl http://localhost:8290/api/v2/billing/invoices/1 \
  -H "Authorization: Bearer $TOKEN"

# 4. Télécharger PDF
curl http://localhost:8290/api/v2/billing/invoices/1/pdf \
  -H "Authorization: Bearer $TOKEN" \
  --output invoice.pdf
```

### Test 3 : Comparaison Legacy vs REST v2

```bash
# Legacy (pas d'auth)
time curl -X POST http://localhost:8290/api/legacy/billing/invoices \
  -H "Content-Type: application/json" \
  -d '{"clientId":1001,"amount":1000}'

# REST v2 (avec auth + JWT)
time curl -X POST http://localhost:8290/api/v2/billing/invoices \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"clientId":1001,"amount":1000}'
```

## 📊 Monitoring

### WSO2 MI Dashboard

```bash
# Accéder au dashboard
http://localhost:9164/dashboard

# Login par défaut
Username: admin
Password: admin
```

### Logs

```bash
# Docker
docker-compose logs -f wso2mi

# Local
tail -f <WSO2_MI>/repository/logs/wso2carbon.log
```

### Métriques

- **Requêtes/s** : Nombre de requêtes traitées
- **Latence** : Temps de réponse moyen
- **Erreurs** : Taux d'erreur
- **Conversions** : SOAP/JSON conversions

## 🔐 Sécurité

### Legacy SOAP

- ❌ Pas d'authentification (legacy)
- ✅ Validation des données
- ✅ Rate limiting (optionnel via WSO2)

### REST v2

- ✅ JWT requis
- ✅ Header Authorization passthrough
- ✅ Validation automatique par backend

### WSO2 Security (optionnel)

```xml
<!-- Ajouter OAuth2 sur l'API -->
<handlers>
    <handler class="org.wso2.carbon.apimgt.gateway.handlers.security.oauth.OAuthHandler"/>
</handlers>
```

## 🐛 Troubleshooting

### Erreur: Backend non accessible

```bash
# Vérifier que billing-service est démarré
curl http://localhost:8081/billing/health

# Vérifier les endpoints dans WSO2
# Logs WSO2 : "Connecting to backend..."
```

### Erreur: Conversion SOAP/JSON échoue

```bash
# Vérifier les logs WSO2
docker-compose logs wso2mi | grep "ERROR"

# Tester le backend SOAP directement
curl -X POST http://localhost:8081/billing/ws \
  -H "Content-Type: text/xml" \
  -d '<soap:Envelope>...</soap:Envelope>'
```

### Erreur: JWT invalide

```bash
# Le JWT est passthrough - vérifier backend REST
curl http://localhost:8081/billing/api/v2/invoices/1 \
  -H "Authorization: Bearer $TOKEN"
```

## 📝 Notes importantes

1. **Backends requis** : Billing Service doit être démarré AVANT WSO2
2. **Ports** : WSO2 (8290), Billing (8081), BRVM (5000)
3. **Conversion** : SOAP/XML ↔ JSON automatique pour legacy
4. **Passthrough** : REST v2 passe directement au backend
5. **Auth** : Legacy = aucune, REST v2 = JWT

---

**Prêt pour la production !** 🎉
