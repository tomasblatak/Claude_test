# 🚀 Quick Start Guide - Stripe → Pipedrive Sync

## Rychlý přehled

Tento workflow automaticky synchronizuje zákazníky ze Stripe do Pipedrive a vytváří recurring revenue produkty.

---

## ⚡ Rychlé kroky k nasazení

### 1️⃣ Připravte Pipedrive (15 minut)

#### Vytvořte Custom Fields:

**Person fields:**
```
Stripe Customer ID → Text field
```

**Deal fields:**
```
Stripe Subscription ID → Text field
Billing Frequency → Dropdown (monthly, yearly)
Subscription Status → Dropdown (active, canceled, past_due)
```

**Product fields:**
```
Stripe Product ID → Text field
Stripe Price ID → Text field
```

#### Vytvořte Pipeline:
- Název: "SaaS Subscriptions"
- Stages: Trial → Active → Payment Issue → Churned → Won

#### Získejte API Token:
- Settings → Personal Preferences → API → Zkopírujte token

---

### 2️⃣ Připravte Stripe (5 minut)

#### Získejte API Key:
- Stripe Dashboard → Developers → API Keys
- Zkopírujte "Secret key" (začíná `sk_test_` nebo `sk_live_`)

#### Připravte webhook endpoint:
- **Zatím nenastavujte** - URL dostanete až z n8n

---

### 3️⃣ Importujte workflow do n8n (5 minut)

#### Import:
1. Otevřete n8n
2. Klikněte **Import from File**
3. Vyberte soubor: `n8n-workflow-template.json`
4. Workflow se importuje

#### Nastavte Credentials:

**Stripe:**
- Klikněte na Stripe node
- Add credential
- Vložte Secret Key ze Stripe

**Pipedrive:**
- Klikněte na Pipedrive node
- Add credential
- Vložte API Token z Pipedrive

---

### 4️⃣ Aktivujte Webhook ve Stripe (5 minut)

#### Získejte webhook URL z n8n:
1. V n8n otevřete "Stripe Webhook Trigger" node
2. Zkopírujte "Webhook URL" (pod Execute Workflow)

#### Nastavte ve Stripe:
1. Stripe Dashboard → Developers → Webhooks
2. Add endpoint
3. URL: [Vložte URL z n8n]
4. Select events:
   - ✅ `customer.subscription.created`
   - ✅ `customer.subscription.updated`
   - ✅ `customer.subscription.deleted`
5. Add endpoint

---

### 5️⃣ Test! (5 minut)

#### Vytvoření test subscription:

**Ve Stripe Dashboard:**
1. Customers → Add customer
   - Email: `test@example.com`
   - Name: `Test User`
2. Subscriptions → Add subscription
   - Vyberte customer
   - Vyberte product a price
   - Start subscription

**Nebo pomocí Stripe CLI:**
```bash
stripe customers create --email="test@example.com" --name="Test User"
stripe subscriptions create --customer=cus_xxx --items[0][price]=price_xxx
```

#### Ověření v Pipedrive:

Zkontrolujte že se vytvořilo:
- ✅ Person: Test User (email: test@example.com)
- ✅ Organization: Test User Company
- ✅ Product: [název z vašeho Stripe produktu]
- ✅ Deal: Test User - [Product]
  - Value: [cena z subscription]
  - Product attached s billing frequency = monthly

---

## 📊 Co workflow dělá

### Když přijde nový Stripe subscription:

```
1. Webhook ze Stripe → n8n
   ↓
2. Získá customer data ze Stripe
   ↓
3. Vyhledá Person v Pipedrive (podle emailu)
   ↓
4a. Pokud NEEXISTUJE:          4b. Pokud EXISTUJE:
    → Vytvoří Organization          → Použije existující Person
    → Vytvoří Person                ↓
    → Vytvoří Product          5. Vytvoří nový Deal
    ↓                               ↓
5. Vytvoří Deal                6. Přidá Product k Deal
   ↓                               (s recurring billing)
6. Přidá Product k Deal
   (s recurring billing = monthly)
```

---

## 🎯 Klíčové funkce

### ✅ Automatické vytváření:
- Person (zákazník)
- Organization (firma)
- Product (z Stripe produktu)
- Deal (obchod s recurring revenue)

### ✅ Recurring Revenue:
- Product připojen k dealu s `billing_frequency = monthly`
- Zobrazuje se v Pipedrive Revenue Forecast
- MRR (Monthly Recurring Revenue) tracking

### ✅ Deduplikace:
- Kontrola existujícího Person podle emailu
- Pokud existuje, vytvoří jen nový Deal

---

## 🔧 Customizace

### Změna billing frequency:

V node "Add Product to Deal" změňte:
```json
"billingFrequency": "yearly"  // místo "monthly"
"durationUnit": "year"         // místo "month"
```

### Přidání custom fields:

V node "Create Person" přidejte:
```json
"customProperties": {
  "property": [
    {
      "name": "nazev_custom_fieldu",
      "value": "{{ hodnota }}"
    }
  ]
}
```

### Změna Pipeline/Stage:

