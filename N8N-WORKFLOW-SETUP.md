# 🔄 N8N Workflow: Google Sheets → Leadspeaker.com

Automatický workflow pro nahrávání kontaktů z Google Sheets do platformy Leadspeaker.com.

## 📋 Co tento workflow dělá?

1. **Monitoruje Google Sheets** - Sleduje nové řádky v zadaném Google Sheets dokumentu
2. **Zpracuje data** - Mapuje sloupce z tabulky na pole v Leadspeaker
3. **Nahraje kontakt** - Automaticky vytvoří nový kontakt v Leadspeaker.com
4. **Loguje výsledky** - Zaznamenává úspěšné i neúspěšné operace

## 🚀 Instalace do N8N

### Krok 1: Import workflow

1. Otevřete N8N
2. Klikněte na **"Workflows"** → **"Add Workflow"** → **"Import from File"**
3. Vyberte soubor `google-sheets-to-leadspeaker.json`
4. Workflow se automaticky načte

### Krok 2: Konfigurace Google Sheets

#### A) Připojte Google účet:

1. Klikněte na node **"Google Sheets Trigger"**
2. V sekci **"Credential to connect with"** klikněte na **"Create New"**
3. Postupujte podle návodu pro autorizaci Google účtu
4. Uložte credentials

#### B) Vyberte spreadsheet:

1. V poli **"Document"** vyberte váš Google Sheets dokument
2. V poli **"Sheet"** vyberte konkrétní list (záložku)
3. Nastavte **"Trigger On"** na **"Row Added"** (nový řádek)
4. Polling interval: **Každou minutu** (nebo dle potřeby)

#### C) Struktura tabulky:

Ujistěte se, že vaše Google Sheets má tyto sloupce (můžete použít české nebo anglické názvy):

| Email | First Name / Jméno | Last Name / Příjmení | Phone / Telefon | Company / Společnost |
|-------|-------------------|---------------------|----------------|---------------------|
| email@example.com | Jan | Novák | +420 123 456 789 | Firma s.r.o. |

**Důležité:** Sloupec **Email** je povinný!

### Krok 3: Konfigurace Leadspeaker API

#### A) Získejte API klíč:

1. Přihlaste se do [Leadspeaker.com](https://leadspeaker.com)
2. Jděte do **Nastavení** → **API Keys**
3. Vytvořte nový API klíč a zkopírujte ho

#### B) Nastavte credentials v N8N:

1. Klikněte na node **"Leadspeaker API"**
2. V sekci **"Credential to connect with"** klikněte na **"Create New"**
3. Vyberte **"Header Auth"**
4. Nastavte:
   - **Name:** `Authorization`
   - **Value:** `Bearer VÁŠ_API_KLÍČ`
   (nebo podle dokumentace Leadspeaker - může být také `X-API-Key: VÁŠ_API_KLÍČ`)

#### C) Ověřte API endpoint:

Zkontrolujte v dokumentaci Leadspeaker.com správný endpoint pro vytváření kontaktů. Ve workflow je nastaveno:
```
https://api.leadspeaker.com/v1/contacts
```

**Pokud má Leadspeaker jiný endpoint, upravte URL v node "Leadspeaker API".**

### Krok 4: Testování

1. **Aktivujte workflow** - Klikněte na tlačítko **"Active"** v pravém horním rohu
2. **Přidejte testovací řádek** do Google Sheets:
   ```
   Email: test@example.com
   Jméno: Test
   Příjmení: Testovací
   Telefon: +420 123 456 789
   Společnost: Test s.r.o.
   ```
3. **Počkejte cca 1 minutu** (podle polling intervalu)
4. **Zkontrolujte Executions** v N8N - měl by se objevit nový záznam
5. **Ověřte v Leadspeaker.com** - kontakt by měl být vytvořen

## 🔧 Přizpůsobení workflow

### Změna mapování polí

Pokud má vaše tabulka jiné názvy sloupců, upravte node **"Mapování polí"**:

```javascript
// Příklad: Pokud máte sloupec "E-mail" místo "Email"
email: ={{ $json['E-mail'] }}

// Příklad: Pokud máte sloupec "Celé jméno"
firstName: ={{ $json['Celé jméno'].split(' ')[0] }}
lastName: ={{ $json['Celé jméno'].split(' ')[1] }}
```

### Přidání dalších polí

Můžete mapovat další pole, která Leadspeaker podporuje:

1. V node **"Mapování polí"** přidejte nové assignment
2. V node **"Leadspeaker API"** přidejte nový body parameter

Příklady dalších polí:
- `job_title` - Pozice
- `website` - Webová stránka
- `notes` - Poznámky
- `tags` - Štítky

### Změna frekvence kontroly

Defaultně workflow kontroluje nové řádky **každou minutu**. Pro změnu:

1. Otevřete node **"Google Sheets Trigger"**
2. V sekci **"Poll Times"** změňte interval:
   - **Every Minute** - Každou minutu
   - **Every 5 Minutes** - Každých 5 minut
   - **Every Hour** - Každou hodinu
   - **Custom** - Vlastní cron výraz

## 📊 Logování a monitoring

Workflow obsahuje automatické logování:

### ✅ Úspěšné přidání kontaktu
```
✅ Kontakt úspěšně přidán do Leadspeaker:
  email: jan.novak@example.com
  jméno: Jan Novák
  timestamp: 2026-01-06T10:30:00.000Z
```

### ❌ Chyba (chybí email)
```
❌ Kontakt NEPŘIDÁN - chybí email:
  data: { ... }
  timestamp: 2026-01-06T10:31:00.000Z
```

Logy najdete v **Executions** v N8N.

## 🛡️ Bezpečnost

- **API klíče** jsou uloženy bezpečně v N8N credentials
- **Nikdy nesdílejte** workflow soubor s uloženými credentials
- **Exportujte workflow bez credentials** před sdílením
- **Používejte HTTPS** pro všechny API volání

## 🐛 Řešení problémů

### Workflow se nespouští

- ✅ Zkontrolujte, že je workflow **aktivní** (zelené tlačítko)
- ✅ Ověřte **Google Sheets credentials** - zkuste reautorizovat
- ✅ Zkontrolujte **polling interval** - není příliš dlouhý?

### Kontakty se nevytvářejí v Leadspeaker

- ✅ Ověřte **API klíč** - je platný?
- ✅ Zkontrolujte **API endpoint** - je správný?
- ✅ Podívejte se do **Execution logs** - jaká je chybová hláška?
- ✅ Ověřte **formát hlavičky autorizace** v dokumentaci Leadspeaker

### Duplicitní kontakty

- ✅ Leadspeaker může automaticky detekovat duplicity podle emailu
- ✅ Pokud ne, přidejte node pro kontrolu existence kontaktu před vytvořením
- ✅ Můžete použít "Update" místo "Create" pro aktualizaci existujících

### Chybí některá pole

- ✅ Zkontrolujte **názvy sloupců** v Google Sheets - musí odpovídat mapování
- ✅ Ověřte **mapování polí** v node "Mapování polí"
- ✅ Některá pole mohou být **case-sensitive** (záleží na velikosti písmen)

## 📚 Další zdroje

- [N8N dokumentace](https://docs.n8n.io/)
- [Google Sheets Trigger node](https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.googlesheettrigger/)
- [HTTP Request node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)
- [Leadspeaker.com API dokumentace](https://leadspeaker.com/api-docs)

## 💡 Tipy a triky

### 1. Přidejte notifikace
Přidejte node pro odeslání notifikace (Email, Slack, Discord) při úspěšném přidání kontaktu.

### 2. Validace dat
Přidejte node pro validaci emailu a telefonu před odesláním do Leadspeaker.

### 3. Batch processing
Pokud máte hodně řádků najednou, zvažte batch processing pro lepší výkon.

### 4. Error handling
Přidejte node pro zpracování chyb a retry mechanismus při selhání API.

### 5. Webhook místo pollingu
Pro okamžitou synchronizaci použijte Google Sheets webhook místo polling triggeru.

## 🤝 Podpora

Pokud narazíte na problémy:

1. Zkontrolujte **Execution logs** v N8N
2. Ověřte **API dokumentaci Leadspeaker**
3. Otestujte **API endpoint** pomocí Postman/Insomnia
4. Vytvořte **issue** v tomto repozitáři

---

**Vytvořeno s ❤️ pro automatizaci marketingu a CRM**
