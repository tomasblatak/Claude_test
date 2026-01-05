# Pipedrive Setup Guide pro Stripe Integraci

## 1. Custom Fields - Nutné vytvořit v Pipedrive

### Person Custom Fields

Navigace: **Pipedrive → Settings → Data fields → Person**

| Field Name | Field Type | API Key | Popis |
|------------|-----------|---------|-------|
| Stripe Customer ID | Text | `stripe_customer_id` | ID zákazníka ze Stripe |
| Customer Source | Single option | `customer_source` | Možnosti: Stripe, Manual, Import |

### Organization Custom Fields

Navigace: **Pipedrive → Settings → Data fields → Organization**

| Field Name | Field Type | API Key | Popis |
|------------|-----------|---------|-------|
| Stripe Customer ID | Text | `org_stripe_customer_id` | ID zákazníka ze Stripe (pokud org = customer) |

### Product Custom Fields

Navigace: **Pipedrive → Settings → Data fields → Product**

| Field Name | Field Type | API Key | Popis |
|------------|-----------|---------|-------|
| Stripe Product ID | Text | `stripe_product_id` | ID produktu ze Stripe |
| Stripe Price ID | Text | `stripe_price_id` | ID ceny ze Stripe |
| Billing Frequency | Single option | `billing_frequency` | Možnosti: monthly, yearly, one-time |

### Deal Custom Fields

Navigace: **Pipedrive → Settings → Data fields → Deal**

| Field Name | Field Type | API Key | Popis |
|------------|-----------|---------|-------|
| Stripe Subscription ID | Text | `stripe_subscription_id` | ID subscription ze Stripe |
| Billing Frequency | Single option | `billing_frequency` | Možnosti: monthly, yearly |
| Recurring Revenue | Monetary | `recurring_revenue` | Měsíční opakující se příjem |
| Subscription Status | Single option | `subscription_status` | Možnosti: active, past_due, canceled, trialing |

---

## 2. Pipeline Setup pro Subscriptions

### Vytvoření Pipeline pro Subscriptions

Navigace: **Pipedrive → Settings → Pipelines**

**Název Pipeline:** `SaaS Subscriptions`

### Stages:

1. **Trial** (probability: 20%)
2. **Active** (probability: 90%)
3. **Payment Issue** (probability: 50%)
4. **Churned** (probability: 0%)
5. **Won** - Dlouhodobý zákazník (probability: 100%)

---

## 3. Nastavení Recurring Revenue

### Aktivace Revenue Forecast

Navigace: **Pipedrive → Settings → Features → Revenue forecast**

1. Zapnout "Revenue forecast"
2. Nastavit "Default billing frequency" = Monthly
3. Zapnout "Show recurring revenue in Insights"

### Nastavení produktu pro recurring:

Při vytváření produktu v n8n workflow:

```json
{
  "name": "Product Name",
  "prices": [
    {
      "price": 29.00,
      "currency": "USD"
    }
  ],
  "billing_frequency": "monthly",
  "billing_frequency_cycles": null
}
```

---

## 4. API Token Setup

### Získání Pipedrive API Tokenu:

1. Přihlaste se do Pipedrive
2. Jděte na **Settings** → **Personal preferences** → **API**
3. Zkopírujte "Your personal API token"
4. Uložte do n8n credentials:
   - Name: `Pipedrive Production`
   - API Token: [váš token]

### Testování API:

```bash
curl -X GET "https://api.pipedrive.com/v1/users/me?api_token=YOUR_TOKEN"
```

---

## 5. Stripe Webhook Setup

### Kroky v Stripe Dashboard:

1. Přihlaste se do Stripe Dashboard
2. Jděte na **Developers** → **Webhooks**
3. Klikněte na **Add endpoint**
4. URL: [URL z n8n Stripe Trigger node]
5. Vyberte events:
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_succeeded`
   - `invoice.payment_failed`

### Webhook Signing Secret:

- Zkopírujte "Signing secret" ze Stripe
- Uložte do n8n Stripe credentials pro ověření webhooků

---

## 6. Mapování Stripe → Pipedrive

### Customer → Person + Organization

| Stripe Field | Pipedrive Field |
|--------------|-----------------|
| `customer.name` | `person.name` |
| `customer.email` | `person.email` |
| `customer.phone` | `person.phone` |
| `customer.name` + " Company" | `organization.name` |
| `customer.id` | `person.custom.stripe_customer_id` |

### Subscription → Deal

| Stripe Field | Pipedrive Field |
|--------------|-----------------|
| `subscription.id` | `deal.custom.stripe_subscription_id` |
| `subscription.items[0].price.unit_amount / 100` | `deal.value` |
| `subscription.currency` | `deal.currency` |
| `subscription.status` | `deal.custom.subscription_status` |
| `subscription.items[0].price.recurring.interval` | `deal.custom.billing_frequency` |

### Product → Product

| Stripe Field | Pipedrive Field |
|--------------|-----------------|
| `product.name` | `product.name` |
| `product.id` | `product.custom.stripe_product_id` |
| `price.id` | `product.custom.stripe_price_id` |
| `price.unit_amount / 100` | `product.prices[0].price` |
| `price.currency` | `product.prices[0].currency` |

---

## 7. N8N Credentials Setup

### Stripe Credentials:

1. V n8n: **Credentials** → **New** → **Stripe API**
2. Vyplňte:
   - **Secret Key**: `sk_live_...` (z Stripe Dashboard → API keys)
   - Pro test mode: `sk_test_...`

### Pipedrive Credentials:

1. V n8n: **Credentials** → **New** → **Pipedrive API**
2. Vyplňte:
   - **API Token**: [váš token z Pipedrive]
   - **Domain**: např. `mycompany.pipedrive.com`

---

## 8. Testing Checklist

### Před spuštěním workflow:

- [ ] Vytvořeny všechny custom fields v Pipedrive
- [ ] Vytvořen pipeline "SaaS Subscriptions"
- [ ] Aktivován Revenue Forecast v Pipedrive
- [ ] Nastaven Stripe webhook endpoint
- [ ] Ověřen Pipedrive API token
- [ ] Ověřen Stripe API key
- [ ] Importován workflow do n8n
- [ ] Nakonfigurovány všechny credentials

### Test scenario:

1. **Vytvoření test subscription ve Stripe:**
   ```bash
   # Stripe CLI
   stripe customers create --email="test@example.com" --name="Test User"
   stripe subscriptions create \
     --customer="cus_xxx" \
     --items[0][price]="price_xxx"
   ```

2. **Ověření v Pipedrive:**
   - [ ] Person vytvořen s emailem `test@example.com`
   - [ ] Organization vytvořena
   - [ ] Product vytvořen
   - [ ] Deal vytvořen s hodnotou subscription
   - [ ] Product přidán k dealu s billing frequency = monthly
   - [ ] Recurring revenue viditelný v Insights

3. **Test existujícího zákazníka:**
   - Vytvořte Person manuálně v Pipedrive
   - Vytvořte subscription ve Stripe se stejným emailem
   - Ověřte, že se vytvoří jen nový deal (ne nový person)

---

## 9. Monitoring a Logging

### N8N Execution Log:

Sledujte v n8n:
- **Executions** → Zkontrolujte successful/failed runs
- Pokud failed, zkontrolujte error message

### Běžné chyby:

| Error | Řešení |
|-------|--------|
| "Person not found" | Zkontrolujte email matching v Search node |
| "Invalid API token" | Ověřte Pipedrive credentials |
| "Product already exists" | Přidejte check nebo použijte "Get or Create" pattern |
| "Currency not supported" | Ověřte, že Pipedrive podporuje měnu ze Stripe |

---

## 10. Pipedrive Revenue Insights

### Zobrazení Recurring Revenue:

Po správném nastavení uvidíte v Pipedrive:

**Insights → Revenue forecast:**
- Monthly Recurring Revenue (MRR)
- Annual Recurring Revenue (ARR)
- Revenue forecast pro příštích 12 měsíců
- Churn rate

### Filtry pro reporting:

- Filtrovat dealy podle `billing_frequency` = monthly
- Filtrovat podle `subscription_status` = active
- Seskupit podle produktu

---

## 11. Rozšíření - Další Stripe Events

### Subscription Updated:

Webhook event: `customer.subscription.updated`

**Akce:**
- Aktualizovat deal value
- Změnit subscription status
- Přidat note s historií změn

### Subscription Canceled:

Webhook event: `customer.subscription.deleted`

**Akce:**
- Změnit deal stage na "Churned"
- Aktualizovat `subscription_status` = canceled
- Přidat note s důvodem (pokud dostupný)

### Payment Failed:

Webhook event: `invoice.payment_failed`

**Akce:**
- Změnit deal stage na "Payment Issue"
- Vytvořit activity (task) pro follow-up
- Poslat notifikaci sales teamu

---

## 12. Best Practices

### Deduplikace:

Před vytvořením nového objektu vždy zkontrolujte:
- Person: podle `email` nebo `stripe_customer_id`
- Product: podle `stripe_product_id`
- Deal: podle `stripe_subscription_id`

### Error Handling:

```
Try/Catch pattern v n8n:
- Hlavní workflow
  └─ Error Trigger
      └─ Send notification (Slack/Email)
      └─ Log to database
```

### Rate Limits:

Pipedrive API limity:
- 100 requests per 10 seconds
- Použijte "Wait" node pokud batch processing

### Security:

- Nikdy nezobrazujte API keys v UI
- Používejte webhook signing verification
- Logujte všechny operace pro audit

---

## 13. Support a Troubleshooting

### Užitečné odkazy:

- [Pipedrive API Docs](https://developers.pipedrive.com/docs/api/v1)
- [Stripe API Docs](https://stripe.com/docs/api)
- [N8N Pipedrive Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.pipedrive/)
- [N8N Stripe Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.stripe/)

### Debug tips:

1. Použijte n8n "Execute Node" pro testování jednotlivých kroků
2. Zkontrolujte raw JSON output každého node
3. Ověřte field names v Pipedrive API
4. Testujte s Stripe test mode před production

