# Eröffnungsbuchungen erstellen

Dieses Tutorial zeigt, wie zu Beginn eines Geschäftsjahres die Anfangsbestände (Eröffnungsbilanz)
erfasst werden.

## Grundsatz

Die Software kennt **keine separate Eröffnungsbilanz**. Bilanz und Erfolgsrechnung werden pro
Geschäftsjahr ausschliesslich aus den Buchungen dieses Jahres berechnet – Schlusssalden des Vorjahres
werden **nicht automatisch übernommen**.

Die Eröffnung wird darum in jedem Jahr als gewöhnliche Buchungen per **1.1.** im Tab
**📝 Journal → Neue Buchung erfassen** erfasst:

- **Pro Bilanzkonto mit Saldo eine Buchung.** Jede Buchung hat genau ein Soll- und ein Haben-Konto;
  Sammelbuchungen (ein Soll an mehrere Haben) sind nicht möglich.
- **Gegenkonto ist immer das Eigenkapital:**
  - Vereins-Kontenplan: `2800` Vereinsvermögen / Eigenkapital
  - KMU-Kontenplan: `2970` Gewinnvortrag / Verlustvortrag (bzw. das passende Eigenkapitalkonto)
- **Erfolgskonten** (Ertrag und Aufwand, Klassen 3–8) werden **nicht** eröffnet – sie beginnen jedes
  Jahr bei null.

| Kontoart | Soll | Haben |
|---|---|---|
| Aktivkonto (z. B. `1020` Bank) | Aktivkonto | `2800` |
| Passivkonto (z. B. `2000` Kreditoren) | `2800` | Passivkonto |

**Tipp:** Als Beleg z. B. `EB-2026` erfassen. So lassen sich die Eröffnungsbuchungen im Journal leicht
filtern und kontrollieren.

## Schritt für Schritt

1. **Schlussbilanz des Vorjahres bereitlegen** – im Programm Jahr auf das Vorjahr stellen und den Tab
   **⚖️ Bilanz** öffnen (oder die gedruckte / exportierte Bilanz verwenden).
2. **Jahr wechseln** auf das neue Geschäftsjahr.
3. Im **📝 Journal** für jedes Aktivkonto mit Saldo buchen: Datum `01.01.`, Soll = Aktivkonto,
   Haben = `2800`, Betrag = Schlusssaldo.
4. Für jedes Passivkonto (Fremdkapital, Reserven) mit Saldo buchen: Soll = `2800`, Haben = Passivkonto.
5. **Fremdwährungskonten** gemäss Abschnitt unten eröffnen.
6. **Kontrolle** im Tab **⚖️ Bilanz** (siehe unten).

## Beispiel: Verein per 1.1.2026

Schlussbilanz 31.12.2025:

| Aktiven | CHF | Passiven | CHF |
|---|---:|---|---:|
| 1000 Kasse | 350.00 | 2000 Kreditoren | 1'150.00 |
| 1020 Bankkonto | 12'400.00 | 2800 Vereinsvermögen | 12'200.00 |
| 1100 Debitoren | 600.00 | | |
| **Total** | **13'350.00** | **Total** | **13'350.00** |

Eröffnungsbuchungen:

| Datum | Beleg | Text | Soll | Haben | CHF |
|---|---|---|---|---|---:|
| 01.01.2026 | EB-2026 | Eröffnung Kasse | 1000 | 2800 | 350.00 |
| 01.01.2026 | EB-2026 | Eröffnung Bankkonto | 1020 | 2800 | 12'400.00 |
| 01.01.2026 | EB-2026 | Eröffnung Debitoren | 1100 | 2800 | 600.00 |
| 01.01.2026 | EB-2026 | Eröffnung Kreditoren | 2800 | 2000 | 1'150.00 |

Danach weist `2800` einen Saldo von **CHF 12'200.00** aus – das Vereinsvermögen (Aktiven minus
Fremdkapital).

## Gewinn oder Verlust des Vorjahres

Das Jahresergebnis wird in der Bilanz unter dem Ergebniskonto des Kontenplans ausgewiesen
(Verein `2900`, KMU `2979`). Im neuen Jahr gehört es ins Eigenkapital:

- **Einfachste Variante:** Alle Bilanzkonten gegen `2800` eröffnen. Das Vorjahresergebnis ist dann
  automatisch im Saldo von `2800` enthalten.
- **Getrennter Ausweis** (z. B. KMU): Zuerst alle Bilanzkonten gegen `2800` eröffnen, danach das
  Vorjahresergebnis auf den Gewinn-/Verlustvortrag umbuchen:
  - Gewinn CHF 3'000: Soll `2800` / Haben `2970`, CHF 3'000.00
  - Verlust CHF 3'000: Soll `2970` / Haben `2800`, CHF 3'000.00

## Reserven und weitere Eigenkapitalkonten

Hat ein weiteres Eigenkapitalkonto einen Saldo (z. B. `2850` Reserven), wird es wie ein Passivkonto
eröffnet: Soll `2800` / Haben `2850`.

## Fremdwährungskonten

Fremdwährungskonten werden mit dem **Betrag in Fremdwährung** und dem **Stichtagskurs per 31.12. des
Vorjahres** eröffnet (derselbe Kurs wie bei der Bewertung im Tab **💱 Fremdwährung**). Nur so stimmen
Saldo in Fremdwährung und Buchwert in CHF überein.

Beispiel EUR-Konto `1021` mit EUR 5'000.00, Stichtagskurs 0.94:

1. Datum `01.01.2026`, Text «Eröffnung Bank EUR», Soll `1021`, Haben `2800`.
2. Im orangen Feld **Betrag EUR** `5000.00` und **Kurs CHF / 1 EUR** `0.94` eingeben.
3. Der CHF-Betrag wird berechnet: **CHF 4'700.00**.
4. **+ Buchen**.

## Kontrolle

- Im Tab **⚖️ Bilanz** müssen nach den Eröffnungsbuchungen **Total Aktiven = Total Passiven** sein und
  jedes Konto denselben Saldo zeigen wie in der Schlussbilanz des Vorjahres.
- Erscheint die Warnung **«Die Bilanz ist um CHF … nicht ausgeglichen. Prüfe die
  Eröffnungsbuchungen.»**, fehlt eine Eröffnungsbuchung oder ein Betrag wurde auf der falschen Seite
  (Soll/Haben vertauscht) erfasst.
- Im Tab **💱 Fremdwährung** muss bei jedem Fremdwährungskonto der Saldo in Fremdwährung mit dem
  Bankauszug per 1.1. übereinstimmen.

## Häufige Fehler

| Problem | Ursache | Lösung |
|---|---|---|
| Bilanz nicht ausgeglichen | Eröffnungsbuchung vergessen | Schlussbilanz Vorjahr Konto für Konto vergleichen |
| Passivkonto mit negativem Saldo | Soll und Haben vertauscht | Buchung löschen und neu erfassen (Soll `2800` / Haben Passivkonto) |
| Ertrag/Aufwand hat schon am 1.1. einen Saldo | Erfolgskonto eröffnet | Buchung löschen – Erfolgskonten werden nicht eröffnet |
| FW-Saldo stimmt, CHF-Saldo weicht ab | falscher Kurs bei der Eröffnung | Stichtagskurs 31.12. des Vorjahres verwenden |
