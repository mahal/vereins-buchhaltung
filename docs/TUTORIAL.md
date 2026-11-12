# Tutorial: Vereins-Buchhaltung Schritt für Schritt

Dieses Tutorial führt durch typische Abläufe bei der Verwendung dieser Buchhaltungssoftware.

---

## Vorbereitung

1. Repository klonen oder `Meine_Buchhaltung.html` zusammen mit dem Ordner `kontenplaene/` herunterladen.
2. Die Datei im Browser öffnen (Doppelklick genügt). Für die KMU-Vorlage die Seite über einen Webserver
   öffnen, z. B. `python3 -m http.server` im Ordner und dann <http://localhost:8000/Meine_Buchhaltung.html>
   (siehe A1).
3. Tab **⚙️ Einstellungen** → unter **Name der Organisation** den Namen des Vereins oder der Firma eintragen
   (Standard: «Meine Buchhaltung»). Der Name erscheint in der Kopfzeile, auf Bilanz und Erfolgsrechnung, in
   den Dateinamen der Exporte und im JSON-Backup.
4. Oben rechts das **Geschäftsjahr** wählen.

> **Wo werden die Daten gespeichert?** Im `localStorage` des Browsers – also nur in *diesem* Browser auf
> *diesem* Gerät. Regelmässig im Tab **📝 Journal** über **💾 JSON Backup** sichern. Das Backup enthält auch den Namen der
> Organisation und des Kontenplans; **📤 Restore** stellt beides wieder her.

---

## Kontenplan mit Fremdwährungskonten

### Kontenplan-Vorlage wählen

Zwei Vorlagen werden mitgeliefert (JSON-Dateien im Ordner `kontenplaene/`):

| Vorlage                                   | Datei                      | Jahresergebnis | Kursgewinne / -verluste |
|-------------------------------------------|----------------------------|----------------|-------------------------|
| Schweizer Vereins-Kontenrahmen (Standard) | `kontenplaene/verein.json` | 2900           | 3450 / 6700             |
| Schweizer KMU-Kontenrahmen                | `kontenplaene/kmu.json`    | 2979           | 6992 / 6942             |

1. Tab **⚙️ Einstellungen → Kontenplan-Vorlage**.
2. Vorlage wählen → **Vorlage übernehmen** → bestätigen.

Der Name des aktiven Kontenplans steht in der Kopfzeile. Den Kontenplan siehst du im Tab **📋 Kontenplan**,
gruppiert nach Aktiven, Passiven, Erträgen und Aufwänden.

> **file:// oder Webserver?** Vorlagen werden per `fetch()` nachgeladen. Wird die HTML-Datei direkt per
> Doppelklick geöffnet (`file://`), blockiert der Browser das. Die Vereinsvorlage ist deshalb zusätzlich
> eingebaut und funktioniert immer. Die KMU-Vorlage dann entweder über einen Webserver laden oder die Datei
> `kontenplaene/kmu.json` über **📤 Kontenplan importieren** wählen (siehe A3).

Vorlagen lassen sich in `config.js` anpassen oder ergänzen (eigene JSON-Dateien), ebenso der Name und die
Vorlage für den ersten Start:

```js
window.APP_CONFIG = {
  name: "Musikverein Beispiel",
  defaultVorlage: "kmu",
  kontenplanVorlagen: [
    { id: "verein", label: "Schweizer Vereins-Kontenrahmen", url: "kontenplaene/verein.json" },
    { id: "kmu",    label: "Schweizer KMU-Kontenrahmen",     url: "kontenplaene/kmu.json"    },
    { id: "eigen",  label: "Unser Kontenplan",               url: "kontenplaene/eigen.json"  },
  ],
};
```

`name` und `defaultVorlage` gelten nur für eine **neue** Buchhaltung (leerer Browser-Speicher). Danach
werden Name und Kontenplan unter **⚙️ Einstellungen** geändert.

### Fremdwährungskonto manuell anlegen

