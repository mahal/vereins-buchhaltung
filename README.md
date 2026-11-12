# vereins-buchhaltung
Eine doppelte Buchhaltung mit Kontenrahmen für Schweizer Vereine. Konten können einfach angepasst werden, Buchhaltungsassistent mit Claude für Belegsverarbeitung und CSV Import vom Kontoauszug, beispielsweise Neon Hypi Lenzburg. Pull-Requests willkommen. Super schnell, läuft alles im Browser und speichert in local storage. Backup und Import mit JSON. Einfachstes Deployment: die HTML-Datei im Browser öffnen. 

Eine Schritt-für-Schritt-Anleitung (Kontenplan mit Fremdwährungen, lokales Modell, camt/CSV-Import) steht in [docs/TUTORIAL.md](docs/TUTORIAL.md).

Auf der Website https://claude-buha.werckmeister.app/Werckmeister_Buchhaltung.html läuft eine Instanz. 

Für die KI-Belegsverarbeitung `config.example.js` duplizieren nach `config.js` – dann den Claude-API-Key eintragen. Ist im .gitignore drin.

## Name und Kontenplan

Unter **⚙️ Einstellungen** werden der Name des Vereins oder der Firma (Standard «Meine Buchhaltung») und der Kontenplan verwaltet. Der Name erscheint in Kopfzeile, Bilanz, Erfolgsrechnung, Exporten und im JSON-Backup.

- **Vorlagen:** Schweizer Vereins-Kontenrahmen (Standard) und Schweizer KMU-Kontenrahmen als JSON-Dateien in `kontenplaene/`. Eigene Vorlagen und die Startvorlage lassen sich in `config.js` über `window.APP_CONFIG` festlegen (siehe `config.example.js`). Vorlagen werden per `fetch()` geladen – beim Öffnen als `file://` ist nur die eingebaute Vereinsvorlage verfügbar; andere Vorlagen dann importieren oder die Seite über einen Webserver öffnen.
- **Import/Export:** Kontenplan als JSON oder CSV (`Nr;Bezeichnung;Kategorie;Gruppe;Währung`). Konten werden beim Import geprüft; Konten mit Buchungen bleiben erhalten.

## Lokales Modell statt Claude

Der Buchungsassistent kann statt Claude auch ein lokales Modell über einen OpenAI-kompatiblen Server verwenden (Ollama, LM Studio, llama.cpp `llama-server`, vLLM). In `config.js`:

```js
window.LLM_CONFIG = {
  provider: "openai",
  baseUrl:  "http://localhost:11434/v1",   // Ollama
  model:    "qwen2.5vl:7b",
  jsonMode: true,
  csvBatchSize: 15,
};
```

- **CORS:** Der Server muss Anfragen aus dem Browser erlauben – Ollama mit `OLLAMA_ORIGINS="*" ollama serve`, LM Studio mit «Enable CORS».
- **Belege:** Für Fotos/Scans ein Vision-Modell verwenden. PDFs werden im Browser per pdf.js in Text umgewandelt; Scans ohne Textebene werden als Bild (max. 3 Seiten) gesendet.
- **Kontoauszug:** siehe unten. Die Transaktionen werden in Blöcken (`csvBatchSize`) analysiert; mit `jsonSchema: true` darf das Modell nur Kontonummern aus dem Kontenplan liefern.
- Weitere Optionen siehe `config.example.js`.


## Kontoauszug-Import (camt.053 / CSV)

Im Buchungsassistenten unter «🏦 Kontoauszug»:

- **Formate:** camt.053 (empfohlen), camt.052, camt.054 (ISO 20022 XML) sowie CSV der meisten Banken. Bei CSV werden Trennzeichen, Kopfzeile (auch nach Vorspann-Zeilen), Datums-/Betragsformate und Spalten wie «Betrag» oder «Belastung»/«Gutschrift» automatisch erkannt. ZIP-Archive vorher entpacken.
- **Datum und Betrag kommen immer aus der Datei.** Die KI wählt nur das Gegenkonto und einen Buchungstext; Soll/Haben ergibt sich aus dem Vorzeichen (Eingang: Bankkonto im Soll).
- **Sammelbuchungen** in camt werden in Einzelzahlungen aufgeteilt, sofern die Bank die Einzelbeträge mitliefert. Vorgemerkte Positionen werden ignoriert.
- **Lernen aus dem Verlauf:** Importierte Buchungen speichern Gegenpartei und IBAN. Wurde dieselbe Gegenpartei mindestens zweimal gleich gebucht, wird sie ohne KI zugeordnet (Badge «Verlauf»); sonst erhält das Modell die früheren Konten als Hinweis.
- **Duplikate:** Bereits importierte Transaktionen werden erkannt und nicht nochmals gebucht.
- **Fremdwährung:** Ein EUR-Auszug wird auf ein EUR-Konto gebucht; der CHF-Betrag ergibt sich aus dem eingegebenen Kurs (vorbelegt mit dem Stichtagskurs des Jahres).

## Fremdwährungskonten

Buchwährung ist CHF (OR 957a Abs. 4). Aktiv- und Passivkonten können im Kontenplan eine Währung erhalten (z. B. `1021 Bankkonto EUR`). Buchungen auf solche Konten werden mit Betrag in Fremdwährung und Transaktionskurs erfasst; der CHF-Betrag wird daraus berechnet. Im Tab **Fremdwährung** werden die Stichtagskurse per 31.12. erfasst und die Bewertungsbuchungen erstellt (Kursverluste immer, unrealisierte Kursgewinne nur auf flüssigen Mitteln/Wertschriften 10xx – Imparitätsprinzip). Gegenkonten: `3450 Kursgewinne` / `6700 Kursverluste`. Die Bilanz zeigt die Fremdwährungspositionen mit Stichtagskurs für den Anhang.
