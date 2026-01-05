# 🔄 Stripe → Pipedrive Synchronizace s Recurring Revenue

Kompletní n8n workflow pro automatickou synchronizaci zákazníků ze Stripe do Pipedrive včetně trackingu měsíčních recurring revenues.

---

## 📋 Co tento projekt obsahuje

### 📄 Dokumentace

1. **[QUICK-START.md](./QUICK-START.md)** - ⚡ Rychlý start za 30 minut
   - Krok-za-krokem setup guide
   - Test scenario
   - Troubleshooting

2. **[n8n-stripe-pipedrive-workflow.md](./n8n-stripe-pipedrive-workflow.md)** - 📚 Detailní dokumentace
   - Kompletní popis workflow
   - Vysvětlení každého node
   - Schéma a logika
   - Best practices

3. **[pipedrive-setup-guide.md](./pipedrive-setup-guide.md)** - ⚙️ Pipedrive konfigurace
   - Custom fields setup
   - Pipeline setup
   - API konfigurace
   - Revenue forecast nastavení
   - Rozšířené funkce

### 📦 Workflow Template

4. **[n8n-workflow-template.json](./n8n-workflow-template.json)** - 🔧 N8N workflow
   - Import-ready JSON
   - Připravené nodes
   - Předkonfigurovaná logika
   - Stačí přidat credentials

---

## 🎯 Hlavní funkce

### ✅ Co workflow dělá:

- **Automaticky vytváří záznamy** při nové Stripe subscription:
  - 👤 Person (zákazník)
  - 🏢 Organization (firma)
  - 📦 Product (na základě Stripe produktu)
  - 💼 Deal (obchod s recurring revenue)

- **Recurring Revenue Tracking:**
  - 💰 Měsíční recurring revenue (MRR)
  - 📊 Roční recurring revenue (ARR)
  - 📈 Revenue forecast v Pipedrive Insights

- **Inteligentní deduplikace:**
  - 🔍 Kontrola existujících zákazníků
  - ➕ Vytvoření nového záznamu jen pokud neexistuje
  - 🔄 Pro existující zákazníky přidá jen nový deal

---

## 🚀 Rychlý start

### Prerekvizity:
- ✅ Stripe account (test nebo live mode)
- ✅ Pipedrive account
- ✅ N8N instance (self-hosted nebo cloud)

### Za 3 kroky k funkčnímu workflow:

#### 1️⃣ Importujte workflow
```bash
# V n8n: Import from File → vyberte n8n-workflow-template.json
```

#### 2️⃣ Nastavte credentials
- Stripe API Key (ze Stripe Dashboard)
- Pipedrive API Token (z Pipedrive Settings)

#### 3️⃣ Aktivujte webhook
- Zkopírujte webhook URL z n8n
- Nastavte ve Stripe Dashboard → Webhooks
- Vyberte event: `customer.subscription.created`

**🎉 Hotovo!** Vytvořte test subscription a sledujte magii.

---

## 📊 Workflow schéma

```
Stripe Webhook
     ↓
Get Customer & Product Data
     ↓
Search Person in Pipedrive
     ↓
   [IF]
     ├─ Person Neexistuje:              ├─ Person Existuje:
     │  1. Create Organization          │  1. Use Existing Person
     │  2. Create Person                │     ↓
     │  3. Create Product               │  2. Create Deal
     │     ↓                             │     ↓
     │  4. Create Deal ←─────────────────┘  3. Add Product
     │     ↓
     │  5. Add Product to Deal
     │     (with monthly recurring)
     ↓
   Done!
```

---

## 💡 Příklady použití

### Use Case 1: Nový SaaS zákazník
**Scénář:** Uživatel si zakoupí $29/měsíc subscription přes Stripe

**Co se stane:**
1. ✅ Vytvoří se Person "John Doe"
2. ✅ Vytvoří se Organization "John Doe Company"
3. ✅ Vytvoří se Product "Premium Plan - $29"
4. ✅ Vytvoří se Deal "$29/month" s recurring billing
5. ✅ Pipedrive Insights ukáže +$29 MRR

### Use Case 2: Existující zákazník, nový produkt
**Scénář:** Zákazník už existuje v Pipedrive, kupuje další subscription

**Co se stane:**
1. ✅ Workflow najde existující Person
2. ✅ Vytvoří nový Deal pro novou subscription
3. ✅ MRR se zvýší o hodnotu nové subscription

### Use Case 3: Upgrade subscription
**Scénář:** Zákazník upgraduje z $29 na $99 plánu

**S rozšířením workflow (subscription.updated event):**
1. ✅ Aktualizuje hodnotu existujícího dealu
2. ✅ MRR se automaticky přepočítá

---

## 🔧 Konfigurace

### Povinná nastavení:

#### Pipedrive Custom Fields:
Vytvořte tyto custom fields (návod v [pipedrive-setup-guide.md](./pipedrive-setup-guide.md)):