1. Tab **📋 Kontenplan** öffnen → **+ Neues Konto**.
2. Felder ausfüllen, z. B. für ein Euro-Bankkonto:

   | Feld        | Wert              |
   |-------------|-------------------|
   | Konto-Nr.   | `1021`            |
   | Bezeichnung | `Bankkonto EUR`   |
   | Kategorie   | `Aktiven`         |
   | Gruppe      | `Umlaufvermögen`  |
   | Währung     | `EUR`             |

3. **Anlegen** klicken.

Regeln, die die Software durchsetzt:

- **Währung nur für Aktiven und Passiven.** Bei Ertrag/Aufwand ist das Feld gesperrt – Erfolgskonten
  werden immer in CHF geführt (Buchwährung CHF, OR 957a Abs. 4).
- Währung als **ISO-Code mit 3 Buchstaben** (EUR, USD, GBP …).
- Die Währung eines Kontos lässt sich später nur ändern, **solange noch keine Buchungen** darauf sind.
- **Kontonummern 10xx** gelten als flüssige Mittel. Nur dort werden bei der Jahresend-Bewertung
  unrealisierte Kursgewinne gebucht (siehe A5). Ein Fremdwährungs-Bankkonto also z. B. als `1021`, `1022` anlegen.

Weitere sinnvolle Konten:

| Konto | Bezeichnung        | Kategorie | Währung |
|-------|--------------------|-----------|---------|
| 1022  | Bankkonto USD      | Aktiven   | USD     |
| 1101  | Debitoren EUR      | Aktiven   | EUR     |
| 2001  | Kreditoren EUR     | Passiven  | EUR     |
| 6800  | Bankspesen (Vereinsvorlage) | Aufwand   | –       |

> **Tipp:** Ein eigenes Konto für Bankspesen hilft der KI beim Kontoauszug-Import, Gebühren korrekt
> zuzuordnen. Die Vereinsvorlage hat keines (z. B. `6800 Bankspesen` anlegen), die KMU-Vorlage hat `6940`.

### Kontenplan importieren und exportieren

Unter **⚙️ Einstellungen → Kontenplan exportieren / importieren**:

- **📥 Export JSON / Export CSV** – speichert den aktuellen Kontenplan als `<Name>_Kontenplan.json` bzw. `.csv`.
- **📤 Kontenplan importieren** – liest einen Kontenplan aus einer Datei ein. Unterstützt werden:
  - Kontenplan-JSON (Format wie der Export bzw. `kontenplaene/*.json`, siehe unten)
  - ein JSON-Backup der Buchhaltung (nur die Konten werden übernommen)
  - CSV mit Kopfzeile, Trennzeichen `;`, `,` oder Tab:

    ```
    Nr;Bezeichnung;Kategorie;Gruppe;Währung
    1000;Kasse;aktiven;Umlaufvermögen;
    1021;Bankkonto EUR;aktiven;Umlaufvermögen;EUR
    3000;Mitgliederbeiträge;ertrag;Vereinserträge;
    ```

    Einen Kontenplan aus Excel also einfach mit diesen Spalten als **CSV (UTF-8)** speichern und importieren.

Beim Import wird jedes Konto geprüft: Nummer und Bezeichnung vorhanden, keine doppelten Nummern, Kategorie
`aktiven`/`passiven`/`ertrag`/`aufwand`, Währung als ISO-Code und nur bei Aktiven/Passiven. Bei einem Fehler
wird **nichts** übernommen und die fehlerhaften Zeilen werden angezeigt.

Vor der Übernahme erscheint eine Bestätigung. Dabei gilt:

- Der bisherige Kontenplan wird ersetzt, **Buchungen bleiben unverändert**.
- Konten **mit Buchungen**, die im neuen Plan fehlen, werden **beibehalten** (und in der Bestätigung genannt).
- Die Währung eines bebuchten Kontos kann nicht geändert werden – der Import wird dann abgelehnt.
- Ändert sich die Kategorie eines bebuchten Kontos, wird gewarnt.

