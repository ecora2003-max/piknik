# Picnic (DE): Technische API-Recherche

*Stichtag: 06.11.2026. Quellen: Quelltexte öffentlicher Community-Projekte und offizielle Picnic-Nutzungsbedingungen. Keine eigenen authentifizierten Live-Abfragen; keine Bestellung.*

## 1. Quellen und Kandidaten

- Primärbibliothek: [MRVDH/picnic-api](https://github.com/MRVDH/picnic-api), Node.js/TypeScript, MIT-Lizenz des **Wrappers**; aktiv gepflegte 5.x-API.
- Alternative: [aminm1364/mcp-de-picnic](https://github.com/aminm1364/mcp-de-picnic), eigenständige Implementierung für DE/NL, [ENDPOINTS.md](https://github.com/aminm1364/mcp-de-picnic/blob/main/ENDPOINTS.md), kein Checkout.
- Älterer Python-Wrapper: [MikeBrink/python-picnic-api](https://github.com/MikeBrink/python-picnic-api).

Picnic bietet nach den geprüften Quellen **keine öffentliche, freigegebene Endkunden-API**. Private App-Endpunkte sind nicht vertraglich stabil.

## 2. Anmeldung und API-Basis

Quelle: [http-client.ts](https://github.com/MRVDH/picnic-api/blob/main/src/http-client.ts), [auth/service.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/auth/service.ts).

- `new PicnicClient({ countryCode: "DE" })`.
- Basis-URL gemäß Wrapper-Quellcode: `https://storefront-prod.de.picnicinternational.com/api/15`.
- Login: `client.auth.login(email, password)` (intern `POST /user/login`); nach Bedarf `generate2FACode("SMS")` und `verify2FACode(code)`.
- Token im Header `x-picnic-auth`; teils zusätzliche Client-Header erforderlich. Keine Zugangsdaten oder Sitzungs-IDs committen, loggen oder an Drittanbieter übertragen.
- Formatmigration der Picnic-App: Fusion-JSON und React Server Components (RSC); der Wrapper kann infolge von App-Änderungen vorübergehend versagen.

## 3. Sortiment und Preisdaten

Quellen: [catalog/service.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/catalog/service.ts), [catalog/types.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/catalog/types.ts), [catalog/categories.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/catalog/categories.ts).

| Methode | Zweck |
|---|---|
| `catalog.getSearchResults("milch")` | Suche mit Produktabschnitten/Filtern |
| `catalog.getCategories()` | Kategorienbaum |
| `catalog.getSubcategories(categoryId)` | Unterkategorien |
| `catalog.getCategoryProducts(categoryId)` | Artikel in einer Kategorie |
| `catalog.getProduct(productId)` | Produktdetails |
| `catalog.getImageUrl(imageId)` | Produktbild |

Die öffentliche Typisierung enthält `id` (Picnic Selling-Unit-ID), `name`, `brand`, `unitQuantity`, `displayPrice` (**Cent**), `originalPrice` (optional), `unitPrice` (optional, Text), `priceRanges` (Mengenstaffeln), `labels` und bei Produktkacheln `isUnavailable`. Auch Nährwerte/Allergene können in Produktdetails verfügbar sein.

**Nicht nachgewiesen:** vollständige Produktliste/Bulk-Export; EAN-/GTIN-Felder in den öffentlichen Typen; lückenlose Grundpreise. Kategorien können technisch besucht werden, garantieren aber keine Vollständigkeit. Suchergebnisse können personalisiert oder adressabhängig sein.

## 4. Angebotsliste

Quellen: [catalog/service.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/catalog/service.ts), [catalog/promotions.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/catalog/promotions.ts).

- `catalog.getPromotions()` holt die Aktionsstartseite `promo-page-root`, im untersuchten Wrapper als Fusion-Seite.
- `catalog.getCategoryPromotions(categoryId)` listet Aktionen einer Kategorie.
- `catalog.getPromotionGroup(groupId)` löst Gruppenkampagnen zu Einzelprodukten auf.
- Aktionskacheln enthalten Preis-/Rabattinformationen (`originalPrice`, `displayPrice`, `labels`) und können Mengenstaffeln enthalten.
- **Kein zugesichertes strukturiertes Start-/Enddatum** in den untersuchten `Promotions`-/`ProductTile`-Typen. Gültigkeit darf nicht aus dem Abrufzeitpunkt abgeleitet werden.
- Personalisierte Angebote und Kombinationsrabatte erfordern gesonderte Behandlung; ein einfaches „Rabatt-%“ ist oft unzutreffend.

## 5. Warenkorb, Lieferung und Bestellung

Quellen: [cart/service.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/cart/service.ts), [checkout/service.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/checkout/service.ts), [checkout/types.ts](https://github.com/MRVDH/picnic-api/blob/main/src/domains/checkout/types.ts).

Warenkorb:
- `cart.getCart()` → Warenkorb, einschließlich `mts` für Änderungen.
- `cart.addProductToCart(productId, count)` → `POST /cart/add_product`.
- `cart.removeProductFromCart(productId, count)` → `POST /cart/remove_product`.
- `cart.clearCart()`, `cart.getDeliverySlots()`, `cart.setDeliverySlot(slotId)`.

Checkout:
1. `checkout.startCheckout({ mts: cart.mts, ... })` → `POST /cart/checkout/start`.
2. `checkout.initiatePayment(orderId, appReturnUrl)` → `POST /cart/checkout/initiate_payment`, kann externe Zahlungsfreigabe erfordern.
3. `checkout.getCheckoutStatus(transactionId)` → Zahlungs-/Checkoutstatus.
4. `checkout.confirmOrder(orderId)` → **bestätigt die Bestellung**, darf nie unbeaufsichtigt ausgeführt werden.
5. `checkout.getOrderStatus(orderId)` → Bestellstatus.

Existierende Methoden sind **kein Beleg**, dass der gesamte Checkout in DE mit jedem Zahlungsweg funktioniert; dies wurde nicht getestet. Vor Bestellabschluss ausdrückliche aktuelle Zustimmung nach Anzeige von Warenkorb, Preis und Termin einholen.

## 6. Vertragliche und technische Grenzen

Quelle: [Offizielle deutsche Picnic-AGB/Nutzungsbedingungen](https://picnic.app/de/agb/), Abschnitt III.3.2–3.4. Ohne ausdrückliche schriftliche Zustimmung untersagt Picnic systematische Extraktion und Wiederverwendung wesentlicher Teile seiner Produkt- und Preisdaten sowie den Aufbau einschlägiger Datenbanken. Eine Wrapper-Open-Source-Lizenz hebt diese Grenzen nicht auf. Für eine produktive Datenbank der Android-App vorab ausdrückliche Datenfreigabe/Partnerschnittstelle klären.

Weitere Risiken: undokumentierte Änderungen, API-Rate-Limits, Sperrungen, Liefergebiet, personalisierte Preise/Rabatte, nicht strukturierte Gültigkeitsangaben und Zahlungsfreigabe.

## 7. Validierungsplan (noch offen)

- [x] Funktionsumfang im Open-Source-Quellcode nachvollzogen.
- [x] Deutsche API-Basis und 2FA im Quellcode identifiziert.
- [x] Checkout-Routen identifiziert.
- [ ] Berechtigung für geplante Datenextraktion und Speicherung klären.
- [ ] Mit eigenem DE-Konto und gültigen Secrets read-only testen: Login, 2FA, Suche, Kategorien, Produktdetails, Aktionen und Lieferfenster.
- [ ] Zehn Stichproben mit der Picnic-App vergleichen (Preise, Grundpreise, Aktionsbedingungen, Verfügbarkeit).
- [ ] EAN/GTIN-Verfügbarkeit, Abdeckung und Gültigkeitsfelder nachweisen; keine unbelegten Zuordnungen für Ernährungsgedächtnis.
- [ ] Warenkorb-Mutationen nur nach expliziter Genehmigung; Checkout niemals automatisch testen.
- [ ] Für zulässige Nutzung einen isolierten Connector mit Token-Schutz, nachweisbaren Preisen in Cent, Zeitstempeln und Qualitätsflags erstellen.

**Empfehlung:** `picnic-api` (DE) als erste technische Basis; vor produktiver Preis-/Angebotsdatenbank Erlaubnis klären. Keine Funktionsfähigkeit ohne authentifizierten DE-Test behaupten.