**Person:**
- `stripe_customer_id` (Text)

**Deal:**
- `stripe_subscription_id` (Text)
- `billing_frequency` (Dropdown: monthly/yearly)
- `subscription_status` (Dropdown: active/canceled/past_due)

**Product:**
- `stripe_product_id` (Text)
- `stripe_price_id` (Text)

#### Pipedrive Pipeline:
- Název: "SaaS Subscriptions"
- Stages: Trial → Active → Payment Issue → Churned → Won

#### Revenue Forecast:
- Aktivujte v Pipedrive → Settings → Features → Revenue forecast

---

## 📈 Výsledky a monitoring

### V Pipedrive uvidíte:

**Insights → Revenue Forecast:**
- 💰 **MRR (Monthly Recurring Revenue)** - celkový měsíční příjem
- 📊 **ARR (Annual Recurring Revenue)** - roční projekce
- 📈 **Revenue forecast** - predikce na příštích 12 měsíců
- 📉 **Churn rate** - míra odchodů zákazníků

**Deals:**
- Každá subscription = 1 deal
- Products připojené s monthly billing
- Automatic revenue calculation

---

## 🐛 Troubleshooting

| Problém | Řešení |
|---------|--------|
| Workflow se nespustil | Zkontrolujte Stripe webhook delivery (Stripe Dashboard → Webhooks → Recent deliveries) |
| Person se duplikuje | Zapněte "Exact Match" v Search node, normalizujte email `.toLowerCase()` |
| Recurring revenue se nezobrazuje | Aktivujte Revenue Forecast v Pipedrive Settings → Features |
| "Invalid API token" | Ověřte credentials v n8n, zkuste vygenerovat nový token |

**Detailní troubleshooting:** [QUICK-START.md](./QUICK-START.md#-troubleshooting)

---

## 🚀 Rozšíření

### Další Stripe events k implementaci:

1. **subscription.updated** - Aktualizace ceny/plánu
2. **subscription.deleted** - Churn tracking
3. **invoice.payment_failed** - Payment issues
4. **invoice.payment_succeeded** - Payment confirmations

### Další integrace:

- **Slack notifications** - Alerting pro nové subscriptions
- **Email automation** - Onboarding emails
- **Google Sheets** - Reporting export
- **Webhooks** - Notifikace do vlastních systémů

---

## 📚 Dokumentace a zdroje

### Tento projekt:
- [Quick Start Guide](./QUICK-START.md) - Začněte zde
- [Workflow dokumentace](./n8n-stripe-pipedrive-workflow.md) - Detaily
- [Pipedrive Setup](./pipedrive-setup-guide.md) - Konfigurace

### Externí dokumentace:
- [N8N Docs](https://docs.n8n.io/)
- [Pipedrive API](https://developers.pipedrive.com/docs/api/v1)
- [Stripe API](https://stripe.com/docs/api)
- [Stripe Webhooks](https://stripe.com/docs/webhooks)

---

## 💬 Support

### Potřebujete pomoc?

1. **Dokumentace** - Zkontrolujte [QUICK-START.md](./QUICK-START.md) a [troubleshooting sekci](./QUICK-START.md#-troubleshooting)
2. **N8N Community** - [community.n8n.io](https://community.n8n.io)
3. **Pipedrive Support** - [support.pipedrive.com](https://support.pipedrive.com)
4. **Stripe Support** - [support.stripe.com](https://support.stripe.com)

---

## 📝 Changelog

### Version 1.0 (2026-01-05)
- ✅ Initial workflow release
- ✅ Stripe subscription sync
- ✅ Automatic Person/Organization creation
- ✅ Product creation with recurring billing
- ✅ Monthly recurring revenue tracking
- ✅ Duplicate person detection

### Plánované featury:
- [ ] Subscription update handling
- [ ] Cancellation workflow
- [ ] Payment failure notifications
- [ ] Multi-currency support
- [ ] Custom field mapping configurator

---

## 📄 Licence

Tento workflow je poskytován "as is" pro vaše vlastní použití a modifikaci.

---

## ⭐ Contributing

Máte vylepšení? Pull requesty jsou vítány!

1. Fork repository
2. Vytvořte feature branch
3. Commitněte změny
4. Otevřete Pull Request

---

**Vytvořeno s ❤️ pro automatizaci SaaS businessu**

---

## 🎓 Use Cases a příklady

### Pro SaaS startupy:
- Automatické onboarding nových zákazníků
- Real-time MRR tracking
- Sales pipeline pro subscriptions

### Pro agentury:
- Tracking klientských subscriptions
- Recurring revenue forecasting
- Automatic billing management

### Pro e-commerce s subscriptions:
- Subscription box tracking
- Member management
- Recurring order automation

---

**🚀 Začněte nyní:** [QUICK-START.md](./QUICK-START.md)
