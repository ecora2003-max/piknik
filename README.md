# piknik – Picnic-API-Recherche

**Stand: 06.11.2026 | Status: Quellcode-Recherche abgeschlossen, authentifizierter Live-Test offen.**

Ziel dieses Repositories: APIs für das **Picnic-Sortiment**, **Preise**, **aktuelle Angebote**, **Warenkorb und Bestellungen** untersuchen, insbesondere für eine mögliche Integration in die App „Ernährungsgedächtnis“.

## Ergebnis

Eine offiziell freigegebene öffentliche Endkunden-API wurde nicht gefunden. Vielversprechend ist der **inoffizielle** TypeScript-/Node.js-Wrapper [MRVDH/picnic-api](https://github.com/MRVDH/picnic-api) ([npm](https://www.npmjs.com/package/picnic-api)). Dieser enthält im Quellcode Deutschland-Unterstützung (`countryCode: "DE"`) und Methoden für:

| Bereich | Nachgewiesene Wrapper-Methoden | Einschränkung |
|---|---|---|
| Sortiment | `catalog.getSearchResults`, `getCategories`, `getSubcategories`, `getCategoryProducts`, `getProduct` | Kein garantierter kompletter Sortimentsexport |
| Preise | `displayPrice`, `originalPrice`, `unitPrice`, `priceRanges` | Werte in Cent, Grundpreise teils nicht vorhanden; keine gesicherte EAN/GTIN |
| Angebote | `catalog.getPromotions`, `getCategoryPromotions`, `getPromotionGroup` | Aktionslaufzeit nicht durchgehend strukturiert; personalisierte Aktionen |
| Warenkorb | `cart.getCart`, `addProductToCart`, `removeProductFromCart` | Noch nicht mit deutschem Konto getestet |
| Lieferfenster | `cart.getDeliverySlots`, `setDeliverySlot` | Noch nicht live getestet |
| Bestellung | `checkout.startCheckout`, `initiatePayment`, `confirmOrder`, `getOrderStatus` | Zahlungs-/Bestellablauf nicht live getestet; keine unbeaufsichtigten Käufe |

**Alternative:** [aminm1364/mcp-de-picnic](https://github.com/aminm1364/mcp-de-picnic) (Deutschland-/Niederlande-Fokus, kein Checkout). Ältere Python-Alternative: [MikeBrink/python-picnic-api](https://github.com/MikeBrink/python-picnic-api).

**Technisch:** Inoffizielle Basis-URL laut Quellcode `https://storefront-prod.de.picnicinternational.com/api/15`; eigene Anmeldung inkl. möglicher SMS-2FA. App-Änderungen zwischen Fusion-JSON und React Server Components können Funktionen brechen.

**Wichtige Nutzungsgrenze:** [Picnics deutsche AGB](https://picnic.app/de/agb/) untersagen unter III.3.2–3.4 ohne ausdrückliche schriftliche Zustimmung die systematische Extraktion und Wiederverwendung wesentlicher Produkt-/Preisdaten sowie den Aufbau entsprechender Datenbanken. Für einen produktiven Preiskatalog in einer externen App muss die Datennutzung zuerst geklärt werden. Die MIT-Lizenz des Wrappers ist keine Lizenz für Picnics Inhalte.

Eine Bestellung oder Zahlung darf nur nach expliziter Freigabe ausgelöst werden. Authentifizierungsdaten und Tokens dürfen niemals in Git gespeichert werden.


## Knuspr (Rohlik) – offiziell verfügbarer MCP-Zugang

**Stand: 06.11.2026.** Für **Knuspr.de** wurde eine **offizielle** Schnittstelle gefunden: [Knuspr MCP](https://www.knuspr.de/mcp-docs), Endpunkt `https://mcp.knuspr.de/mcp`, Anmeldung per OAuth. Damit sind Produktsuche, Preise, Verfügbarkeit, aktuelle Angebote, Warenkorbverwaltung sowie Bestellhistorie/-status laut Hersteller verfügbar. Ein **vollständiger Preiskatalogexport** und der **verbindliche Bestellabschluss** sind öffentlich nicht nachgewiesen; dafür sind Tool-Discovery nach Anmeldung bzw. Händlerfreigabe erforderlich.

Für die Kotlin-Android-App bietet sich das [offizielle MCP-Kotlin-SDK](https://github.com/modelcontextprotocol/kotlin-sdk) an. Detaillierte Funktionsmatrix, alternative Read-only-REST-API (ReefAPI), rechtliche Grenzen und Test-/Integrationsplan:

➡️ [**docs/KNUSPR-API.md**](docs/KNUSPR-API.md)

## Technische Dokumentation

[docs/API-RECHERCHE.md](docs/API-RECHERCHE.md) dokumentiert die geprüften Methoden und Quellcodepfade, Datenfelder, API-Routen, offenen Nachweise und nächsten Validierungsschritte.
