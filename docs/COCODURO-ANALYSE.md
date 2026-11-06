# cocoduro: Recherche zu Preisquellen und Android-App

**Stand: 06.11.2026**  
**Untersuchungsstatus: Öffentlich verfügbare Metadaten und Eigenangaben gesichtet; APK-Binärdatei nicht beschafft, daher KEINE Dekompilierung und KEIN Netzwerk-Mitschnitt.**

## Ausgangsfrage

Bezieht cocoduro reguläre Preise von REWE, ALDI Süd und Knuspr direkt über deren Händler-APIs, durch eigene Server/Scraper oder über legitimierte Partnerdatenfeeds? Die Vermutung einer fehlenden Händlerzustimmung ist **nicht belegt**. Selbst der technische Nachweis einer API-Nutzung könnte das Fehlen einer Erlaubnis nicht beweisen.

## Verifizierbare Befunde

| Punkt | Nachweis | Bedeutung |
|---|---|---|
| App-Paket | `com.cocoduro.app.android` laut Google Play | Identifizierbar für spätere APK-Prüfung |
| Version | 1.24, gemeldet zum 27.10.2026 (Chrome-Stats) | Referenzstand; vor Test APK-Hash und Signatur sichern |
| Händler | REWE, ALDI Süd, Knuspr laut cocoduro-Website | Mehrhändlerpreisvergleich ist Produktfunktion |
| Preise | Angebots-/Normalpreise, Grundpreise, Packungsgrößen und Warenkorbsummen laut Anbieter-Blog | Datenbedarf umfasst mehr als Prospekt-Teaser |
| REWE-Warenkorb | Anbieter nennt Übergabe/Übernahme zu REWE | Kein Schluss auf direkte Händler-API |
| Infrastruktur | cocoduro-Datenschutzerklärung nennt AWS für Website-Hosting und CloudFront; Amazon Bedrock für KI | KEIN Nachweis, wo die Preisdatenbeschaffung läuft |
| Affiliate-Dienste | cocoduro-Datenschutzerklärung nennt Awin und Amazon Partnerprogramm | Affiliate-Verbindung ist denkbar, aber keine bestätigte konkrete Produktdaten-Lizenz |
| Potenziell legaler Bezugsweg | Awin-Dokumentation beschreibt Händler-Produktdatenfeeds für Preisvergleichsportale; REWE bietet ein Affiliate-Programm | Es gibt Alternativen zu unautorisierten API-Zugriffen; dass cocoduro diese nutzt, bleibt unklar |
| Open-Source-Alternative | `ByteSizedMarius/rewerse-engineering` demonstriert prinzipiell nichtoffizielle REWE-App-API-Zugriffe | Existenz eines unabhängigen Projekts belegt **nichts** über cocoduro |

## Externe Quellen

- [Google Play – cocoduro](https://play.google.com/store/apps/details?id=com.cocoduro.app.android)
- [Chrome-Stats – App-Metadaten](https://chrome-stats.com/d/com.cocoduro.app.android)
- [cocoduro – Preisvergleich](https://cocoduro.de/blog/lebensmittel-preisvergleich-einkaufsliste)
- [cocoduro – Warenkorb](https://cocoduro.de/blog/smhaggle-alternative)
- [cocoduro – Datenschutzerklärung](https://cocoduro.de/datenschutz)
- [Awin – Produktdatenfeeds für Preisvergleiche](https://success.awin.com/articles/de/Knowledge/Product-Data-Feed-FAQ)
- [REWE – Partnerprogramm](https://www.rewe.de/partnerprogramm/)
- [Unabhängiger REWE-Wrapper](https://github.com/ByteSizedMarius/rewerse-engineering)

## Was eine APK-Analyse zeigen könnte

1. Original-APK aus Google Play oder nachprüfbarer Originalquelle beschaffen; Versionscode, SHA-256, APK-Signatur und Paketnamen festhalten.
2. Manifest und DEX/RN-/Hermes-JS-Bundles statisch analysieren (z. B. `jadx`, `apktool`, `strings`). Nach Hostnamen, API-Basispfaden, `rewe`, `aldi`, `knuspr`, `price`, `search`, `graphql`, `catalog` suchen.
3. Treffer klassifizieren: eigener cocoduro-Backend-Endpunkt vs. direkter Händler-Host; SDK-/Marketing-/Affiliate-URLs von Preisabrufen unterscheiden.
4. In einer kontrollierten Testumgebung **nur normale, autorisierte App-Bedienung** beobachten: Netzwerkverbindungen, DNS/SNI/Hosts und – soweit transparent und technisch möglich – HTTP-Zielpfade. Keine fremden Konten, kein Umgehen von Authentifizierung.
5. Wenn nur cocoduro-eigene Server kontaktiert werden: Die APK kann grundsätzlich nicht zeigen, ob deren Server später Händler-APIs, Scraper oder Partnerfeeds verwenden. Dann sind serverseitige Quelle/Verträge nicht aus dem Client ableitbar.
6. Im Untersuchungsbericht jede Aussage als BELEGT / PLAUSIBEL / UNBELEGT markieren und konkrete Strings, Datei-/Methodenpfade, Signatur und Zeitstempel als Reproduktionsnachweis angeben.

**Beweisgrenze:** Auch direkte Anfragen an `rewe.de` o. ä. würden eine Nutzung technischer Schnittstellen zeigen, nicht die *fehlende* Zustimmung. Ebenso beweist ein Awin-Eintrag keine tatsächliche Preisdaten-Lizenz.

## Ergebnis derzeit

- [x] App eindeutig identifiziert.
- [x] Umfang des Preisvergleichs aus Herstellerquellen dokumentiert.
- [x] Mögliche legitime Bezugswege (Awin/REWE-Affiliate) ermittelt.
- [ ] Original-APK verfügbar und Signatur/Hash geprüft.
- [ ] App-Binärdatei dekompiliert.
- [ ] Händler- oder Backend-API-Hosts aus APK extrahiert.
- [ ] Live-Netzwerkverhalten beobachtet.
- [ ] Preisdatenquelle und rechtlicher Zugangsweg bestätigt.

**Für die eigentliche APK-Analyse fehlt derzeit die APK-Datei.** Nach Bereitstellung kann die statische Untersuchung anhand überprüfbarer Artefakte beginnen.
