# Knuspr (Rohlik): API-Recherche und Integrationsplan

**Recherchestand: 06.11.2026**
**Status: Offiziellen MCP-Endpunkt und öffentliche Dokumentation verifiziert; keine authentifizierte Session und keine reale Bestellung durchgeführt.**

## 1. Wichtigstes Ergebnis: Es gibt eine OFFIZIELLE Knuspr-MCP-Schnittstelle

- Dokumentation: https://www.knuspr.de/mcp-docs
- Ergänzende Funktionsübersicht: https://www.knuspr.de/seite/mcp-docs
- **Endpunkt (MCP über HTTP): `https://mcp.knuspr.de/mcp`**
- Laut Knuspr mit **OAuth-Anmeldung eines aktiven Knuspr-Kontos** (kein Passwort in App oder Repository ablegen).
- Offiziell beworbene Fähigkeiten: Produkte suchen, aktuelle Preise/Verfügbarkeit prüfen, Angebote anzeigen, Warenkorb ändern, Bestellhistorie/-status lesen, frühere Bestellungen wiederholen bzw. wieder zusammenstellen, Rezepte, Einkaufslisten und Kundendienst.

**Wichtige Grenze:** Die öffentlich zugänglichen Beschreibungen veröffentlichen keine verbindliche Liste aller MCP-Toolnamen mit JSON-Schemas. Nach erfolgreicher OAuth-Authentifizierung muss der Client `tools/list` ausführen und die Schemas selbst auslesen. **Ein endgültiger Checkout/Bezahlvorgang wird nicht ausdrücklich dokumentiert und darf vor einem Live-Nachweis nicht als unterstützt gelten.** „Frühere Bestellungen wiederholen“ ist kein Nachweis für eigenständige Zahlungs-/Bestellbestätigung.

### Evidenzmatrix

| Ziel | Offizielle MCP-Dokumentation | Beweisgrad / offene Frage |
|---|---|---|
| Produktsuche | Ja | Fähigkeiten belegt; genaue Toolnamen und Suchparameter erst nach `tools/list` |
| Einzelpreis | Ja, Produktpreise vergleichen | Produktidentifikatoren, Staffel-/Grundpreise und Antwortschema offen |
| Regulärer **vollständiger Preiskatalog** | Nicht zugesichert | Kein Bulk-Export belegt, Nutzung/Lizenzen separat klären |
| Aktuelle Angebote | Ja, Promotions/Deals | Konditionen/Laufzeitfelder und vollständige Erfassung offen |
| Verfügbarkeit | Ja | Lieferadresse/Regionalisierung untersuchen |
| Warenkorb | Ja, hinzufügen/entfernen/Menge ändern | Reale Mutationen nur nach ausdrücklicher Zustimmung |
| Bestellungen/Status | Ja, Historie/Tracking | Lesender Zugriff belegt |
| Neue **verbindliche Bestellung** | Nicht ausdrücklich dokumentiert | Nur mit ausdrücklich verfügbarer Checkout-Funktion + Nutzerfreigabe; sonst Übergabe zum Knuspr-Webcheckout |

Offizieller Endpunkt in der eigenen Doku: `https://mcp.knuspr.de/mcp`.
Beispiel-Verbindung des Anbieters (Claude CLI): `claude mcp add --transport http knuspr-mcp https://mcp.knuspr.de/mcp` mit anschließender OAuth-Anmeldung.

## 2. Android-Anbindung: bevorzugte Architektur

Das Android-Projekt „Ernährungsgedächtnis“ kann einen **isolierten Knuspr-Connector** erhalten:

1. Benutzer wählt „Knuspr verbinden“ und authentifiziert sich in einem sicheren externen OAuth-Browserfluss (keine Übergabe von Knuspr-Passwort an unsere App).
2. Ein **MCP-Client** verbindet sich über Streamable HTTP mit der offiziellen URL; nach erfolgreicher Verbindung Capability-/Tool-Discovery über `tools/list`.
3. Für die Einträge auf der Einkaufsliste passende Knuspr-Artikel suchen; Ergebnisse nach Marke, Packungsmenge, Einheit und ggf. EAN validieren. Keine fingierten ID-Übereinstimmungen.
4. **Read-only-Modus zuerst:** Preis/Grundpreis (falls geliefert), Aktionshinweise und Lieferbarkeit holen, in vergleichbare Einheiten normalisieren; Preis in Integer-Cent und mit Zeitstempel/Quelle kennzeichnen.
5. Nutzer wählt Artikel und Mengen aus; **erst nach ausdrücklicher Bestätigung** Warenkorb ändern.
6. Vor einer Bestellung Gesamtsumme, Lieferfenster, Ersatzartikel, Gebühren und Kaufbestätigung gesondert prüfen. Wenn das offizielle MCP keinen vollständigen Checkout anbietet, für den tatsächlichen Kauf **zum offiziellen Knuspr-Checkout weiterleiten** statt nicht dokumentierte Bestellrouten aufzurufen.

