# N8N Workflow: Stripe → Pipedrive Synchronizace

## Přehled
Automatická synchronizace zákazníků ze Stripe do Pipedrive s vytvořením Person, Company a produktů s recurring revenue.

## Workflow struktura

### 1. Trigger Node: Stripe Webhook
**Node Type:** `Stripe Trigger`
**Konfigurace:**
- **Event:** `customer.subscription.created` nebo `checkout.session.completed`
- **Credentials:** Stripe API credentials
- **Webhook endpoint:** Automaticky vygenerován n8n

**Výstupní data:**
```json
{
  "customer": {
    "id": "cus_xxx",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "subscription": {
    "id": "sub_xxx",
    "items": [{
      "price": {
        "id": "price_xxx",
        "product": "prod_xxx",
        "unit_amount": 2900,
        "recurring": {
          "interval": "month"
        }
      }
    }]
  }
}
```

---

### 2. Získání detailů ze Stripe
**Node Type:** `Stripe` (Action Node)
**Konfigurace:**
- **Resource:** Customer
- **Operation:** Get
- **Customer ID:** `{{ $json.customer.id }}`

**A také druhý Stripe node pro Product:**
**Node Type:** `Stripe` (Action Node)
- **Resource:** Product
- **Operation:** Get
- **Product ID:** `{{ $json.subscription.items[0].price.product }}`

---

### 3. Check - Existuje zákazník v Pipedrive?
**Node Type:** `Pipedrive` (Search Node)
**Konfigurace:**
- **Resource:** Persons
- **Operation:** Search
- **Search By:** Email
- **Email:** `{{ $json.customer.email }}`

---

### 4. IF Node - Rozhodovací logika
**Node Type:** `IF`
**Konfigurace:**
- **Condition:** `{{ $json.data.length > 0 }}`
- **True Branch:** Osoba existuje → Aktualizuj záznam
- **False Branch:** Osoba neexistuje → Vytvoř nový záznam

---

## FALSE BRANCH - Vytvoření nového záznamu

### 5a. Vytvoření Organization (Company) v Pipedrive
**Node Type:** `Pipedrive`
**Konfigurace:**
- **Resource:** Organizations
- **Operation:** Create
- **Fields:**
  - **Name:** `{{ $json.customer.name || $json.customer.email.split('@')[0] + ' Company' }}`
  - **Owner ID:** [Váš Owner ID v Pipedrive]

**Výstup:** `organization.id`

---

### 5b. Vytvoření Person v Pipedrive
**Node Type:** `Pipedrive`
**Konfigurace:**
- **Resource:** Persons
- **Operation:** Create
- **Fields:**
  - **Name:** `{{ $json.customer.name }}`
  - **Email:** `{{ $json.customer.email }}`
  - **Organization ID:** `{{ $node["Vytvoření Organization"].json.data.id }}`
  - **Owner ID:** [Váš Owner ID]
  - **Custom Fields:**
    - `stripe_customer_id`: `{{ $json.customer.id }}`

**Výstup:** `person.id`

---

### 5c. Vytvoření nebo získání Produktu v Pipedrive
**Node Type:** `Pipedrive`
**Konfigurace:**
- **Resource:** Products
- **Operation:** Create
- **Fields:**
  - **Name:** `{{ $node["Get Stripe Product"].json.name }}`
  - **Code:** `{{ $node["Get Stripe Product"].json.id }}`
  - **Unit:** `subscription`
  - **Price:** `{{ $json.subscription.items[0].price.unit_amount / 100 }}`
  - **Currency:** `{{ $json.subscription.currency.toUpperCase() }}`
  - **Visible To:** Owner & followers
  - **Custom Fields:**
    - `stripe_price_id`: `{{ $json.subscription.items[0].price.id }}`
    - `stripe_product_id`: `{{ $node["Get Stripe Product"].json.id }}`

---

### 5d. Vytvoření Deal s Recurring Revenue
**Node Type:** `Pipedrive`
**Konfigurace:**
- **Resource:** Deals
- **Operation:** Create
- **Fields:**
  - **Title:** `{{ $json.customer.name }} - Subscription`
  - **Person ID:** `{{ $node["Vytvoření Person"].json.data.id }}`
  - **Organization ID:** `{{ $node["Vytvoření Organization"].json.data.id }}`
  - **Value:** `{{ $json.subscription.items[0].price.unit_amount / 100 }}`
  - **Currency:** `{{ $json.subscription.currency.toUpperCase() }}`
  - **Status:** `open`
  - **Stage ID:** [ID prvního stage vašeho pipeline]
  - **Custom Fields (DŮLEŽITÉ pro recurring revenue):**
    - `billing_frequency`: `monthly`
    - `billing_frequency_cycles`: `null` (nekonečné)
    - `recurring_revenue`: `{{ $json.subscription.items[0].price.unit_amount / 100 }}`
    - `stripe_subscription_id`: `{{ $json.subscription.id }}`

**Výstup:** `deal.id`

---