> **Tipp:** Vorher trotzdem ein **💾 JSON Backup** im Journal erstellen.

#### JSON-Format

```json
{
  "format": "kontenplan",
  "version": 1,
  "name": "Mein Kontenplan",
  "resultAccount": "2900",
  "fxAccounts": { "gain": "3450", "loss": "6700" },
  "accounts": [
    { "number": "1000", "name": "Kasse",         "category": "aktiven", "group": "Umlaufvermögen" },
    { "number": "1021", "name": "Bankkonto EUR", "category": "aktiven", "group": "Umlaufvermögen", "currency": "EUR" },
    { "number": "3450", "name": "Kursgewinne (Fremdwährungen)",  "category": "ertrag",  "group": "Finanzerträge" },
    { "number": "6700", "name": "Kursverluste (Fremdwährungen)", "category": "aufwand", "group": "Finanzaufwand" }
  ]
}
```

- `name` – Name des Kontenplans (erscheint in der Kopfzeile und auf der Bilanz).
- `resultAccount` – Kontonummer, unter der die Bilanz das Jahresergebnis ausweist (Vereinsvorlage `2900`,
  KMU-Vorlage `2979`; Standard `2900`).
- `fxAccounts` – Konten für Kursgewinne/-verluste bei der Fremdwährungsbewertung (optional, Standard 3450/6700).
- `currency` weglassen für CHF-Konten.

#### Kontenplan aus PDF oder Word (z. B. vom Treuhänder)

Die Software liest kein PDF oder Word. Am einfachsten den Text aus dem Dokument kopieren und von einem
Sprachmodell (z. B. dem lokalen Modell aus Teil B) in das CSV- oder JSON-Format oben umwandeln lassen:

```bash
curl -s http://localhost:11434/v1/chat/completions -H "Content-Type: application/json" -d '{
  "model": "qwen2.5:7b", "temperature": 0,
  "messages": [{"role":"user","content":"Wandle diesen Kontenplan in CSV um, Kopfzeile Nr;Bezeichnung;Kategorie;Gruppe;Währung. Kategorie ist aktiven, passiven, ertrag oder aufwand. Nur die CSV ausgeben.\n\n<KONTENPLAN-TEXT HIER EINFÜGEN>"}]
}' | python3 -c 'import json,sys; print(json.load(sys.stdin)["choices"][0]["message"]["content"])' > kontenplan.csv
```

Das Ergebnis **vor dem Import kontrollieren** – der Import prüft zwar das Format, aber nicht, ob die
Zuordnung inhaltlich stimmt.

### Mit Fremdwährungskonten buchen

**Eröffnungssaldo:** Die Software kennt keine separate Eröffnungsbilanz. Den Anfangsbestand eines
EUR-Kontos als Buchung per 1.1. erfassen, z. B. Soll `1021` / Haben `2800`.

**Buchung im Journal** (Tab **📝 Journal → Neue Buchung erfassen**):

1. Datum, Buchungstext, Soll- und Haben-Konto wählen – sobald ein Fremdwährungskonto beteiligt ist,
   erscheint ein oranges Feld mit **Betrag EUR** und **Kurs CHF / 1 EUR** (sowie «Kursdifferenz (nur CHF)» für reine Bewertungskorrekturen).
2. Betrag in EUR und Kurs (CHF je 1 EUR) eingeben → der **CHF-Betrag wird berechnet**.
   Umgekehrt: EUR-Betrag und CHF-Betrag eingeben → der Kurs wird berechnet.
3. **+ Buchen**.

Einschränkung: Soll und Haben dürfen nicht zwei *verschiedene* Fremdwährungen haben. Ein Wechsel
EUR → USD wird in zwei Buchungen über ein CHF-Konto erfasst.

### Jahresabschluss: Bewertung zum Stichtagskurs

1. Tab **💱 Fremdwährung** öffnen.
2. Unter **Stichtagskurse per 31.12.** für jede Währung den Kurs eintragen (Mittelkurs, z. B. aus der
   Jahresend-Kursliste der ESTV oder dem Bankauszug).