Mögliche Implementierungsbasis: offizielles [Kotlin MCP SDK](https://github.com/modelcontextprotocol/kotlin-sdk), Artefakt `io.modelcontextprotocol:kotlin-sdk-client`, mit geeignetem Ktor-HTTP-Client. Das SDK stellt `listTools`, `callTool` und Streamable HTTP bereit. **OAuth-Interoperabilität mit Knuspr und Android-Redirects ist noch zu testen**; keine feste Tool-Signatur erfinden.

Empfohlenes gemeinsames Datenmodell (als Planung, kein belegtes Knuspr-Schema):
```kotlin
data class RetailerOffer(
    val retailer: String = "KNUSPR",
    val sourceProductId: String,
    val name: String,
    val brand: String?,
    val packQuantityText: String?,
    val priceCents: Long?,
    val originalPriceCents: Long?,
    val unitPriceText: String?,
    val promotionText: String?,
    val available: Boolean?,
    val observedAt: java.time.Instant,
    val matchConfidence: Double
)
```
Die Rohantwort soll in einen Adapter übersetzt werden; niemals nicht vorhandene Preis- oder EAN-Felder stillschweigend ergänzen.

### Sicherheit

- Token sicher speichern (z. B. Android Keystore / verschlüsselte Sitzung nach Plattformempfehlungen), keine OAuth-Tokens in Crash-Logs, GitHub oder Telemetrie.
- **Keine automatische Bestellung, Zahlung oder Warenkorbänderung** ohne Nutzerinteraktion.
- Rate-Limits, Gerätewechsel, Berechtigungsentzug, Redirect-Sicherheit und Abfragefehler behandeln.
- Kein stiller vollständiger Katalog-Crawler; nur für einen begrenzten, vereinbarten Zweck abfragen.

## 3. Weitere Datenquellen als Alternativen, nicht mit „offiziell“ verwechseln

| Kandidat | Preis-/Katalogfunktion | Warenkorb/Bestellung | Bewertung |
|---|---|---|---|
| **Offizielles Knuspr MCP** – https://www.knuspr.de/mcp-docs | Produktsuche, Preise, Verfügbarkeit, Angebote | Warenkorb und Bestellinfos; Checkout ungeklärt | **Priorität 1** |
| **ReefAPI Rohlik** – https://reefapi.com/docs/rohlik | Drittanbieter-REST: `POST /rohlik/v1/search`, `/product`, `/categories`, `/category`, `/promotions`, `country:"de"` | Laut Anbieter ausdrücklich **read-only**, keine Bestellungen | Für Preisrecherche technisch interessant, **kein** offizieller Knuspr-Vertrag; Datenrechte/Preismodell klären |
| **rohlik-api** – https://pypi.org/project/rohlik-api/ | Inoffizieller Python-Wrapper: Suche, Produktpreise, Karteikarten, Wochenangebote | Warenkorb, Bestellstatus; kein belegtes Checkout | Fallback nur mit bewusster Entscheidung über private API |
| **knuspr-api** – https://pypi.org/project/knuspr-api/ | Inoffizieller Python-Wrapper mit Suche | Warenkorb, Lieferfenster, Bestellhistorie; kein Checkout-Nachweis | Fallback, privater Endpoint |
| **Apify knuspr-products** – https://apify.com/bakos_bence/knuspr-products | Drittanbieter-Scraper für Kategorien, Produktpreise, URLs | Kein Checkout | Weniger empfehlenswert: unbekannte Datenrechte und Stabilität |

**ReefAPI-Beispiel für lesende Produktabfrage (Drittanbieter, API-Key erforderlich, Gebühren beachten):**
```bash
curl -X POST https://api.reefapi.com/rohlik/v1/search \
  -H "x-api-key: $REEF_KEY" \
  -H "content-type: application/json" \
  -d '{"query":"Milch","country":"de","page":1,"page_size":24}'
```
ReefAPI nennt für `search` 3 Credits pro erfolgreich bearbeitetem Aufruf und bis zu 60 Artikel pro Seite; die angegebene REST-API umfasst fünf **lesende** Endpunkte. Der Dienst behauptet Live-Daten ohne Knuspr-Anmeldung und eigene EAN-Informationen für Einzelprodukte. Diese Anbieterangaben wurden nicht selbst live validiert; Lizenz von ReefAPI bedeutet nicht automatisch, dass Knuspr die Weiterverwendung für unseren Katalog gestattet.

## 4. Rechtliche Verwendung der Daten

Knusprs [AGB](https://www.knuspr.de/seite/agb-general) untersagen in den Nutzungsbedingungen des E-Shops das systematische/automatisierte Beobachten bzw. Kopieren bereitgestellter Inhalte ohne schriftliche Zustimmung. Eine **offiziell bereitgestellte MCP-Schnittstelle** stellt eine ausdrückliche technische Integrationsmöglichkeit für die dort kommunizierten Funktionen bereit, aber **kein pauschales Recht auf vollständige Katalogspiegelung, unbegrenzte Preisarchivierung oder kommerzielle Weiterverwertung**. Vor einer kommerziellen oder massenhaften Datennutzung schriftlich Nutzungsumfang, Limits, Speicherfristen und Lizenz klären.

Die öffentliche Seite https://www.knuspr.de/seite/kontakt/ nennt `partner@knuspr.de`; die dort beschriebene Partnerrolle bezieht sich jedoch primär auf Produktlieferanten, nicht auf eine explizite offene Datenlizenz.

## 5. Technische Abnahme / nächster Schritt

- [x] Offiziellen Knuspr-MCP-Endpunkt bestätigt.
- [x] Offizielle Authentifizierungsart (OAuth) bestätigt.
- [x] Öffentliche Fähigkeiten Produktsuche/Preise/Angebote/Warenkorb/Bestellinformationen dokumentiert.
- [x] Kotlin-MCP-Client als Implementierungsbasis identifiziert.
- [ ] OAuth-Verbindung mit eigenem Knuspr-Konto herstellen.
- [ ] `tools/list` ausführen und **echte Toolnamen, Parameter und Antwortformate** protokollieren (ohne persönliche Daten).
- [ ] 5–10 Stichproben auf Packungsgröße, Grundpreis, Rabatt, Verfügbarkeit und Aktualität mit Website vergleichen.
- [ ] Lieferregion/-adresse und Preisvarianten untersuchen.
- [ ] Prüfen, ob vollständige Kategorienlisten, EAN oder offizielle Bulk-Preiskatalogfunktionen überhaupt verfügbar sind.
- [ ] Ermitteln, ob Bestellabschluss / Checkout als explizites MCP-Tool angeboten wird.
- [ ] Falls nicht: Benutzerführung bis zum Warenkorb, Weiterleitung zum Knuspr-Checkout.
- [ ] Datenschutz-/Nutzungsfreigabe für die beabsichtigte Speicherung/Weiterverwendung klären.
- [ ] Erst danach produktiven Connector in der Android-App entwickeln und Integrationstests mit Mock-Daten durchführen.

## Quellen

1. https://www.knuspr.de/mcp-docs (offiziell)
2. https://www.knuspr.de/seite/mcp-docs (offiziell, Einrichtungsanleitung)
3. https://www.knuspr.de/seite/agb-general (offizielle AGB)
4. https://github.com/modelcontextprotocol/kotlin-sdk (offizielles MCP-Kotlin-SDK)
5. https://kotlin.sdk.modelcontextprotocol.io/kotlin-sdk-client/index.html (Client-API)
6. https://reefapi.com/docs/rohlik (Drittanbieter, read-only)
7. https://pypi.org/project/rohlik-api/ (inoffiziell)
8. https://pypi.org/project/knuspr-api/ (inoffiziell)
