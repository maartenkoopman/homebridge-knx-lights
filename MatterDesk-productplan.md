# MatterDesk — Product- & Architectuurplan

> Werktitel: **MatterDesk** (alternatief: **Matterly**) — geleverd en gefactureerd door **WestAI**.
> Status: levend document. Laatst bijgewerkt: 2026-08-13.

Een AI-native "matter center" voor de advocatuur, volledig geïntegreerd in Outlook, Word en Excel.
Dossierdata blijft in de eigen Microsoft 365-omgeving van de klant. Doelgroep: kantoren tot ~10 medewerkers,
focus Nederland en Caribisch Nederland.

---

## 1. Kernfilosofie: tenant-native, near-zero-hosting

MatterDesk is een **product/invoegtoepassing**, geen klassieke SaaS-hoster.

| Laag | Waar draait het | Bevat klantdata? |
|---|---|---|
| Add-in (Outlook/Word/Excel) + manifest | Statische bundel bij WestAI (nu NAS, later Azure) | **Nee** |
| Business logic | Client-side in de add-in, via Microsoft Graph | Nee |
| **Dossierdata (Word/PDF/Excel/e-mail)** | **SharePoint Embedded in de klant-tenant** | Ja — blijft daar |
| Identiteit & rechten | Entra ID van de klant | Nee |
| Metering & billing | WestAI (tellers/rechten, **nooit dossierinhoud**) | Nee |

**Belofte richting klant:** "Jullie dossierdata raakt onze servers nooit aan."

WestAI levert **geen** Azure/M365-licenties — dat loopt via partners. De add-in doet bij onboarding een
**licentie-/permissie-precheck** vóór er iets wordt uitgerold.

---

## 2. Functionele kern

- **Dossier = SharePoint Embedded container.** Rechten worden per container gezet bij aanmaken (basis voor "ethical walls").
- **Document-koppeling bij creatie, niet per opslag.** Een document wordt bij aanmaken aan een dossier gebonden;
  daarna landt elke automatische opslag vanzelf in de juiste container. (Office heeft geen betrouwbare "onderschep elke save"-hook — dus we werken mét de runtime mee, niet ertegen.)
- **Outlook e-mail filing.** Bij verzenden (`OnMessageSend`) een venster met een **voorstel** van recente/relevante dossiers
  (afzender/ontvanger, laatst gebruikt, onderwerp-match).
- **Uren schrijven** per dossier — tenant-native opgeslagen; voedt later de boekhoudkoppeling.
- **Event-/outbox-laag** vanaf dag 1 (bv. "dossier aangemaakt", "uren geschreven"): nu inert, later adapters
  richting NL-boekhoudpakketten (Twinfield, Exact, AFAS). Alleen dun abstraheren, niet hardcoden.

---

## 3. AI — twee modellen, eerlijk naast elkaar

| | **Managed AI (door WestAI)** | **Eigen Foundry (klant-Azure)** |
|---|---|---|
| Waar draait AI | WestAI Azure OpenAI (EU) via gateway | Azure van de klant (Bicep deploy) |
| Wie betaalt verbruik | Klant → **WestAI** (Matter AI tokens) | Klant → **Microsoft** rechtstreeks |
| Verdienmodel WestAI | Tokenmarge | Setup + moduletarief + seats |
| Vertrouwelijkheid | Verwerking bij verwerker (EU, DPA, zero-retention) | Data verlaat tenant nooit — maximaal |
| Voor wie | "Gewoon werken", direct starten | Maximale controle |

**Websitepositionering (eerlijk):** twee kolommen, geen verstopte optie. Eén regel:
*"Beide werken identiek in Outlook, Word en Excel. Kies op basis van gemak versus controle."*

### Vertrouwelijkheidsarchitectuur (twee lagen)
- **Laag A — infrastructuur:** Azure OpenAI in EU-region, verwerkersovereenkomst, geen training, zero-retention.
  Dit is de échte juridische garantie.
- **Laag B — actief filter (verplicht in Managed):** retrieval blijft in de klant-tenant (alleen relevante
  fragmenten, nooit hele dossiers); pseudonimisering van namen met lokale terugvertaling (instelbaar); harde
  blokkade op "exporteer heel dossier naar AI".
- **Afweging bewust maken:** hoe zwaarder het filter, hoe minder scherp de AI-antwoorden. Filter = instelbaar, niet blind altijd-aan.

---

## 4. Prijsmodel

Alle prijzen **config-gestuurd en op elk moment aanpasbaar** (prijstabel met ingangsdatum).
Prijswijzigingsclausule in de algemene voorwaarden (B2B, ±30 dagen aankondiging). Reeds gekochte AI-bundels
worden op de aankoopprijs gehonoreerd.