3. Die Tabelle zeigt pro Konto Saldo in Fremdwährung, Buchwert CHF, Wert zum Stichtagskurs und die Differenz.
   - **Kursverluste** sind immer angehakt (Vorsichtsprinzip, nicht abwählbar).
   - **Kursgewinne** sind nur bei flüssigen Mitteln (Konten `10xx`) angehakt; bei Debitoren/Kreditoren
     sind sie als «unrealisiert – Imparitätsprinzip» markiert und standardmässig nicht gebucht.
   - **Eigenkapitalkonten** (Gruppe «Eigenkapital…», z. B. Privat EUR) werden nicht bewertet und unter
     der Tabelle als «Nicht bewertet (Eigenkapital)» aufgeführt.
4. Gegenkonten prüfen (Standard `3450` / `6700`; fehlen sie, mit **Anlegen** erzeugen).
5. **Bewertungsbuchungen erstellen** → Buchungen per 31.12. mit Beleg `FW-BEW-<Jahr>`.

Die **⚖️ Bilanz** zeigt die Fremdwährungspositionen danach mit Stichtagskurs (für den Anhang).

---

## Lokales Modell und Kontoauszug-Import

### Lokales Modell installieren und starten

Die Buchhaltung spricht jeden **OpenAI-kompatiblen** Server an. Empfohlen wird **Ollama**.

#### Ollama (Linux / macOS / Windows)

1. Installieren: <https://ollama.com/download> (Linux: `curl -fsSL https://ollama.com/install.sh | sh`).
2. Modell herunterladen:

   ```bash
   ollama pull qwen2.5vl:7b     # Vision-Modell: Belegfotos/Scans + Kontoauszüge
   # oder, nur für Kontoauszüge/Text, schneller:
   ollama pull qwen2.5:7b
   ```

3. Server **mit CORS-Freigabe** starten – sonst blockiert der Browser die Anfragen:

   ```bash
   OLLAMA_ORIGINS="*" ollama serve
   ```

   Läuft Ollama bereits als Systemdienst (Linux), stattdessen:

   ```bash
   sudo systemctl edit ollama
   #   [Service]
   #   Environment="OLLAMA_ORIGINS=*"
   sudo systemctl restart ollama
   ```

4. Test:

   ```bash
   curl http://localhost:11434/v1/models
   ```

   Das geladene Modell muss in der Liste erscheinen.

#### Alternativen

| Server     | Start                                                            | `baseUrl`                    | `jsonMode` |
|------------|------------------------------------------------------------------|------------------------------|------------|
| LM Studio  | Developer → Server starten, **«Enable CORS»** einschalten        | `http://localhost:1234/v1`   | `false`    |
| llama.cpp  | `llama-server -m modell.gguf --mmproj mmproj.gguf --port 8080`   | `http://localhost:8080/v1`   | `true`     |

### B2. Buchhaltung mit dem Modell verknüpfen

1. Im Ordner der HTML-Datei die Beispielkonfiguration kopieren:

   ```bash
   cp config.example.js config.js
   ```

2. In `config.js` die Claude-Zeile (`window.ANTHROPIC_API_KEY = …`) auskommentieren und die
   Variante 2 aktivieren:

   ```js
   window.LLM_CONFIG = {
     provider:     "openai",
     baseUrl:      "http://localhost:11434/v1",  // Ollama
     model:        "qwen2.5vl:7b",               // exakt wie bei «ollama pull»
     jsonMode:     true,                         // Ollama: true, LM Studio: false
     jsonSchema:   true,                         // nur Kontonummern aus dem Kontenplan erlaubt
     csvBatchSize: 15,                           // Transaktionen pro KI-Aufruf
   };
   ```

   | Option            | Bedeutung                                                                                           |
   |-------------------|-----------------------------------------------------------------------------------------------------|
   | `jsonSchema`      | Erzwingt gültige Kontonummern (Ollama ≥ 0.5, LM Studio, llama.cpp). Sehr empfohlen.                  |
   | `csvBatchSize`    | Kleiner wählen (z. B. 8), wenn Antworten abgeschnitten werden oder das Modell Zeilen auslässt.       |
   | `maxTokensFactor` | Bei Reasoning-Modellen mit `<think>`-Ausgabe auf z. B. `4` erhöhen.                                 |
   | `apiKey`          | Nur nötig, wenn der Server einen Key verlangt.                                                      |

