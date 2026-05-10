## buildingpay-v29

> Modulo custom per la gestione di amministratori di condomini su piattaforma Odoo Enterprise 17.

# BuildingPay v9 - Odoo EE 17 Module

## Panoramica
Modulo custom per la gestione di amministratori di condomini su piattaforma Odoo Enterprise 17.
Installato su ambiente odoo.sh con 3 siti web esistenti.

## Dipendenze
```
base, base_setup, website, portal, account, purchase, product,
mail, auth_signup, l10n_it_edi
```
Librerie Python aggiuntive nel virtualenv Odoo:
- `python-docx` — sostituzione placeholder nel template contratto
- `openpyxl` — generazione e lettura file Excel

## Struttura del modulo

```
buildingpay_v12/
├── models/
│   ├── buildingpay_config.py      # Configurazione singleton (sito web, email, attività)
│   ├── res_partner.py             # Estensione contatti (is_amministratore, condomini, contratto)
│   ├── product_pricelist.py       # Flag listino_condominio + % retrocessioni
│   ├── product_template.py        # Flag is_condominio_pagopa
│   ├── buildingpay_import.py      # Import Excel → fatture + ordini acquisto
│   └── buildingpay_import_error.py
├── views/
│   ├── buildingpay_config_views.xml
│   ├── res_partner_views.xml      # Estensione form/list/search contatti
│   ├── product_pricelist_views.xml
│   ├── product_template_views.xml
│   ├── buildingpay_import_views.xml
│   └── buildingpay_menus.xml
├── controllers/
│   ├── portal_auth.py             # Registrazione con referrer (/web/signup?referrer=CODE)
│   └── portal_main.py             # /my/contratto e /my/condomini
├── templates/
│   ├── portal_home_inherit.xml    # Aggiunge sezioni alla home portale
│   ├── portal_registration.xml   # Form registrazione esteso
│   ├── portal_contratto.xml      # Sezione Contratto Generale
│   └── portal_condomini.xml      # Sezione Condomini (lista + form)
├── security/
│   ├── buildingpay_security.xml  # Gruppi di sicurezza
│   └── ir.model.access.csv
├── data/
│   ├── mail_template_data.xml    # "Mail benvenuto amministratore"
│   └── ir_cron_data.xml          # Cron 23:30 + sequenza import
└── static/
    └── modello_importazione_fatture.xlsx  # Template Excel per import fatture
```

## Gruppi di sicurezza
| ID XML | Nome | Permessi |
|--------|------|----------|
| `group_buildingpay_user` | Utente BuildingPay | Lettura configurazione |
| `group_buildingpay_manager` | Manager BuildingPay | Scrittura configurazione, gestione checkbox Amministratore |
| `group_responsabile_amministrativo` | Responsabile Amministrativo Condomini | Menu Contabilità (import fatture) |

## Modello `buildingpay_v12.config`
Singleton per sito web. Campi chiave:
- `website_id` — sito web su cui è attivo il modulo
- `create_activity_on_contract` + `activity_responsible_id` + `activity_days` — attività automatica
- `contratto_template` (Binary) — template .docx Accordo Condomini Aggregati
- `condomini_attivati_email` / `condomini_dismessi_email` — destinatari report Excel

## Estensione `res.partner`
Campi aggiunti:
- `is_amministratore` (Boolean) — solo Manager può modificare
- `referrer_code` (Char, auto-generato) — codice univoco per link referral
- `referrer_id` (Many2one res.partner) — chi ha invitato questo admin
- `listino_condominio_id` — listino con flag `is_listino_condominio`
- `accordo_condomini_aggregati` (Boolean) — attivato all'upload contratto
- `fiscalcode`, `pec_mail`, `codice_destinatario` — dati italiani
- `electronic_invoice_subjected`, `electronic_invoice_obliged_subject`
- `type` esteso con `'condominio'`

**Nota compatibilità l10n_it_edi**: i campi `fiscalcode`, `pec_mail`, `codice_destinatario`
sono definiti nel modulo BuildingPay. Se l10n_it_edi li definisce già con nomi diversi
(es. `l10n_it_codice_fiscale`), aggiungere un `related` o adattare i nomi.

## Flusso registrazione portale
1. Amministratore esistente condivide link: `/web/signup?referrer=<referrer_code>`
2. Nuovo utente si registra → `is_amministratore = True`, `referrer_id` salvato
3. Odoo invia email attivazione (standard) + email benvenuto (template "Mail benvenuto amministratore")

## Flusso Contratto Generale (portale)
1. Download: `/my/contratto/download` — sostituisce `[NOME AMMINISTRATORE]` e `[________]` con python-docx
2. Upload: `/my/contratto/upload` — salva file, attiva `accordo_condomini_aggregati`, crea attività se configurato

## Flusso import fatture
File Excel colonne: `id_esterno_condominio | data_fattura | quantita | descrizione_riga | prezzo_unitario`
- Raggruppamento per (condominio, data) → una fattura per gruppo
- Prezzo: dal listino `listino_condominio_id` dell'amministratore (fallback: valore Excel)
- Prodotto: primo con `is_condominio_pagopa = True`
- PO amministratore: `% retrocessione_amministratore × imponibile fattura`
- PO referrer: `% retrocessione_referrer × imponibile fattura`

## Azione pianificata
- Modello: `res.partner`, metodo: `action_send_daily_condomini_report()`
- Frequenza: ogni giorno alle 23:30 (regolare `nextcall` dopo installazione)
- Output: Excel con tutti i condomini attivi → email a `condomini_attivati_email`

## Convenzioni di codice
- Lingua commenti/stringhe: Italiano (interfaccia), inglese (codice)
- Logging: `_logger = logging.getLogger(__name__)` in ogni file
- Sudo: usare `.sudo()` nei controller portale per accesso dati
- Errori: usare `UserError` per errori utente, `ValidationError` per vincoli
- XML IDs: prefisso `buildingpay_` o `view_buildingpay_`

## Note di deployment (odoo.sh)
1. Copia la cartella `buildingpay_v12/` in `addons/` del repository
2. Installa `python-docx` e `openpyxl` nel requirements.txt del progetto
3. Aggiornare la lista applicazioni e installare il modulo
4. Configurare `buildingpay_v12.config` per ogni sito web attivo
5. Caricare il template "Accordo Condomini Aggregati.docx" nella configurazione
6. Verificare il `nextcall` dell'azione pianificata (impostare alle 23:30 ora locale)

---
> Source: [FrancescaBianchini/BuildingPay_v29](https://github.com/FrancescaBianchini/BuildingPay_v29) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-04-29 -->