| Component | Managed AI | Eigen Foundry |
|---|---|---|
| **Eenmalige setup** | €295 | €950 (Bicep-uitrol, Azure-inrichting, budget-guards) |
| **Moduletarief** (per kantoor/maand) | €39 | €39 |
| **Per gebruiker/maand** | **€45** | **€45** |
| **AI** | Bundel inbegrepen per seat + bijkoop credit-packs | Klant betaalt Azure zelf; optioneel €5/seat AI-beheer |

- **Gebruikerstelling = high-water mark:** dagelijkse momentopname, **hoogste aantal in de maand** × €45.
  Altijd **hele maanden**, geen proratie. Exact zo in de voorwaarden vastleggen.
- **Matter AI tokens:** WestAI-eigen credit met marge (richtlijn ~2,5× Azure-kostprijs). Onderliggend model
  onzichtbaar voor de klant → vrij te wisselen zonder klantprijs te raken.
- **Kostenbasis (referentie):** gpt-4o ≈ $2,50/1M input + $10/1M output → ±€0,02 per dossiervraag.
  SharePoint Embedded opslag $0,20/GB/mnd — betaalt de klant, niet WestAI.

### Automatische bijkoop + plafond
- Dashboard toont real-time: tokensaldo, verbruik deze maand, €-teller.
- Klant stelt een **maandplafond in euro's** in; auto-bijkoop tot dat plafond, **daarna stop** (AI pauzeert, rest van het pakket blijft werken).
- Waarschuwing bij 80%, harde stop op 100%, "verhoog plafond" met één klik.

### Facturatie
- Eén maandfactuur = moduletarief + (piek-gebruikers × seatprijs) + bijgekochte AI-bundels.
- **Automatische SEPA-incasso** (Mollie; B2B-machtiging in de onboarding).

### Benchmark (referentie)
- Legal practice management: $39–149/user/mnd. Kleine kantoren $30–70.
- NetDocuments (concurrent): $50–65, reëel $80–120 met add-ons; implementatie $1.000–5.000.
- MatterDesk-wig: lichter, AI-native, eigen data, betaalbaar voor het kleine kantoor.

---

## 5. Fiscaal / juridisch

- Geen prijsregulering op B2B-SaaS in NL — vrij te bepalen.
- Tokens wederverkopen **mag** via eigen Azure-resource (waarde-toevoegende dienst). Licenties doorverkopen mag
  níét zonder Microsoft CSP-status → bewust bij partners belegd.
- **BTW 21%** op NL-facturen. **Caribisch Nederland (BES): ABB i.p.v. BTW** — aparte behandeling, met boekhouder afstemmen vóór eerste BES-klant.
- SEPA **B2B-incasso**: getekende machtiging + bank-preregistratie; in aanmeldflow opnemen.

---

## 6. Hosting-roadmap

- **Nu:** statische add-in-bundel + metering/billing op de NAS. Vereist vast (sub)domein, geldig HTTPS-cert (Let's Encrypt), 24/7 beschikbaar. Kritische productie-afhankelijkheid: ligt de NAS plat → add-in laadt bij niemand.
- **Bij tractie:** metering/billing als eerste naar Azure (betrouwbaarheid/incasso). Daarom nu al als losse container bouwen, niet verweven.
- Klantdata + AI blijven altijd in de klant-tenant — nooit op de NAS.

---

## 7. Naam & merk — te verifiëren

- **MatterDesk** — sterk, maar ⚠️ fonetische nabijheid tot bestaande legal-vendor **Maatdesk** → merkverwarringsrisico.
- Verifieer vóór vastleggen: `matterdesk.nl` (SIDN/registrar) · BOIP-merkcheck (incl. nabijheid Maatdesk) · KVK-handelsnaam.
- **Matterly** — door gebruiker als vrij gezien; veilige tweede keuze (geen branche-collision).
- Vermijd "Matter Center" als merk (= Microsofts eigen dode product).

---

## 8. Roadmap / open punten

- **MVP:** Outlook e-mail filing + dossier/container-aanmaak met rechten + document-koppeling bij creatie.
- **Kort daarna:** uren schrijven; AI-bevraging (Managed) met filter + dashboard/plafond.
- **Later:** boekhoudkoppeling (adapters op de event-laag); Eigen Foundry Bicep deploy-knop met versiebeheer + budget-guards.
- **Open:** definitieve naamkeuze na verificatie; exacte AI-bundelgrootte per seat; hoogte credit-pack-marge; keuze Mollie vs Stripe.