### 5e. Přidání produktu k Deal
**Node Type:** `Pipedrive`
**Konfigurace:**
- **Resource:** Deal Products
- **Operation:** Add
- **Fields:**
  - **Deal ID:** `{{ $node["Vytvoření Deal"].json.data.id }}`
  - **Product ID:** `{{ $node["Vytvoření Produktu"].json.data.id }}`
  - **Quantity:** `1`
  - **Item Price:** `{{ $json.subscription.items[0].price.unit_amount / 100 }}`
  - **Duration:** `1`
  - **Duration Unit:** `month`
  - **Billing Frequency:** `monthly`
  - **Billing Frequency Cycles:** `null` (pro nekonečné opakování)

---

## TRUE BRANCH - Aktualizace existujícího záznamu

### 6a. Aktualizace Person
**Node Type:** `Pipedrive`
**Konfigurace:**
- **Resource:** Persons
- **Operation:** Update
- **Person ID:** `{{ $json.data[0].id }}`
- **Fields:**
  - Aktualizovat pouze potřebná pole

### 6b. Vytvoření nového Deal pro existujícího zákazníka
Stejně jako krok 5d výše

---

## Nastavení Recurring Revenue v Pipedrive

### Důležité poznámky:

1. **Pipedrive Revenue Forecast:**
   - Pro správné zobrazení jako "recurring revenue" musíte v Pipedrive nastavit:
   - Deal → Products → nastavit "Billing frequency" = Monthly
   - "Duration" nastavit podle předplatného

2. **Custom Fields v Pipedrive:**
   Vytvořte tyto custom fields v Pipedrive (pokud ještě neexistují):
   - `stripe_customer_id` (text)
   - `stripe_subscription_id` (text)
   - `stripe_price_id` (text)
   - `recurring_revenue` (monetary)
   - `billing_frequency` (enum: monthly, yearly)

3. **Pipedrive API pro Recurring:**
   ```json
   {
     "products_attached_to": [
       {
         "product_id": 123,
         "item_price": 29.00,
         "quantity": 1,
         "billing_frequency": "monthly",
         "billing_frequency_cycles": null
       }
     ]
   }
   ```

---

## Vizuální schéma workflow

```
┌─────────────────────┐
│ Stripe Webhook      │
│ (New Subscription)  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Get Customer Data   │
│ Get Product Data    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Search Person       │
│ in Pipedrive        │
└──────────┬──────────┘
           │
           ▼
      ┌────────┐
      │   IF   │
      └───┬─┬──┘
          │ │
    FALSE │ │ TRUE
          │ │
          │ └──────────────┐
          │                │
          ▼                ▼
┌──────────────────┐  ┌──────────────┐
│ Create Company   │  │ Update       │
└────────┬─────────┘  │ Person       │
         │            └──────┬───────┘
         ▼                   │
┌──────────────────┐         │
│ Create Person    │         │
└────────┬─────────┘         │
         │                   │
         ▼                   │
┌──────────────────┐         │
│ Create Product   │         │
└────────┬─────────┘         │
         │                   │
         └────────┬──────────┘
                  │
                  ▼
         ┌──────────────────┐
         │ Create Deal      │
         │ (with recurring) │
         └────────┬─────────┘
                  │
                  ▼
         ┌──────────────────┐
         │ Add Product      │
         │ to Deal          │
         └──────────────────┘
```

---

## Testování workflow

### Testovací kroky:
1. Vytvořte test subscription ve Stripe test mode
2. Ověřte, že webhook byl doručen do n8n
3. Zkontrolujte v Pipedrive:
   - Vytvořen nový Person
   - Vytvořena Organization
   - Vytvořen Product
   - Vytvořen Deal s produktem
   - Revenue forecast zobrazuje měsíční recurring revenue

### Stripe Test Webhook:
```bash
stripe trigger customer.subscription.created
```

---

## Error Handling

Přidejte Error Trigger node:
- **Node Type:** `Error Trigger`
- **Connected to:** Slack/Email notification
- **Message:** Odešle detaily chyby pro debugging

---

## Optimalizace a Best Practices

1. **Deduplikace:** Kontrola `stripe_subscription_id` před vytvořením nového deal
2. **Idempotence:** Použití Stripe subscription ID jako unique identifier
3. **Logging:** Přidat node pro logování všech operací
4. **Retry Logic:** Nastavit retry v n8n settings pro failed executions
5. **Rate Limiting:** Respektovat API limity Pipedrive (100 requests/10s)

---

## Potřebné Credentials v n8n

1. **Stripe API Key:**
   - Získat z Stripe Dashboard → Developers → API Keys
   - Použít Restricted Key s minimálními oprávněními

2. **Pipedrive API Token:**
   - Získat z Pipedrive → Settings → API
   - Nebo použít OAuth2

---

## Rozšíření workflow (optional)

1. **Webhook pro subscription.updated:** Aktualizace ceny při změně
2. **Webhook pro subscription.deleted:** Označení deal jako Lost
3. **Webhook pro invoice.payment_failed:** Poznámka v Pipedrive
4. **Synchronizace zpět:** Pipedrive změny → Stripe metadata