3. `config.js` liegt im `.gitignore` und wird nicht eingecheckt.
4. HTML-Datei **neu laden**. Im Tab **🤖 Buchungsassistent** steht nun «lokales Modell (qwen2.5vl:7b)».

> **Fehlermeldung «Modell-Server unter … nicht erreichbar»** → Server läuft nicht, falsche `baseUrl`
> oder CORS nicht freigegeben (`OLLAMA_ORIGINS="*"` bzw. «Enable CORS»).

### Kontoauszug bei der Bank exportieren

| Format        | Empfehlung                                                                                                   |
|---------------|--------------------------------------------------------------------------------------------------------------|
| **camt.053**  | ✅ Empfohlen. Im E-Banking unter «Kontoauszug / camt / ISO 20022» als XML exportieren. Enthält IBAN, Gegenpartei, Referenzen, Einzelzahlungen. |
| camt.052/.054 | Ebenfalls möglich (Zwischenauszug / Avis).                                                                     |
| CSV           | Funktioniert mit den meisten Banken (neon, PostFinance, UBS, Raiffeisen, ZKB, Revolut …).                     |

Hinweise:

- **ZIP-Archive vorher entpacken.**
- **Pro Datei nur ein Konto und eine Währung.** Ein EUR-Konto separat exportieren.
- Bei CSV werden Trennzeichen, Kopfzeile (auch nach Vorspann-Zeilen), Datums- und Betragsformate
  automatisch erkannt. Nötig ist mindestens eine Datumsspalte und eine Betragsspalte (`Betrag` oder
  `Belastung`/`Gutschrift`).

### Kontoauszug importieren und buchen

1. Tab **🤖 Buchungsassistent** → Umschalter **🏦 Kontoauszug**.
2. **Kontoauszug hochladen** → `.xml` (camt) oder `.csv` wählen.
3. **Kopfzeile prüfen:** Format, IBAN, Währung, Anzahl Transaktionen, ggf.
   «x Sammelbuchungen aufgeteilt» / «x bereits importiert» / «x aus Verlauf zugeordnet».
4. **Bankkonto im Kontenplan** wählen. Die Software schlägt eines vor (gleiche Währung, Konten `10xx`,
   bei CHF bevorzugt `1020`) und merkt sich die Zuordnung IBAN → Konto für die nächsten Importe.
   Ohne bekannte IBAN wird das zuletzt gewählte Bankkonto vorgeschlagen (sofern die Währung passt).
   Zur Auswahl stehen Aktiv- und Passivkonten – z. B. für eine Kreditkarte oder einen
   Kontokorrentkredit, der als Verbindlichkeit geführt wird.
5. **🔍 Gegenkonten für N vorschlagen** klicken. Das Modell erhält *nur* Gegenpartei, Text, Richtung
   und Betrag und liefert Gegenkonto + Buchungstext. Bei grösseren Auszügen läuft die Analyse in
   Blöcken («Block 2/4»).
6. **Vorschläge prüfen** – die Badges zeigen die Herkunft:
   - **Verlauf** – dieselbe Gegenpartei wurde schon ≥ 2× gleich gebucht, Zuordnung ohne KI.
   - **KI** – Vorschlag des Modells (Maus darüber → Begründung).
   - **manuell** – von dir gewählt.
   - ⚠ – Modell hat die Zeile ausgelassen oder ein ungültiges Konto geliefert → selbst wählen.

   Gegenkonto (Dropdown) und Buchungstext sind editierbar. Unter dem Gegenkonto steht die
   resultierende Buchung (`S 1020 · H 3000`).