V node "Create Deal" přidejte:
```json
"pipelineId": 123,  // ID vašeho pipeline
"stageId": 456      // ID konkrétního stage
```

**Jak získat IDs:**
```bash
curl "https://api.pipedrive.com/v1/pipelines?api_token=YOUR_TOKEN"
```

---

## 🐛 Troubleshooting

### ❌ Workflow se nespustil po vytvoření subscription

**Možné příčiny:**
1. Webhook není správně nastaven ve Stripe
2. Webhook URL není aktivní (aktivujte workflow v n8n)
3. Event typ není vybraný ve Stripe webhook

**Řešení:**
- Zkontrolujte Stripe → Webhooks → Your endpoint → Recent deliveries
- Měli byste vidět úspěšné doručení (200 OK)

---

### ❌ Person se nevytváří v Pipedrive

**Možné příčiny:**
1. Nesprávný Pipedrive API token
2. Chybějící povinná pole
3. Email formát není validní

**Debug:**
- V n8n klikněte na "Create Person" node
- Zkontrolujte "Output" tab
- Pokud chyba, uvidíte error message od Pipedrive

---

### ❌ Recurring revenue se nezobrazuje

**Možné příčiny:**
1. Revenue Forecast není aktivován v Pipedrive
2. Billing frequency není správně nastavena
3. Product není připojen k dealu

**Řešení:**
1. Pipedrive → Settings → Features → zapněte "Revenue forecast"
2. V dealu zkontrolujte Products tab
3. Ověřte že product má "Billing frequency = Monthly"

---

### ❌ Vytváří duplicitní Person

**Možné příčiny:**
1. Email se neshoduje (case sensitive)
2. Search node nenašel existující person

**Řešení:**
- V "Search Person in Pipedrive" node zapněte "Exact Match"
- Normalizujte email: `{{ $json.email.toLowerCase() }}`

---

## 📈 Sledování výsledků

### N8N Dashboard:
- Executions → Sledujte successful/failed runs
- Každá subscription by měla = 1 successful execution

### Pipedrive Insights:
- Insights → Revenue Forecast
- Měli byste vidět:
  - **MRR (Monthly Recurring Revenue)**
  - **ARR (Annual Recurring Revenue)**
  - Revenue forecast graf

### Stripe Dashboard:
- Webhooks → Your endpoint → Recent deliveries
- Všechny delivery by měly být zelené (2xx status)

---

## 🚀 Další kroky (optional)

### 1. Přidejte subscription updates:

Webhook event: `customer.subscription.updated`

**Co dělat:**
- Aktualizovat hodnotu dealu
- Změnit subscription status field

### 2. Přidejte cancelation handling:

Webhook event: `customer.subscription.deleted`

**Co dělat:**
- Změnit deal stage na "Churned"
- Update status field = "canceled"

### 3. Přidejte payment failure handling:

Webhook event: `invoice.payment_failed`

**Co dělat:**
- Změnit stage na "Payment Issue"
- Vytvořit Activity/Task pro follow-up
- Poslat notifikaci

### 4. Přidejte error notifications:

V n8n:
- Přidejte "Error Trigger" node
- Připojte na Slack/Email node
- Dostanete alert při každé chybě

---

## 📚 Další dokumentace

- **Detailní návod:** `n8n-stripe-pipedrive-workflow.md`
- **Setup guide:** `pipedrive-setup-guide.md`
- **Workflow template:** `n8n-workflow-template.json`

---

## ❓ Časté dotazy

**Q: Kolik to stojí?**
- N8N: Free pro self-hosted, nebo od $20/měsíc cloud
- Stripe: Standardní poplatky (2.9% + 30¢)
- Pipedrive: Standardní subscription

**Q: Jak rychle to běží?**
- Webhook delivery: < 1 sekunda
- Workflow execution: 2-5 sekund
- Celkem: zákazník viditelný v Pipedrive do 10 sekund

**Q: Můžu to použít s jinými payment procesory?**
- Ano! Jen změňte trigger node (např. PayPal, Paddle)
- Logika zůstává stejná

**Q: Co když zmažu person v Pipedrive?**
- Příští Stripe event vytvoří nový person
- Doporučuji: místo mazání použijte archivaci

**Q: Synchronizuje se to obousměrně?**
- Ne, zatím jen Stripe → Pipedrive
- Pro obousměrnou sync potřebujete další workflow

---

## 💡 Tips & Tricks

### Performance:
- Pro velké množství subscriptions použijte queue node
- Batch processing pro rate limit handling

### Security:
- Používejte webhook signing verification
- Rotujte API keys pravidelně
- Nelogujte sensitive data

### Monitoring:
- Nastavte alerting pro failed executions
- Sledujte execution time trends
- Pravidelně kontrolujte duplicate records

---

**Potřebujete pomoc?**
- N8N Community: https://community.n8n.io
- Pipedrive Support: https://support.pipedrive.com
- Stripe Support: https://support.stripe.com

---

✅ **Máte vše nastaveno? Gratulujeme!** 🎉

Vaše Stripe subscriptions se teď automaticky synchronizují do Pipedrive s full recurring revenue tracking.