7. Zeilen, die nicht gebucht werden sollen, abhaken.
8. **✓ N ausgewählte buchen** → die Buchungen erscheinen im **📝 Journal**.

Was die Software garantiert:

- **Datum und Betrag stammen immer aus der Datei**, nie von der KI.
- **Soll/Haben ergibt sich aus dem Vorzeichen:** Eingang → Bankkonto im Soll, Ausgang → Bankkonto im Haben.
- **Duplikate:** Wird derselbe Auszug (oder ein überlappender) nochmals importiert, sind bereits
  gebuchte Transaktionen als «bereits importiert» markiert und werden nicht doppelt gebucht.
- **Sammelbuchungen** (camt) werden in Einzelzahlungen aufgeteilt, sofern die Bank die Einzelbeträge
  mitliefert. Vorgemerkte (nicht gebuchte) Positionen werden ignoriert.

### Fremdwährungs-Kontoauszug (z. B. EUR) buchen

1. Voraussetzung: Ein EUR-Konto existiert im Kontenplan (Teil A2, z. B. `1021 Bankkonto EUR`).
2. EUR-Auszug hochladen → als Bankkonto `1021 – Bankkonto EUR [EUR]` wählen.
   Wird ein CHF-Konto gewählt, erscheint eine Warnung und das Buchen ist gesperrt.
3. Feld **Kurs CHF je 1 EUR** ausfüllen. Es ist mit dem Stichtagskurs des Geschäftsjahres
   (Tab **💱 Fremdwährung**) vorbelegt, falls dort schon einer erfasst ist.
4. Gegenkonten vorschlagen lassen, prüfen, buchen. Für jede Zeile gilt:
   CHF-Betrag = EUR-Betrag × Kurs; der EUR-Betrag wird auf dem Konto mitgeführt.
5. Abweichende Tageskurse können danach im **📝 Journal** pro Buchung angepasst werden (✏).
6. Am Jahresende die Bewertung gemäss **A5** durchführen.

Sonderfall: Eine **CHF-Zahlung auf ein EUR-Gegenkonto** (z. B. Begleichung einer EUR-Kreditorenrechnung
vom CHF-Konto) übernimmt den EUR-Originalbetrag automatisch, sofern der camt-Auszug ihn enthält
(`InstdAmt`). Bei CSV muss er im Journal nachgetragen werden.

### Tipps für gute Ergebnisse mit lokalen Modellen

- **Sprechende Kontonamen** verwenden («Mitgliederbeiträge», «Bankspesen», «Miete Probelokal») –
  das Modell sieht nur Nummer, Name und Kategorie.
- Die ersten Importe sorgfältig korrigieren: Ab der zweiten gleichen Buchung pro Gegenpartei greift
  der **Verlauf**, und die KI wird kaum noch gebraucht.
- `jsonSchema: true` verhindert erfundene Kontonummern.
- Bei «Antwort abgeschnitten (max_tokens erreicht)» `csvBatchSize` verkleinern oder `maxTokensFactor` erhöhen.
- Bei fehlerhaftem JSON ein grösseres Modell versuchen (z. B. `qwen2.5:14b`) oder `jsonMode` umstellen.

---

## Checkliste Jahresablauf

1. Name setzen, Kontenplan-Vorlage wählen oder importieren, Fremdwährungskonten anlegen (A1–A3)
2. Eröffnungssalden per 1.1. buchen (A4)
3. Laufend: Kontoauszüge importieren (B4/B5), Einzelbelege über **📎 Einzelbeleg**
4. Regelmässig **💾 JSON Backup**
5. Per 31.12.: Stichtagskurse erfassen und Bewertungsbuchungen erstellen (A5)
6. **⚖️ Bilanz** und **📊 Erfolgsrechnung** prüfen, als Word exportieren oder drucken
