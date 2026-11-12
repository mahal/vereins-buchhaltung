# Viele Buchungen auf einmal löschen oder bearbeiten

Im Tab **📝 Journal** lassen sich Buchungen nur einzeln löschen (🗑 bzw. × und Bestätigung mit **Ja**).
Eine Funktion zum Löschen mehrerer Buchungen gibt es (noch) nicht. Diese Anleitung zeigt zwei Wege, um
trotzdem viele Buchungen in einem Schritt zu entfernen. Am Ende folgt ein Beispiel für die
Massenbearbeitung (Soll und Haben in einem Belegnummer-Bereich tauschen).

> **Zuerst ein Backup erstellen!** Im Tab **📝 Journal** auf **💾 JSON Backup** klicken. Falls etwas
> schiefgeht, lässt sich der alte Stand mit **📤 Restore** wiederherstellen.

---

## Variante A: JSON-Backup bearbeiten (ohne Programmierung)

Geeignet, wenn die zu löschenden Buchungen von Hand ausgewählt werden.

1. Im Tab **📝 Journal** auf **💾 JSON Backup** klicken. Es wird eine Datei
   `…_Backup_JJJJ-MM-TT.json` heruntergeladen.
2. Die Datei in einem Texteditor öffnen. Die Buchungen stehen im Array `"entries": [ … ]`, jede Buchung
   ist ein Objekt `{ … }` mit Feldern wie `date`, `reference`, `description`, `debit`, `credit`,
   `amount`.
3. Die nicht mehr gewünschten Objekte löschen. Dabei auf die Kommas achten: Zwischen zwei Objekten steht
   genau ein Komma, nach dem letzten Objekt keines. Sonst ist die Datei kein gültiges JSON mehr.
4. Im Tab **📝 Journal** auf **📤 Restore** klicken und die bearbeitete Datei wählen.

**Wichtig:** Restore **ersetzt** alle Buchungen durch den Inhalt der Datei. Was nicht in der Datei steht,
ist danach weg. Das gilt auch für Kontenplan und Fremdwährungskurse, die ebenfalls aus der Datei
übernommen werden.

---

## Variante B: Browser-Konsole (Löschen nach Regel)

Geeignet, wenn die Buchungen einem Muster folgen, z. B. alle Buchungen eines Zeitraums mit einem
bestimmten Buchungstext oder alle Buchungen eines fehlerhaften Bank-Imports.

### Hintergrund

Die Software speichert alle Daten im Browser (`localStorage`) unter dem Schlüssel
`werckmeister_buch_v1` und liest sie nur beim Öffnen der Seite ein. Man kann die gespeicherten Daten
also direkt filtern und die Seite danach neu laden.

### Schritt 1: Auswahl prüfen

1. `Meine_Buchhaltung.html` im Browser öffnen.
2. Mit **F12** die Entwicklerwerkzeuge öffnen und den Tab **Konsole** (Console) wählen.
3. Folgenden Code einfügen und die Funktion `toDelete` an die eigenen Bedürfnisse anpassen:

```js
const KEY = "werckmeister_buch_v1";
const d = JSON.parse(localStorage.getItem(KEY));
const before = d.entries.length;

// true zurückgeben für Buchungen, die GELÖSCHT werden sollen, z. B.:
const toDelete = e =>
  e.date >= "2026-01-01" && e.date <= "2026-03-31" &&   // Zeitraum
  /PayPal/i.test(e.description);                         // und Buchungstext enthält «PayPal»
// weitere Felder: e.reference (Belegnr.), e.debit (Soll-Konto), e.credit (Haben-Konto), e.amount (Betrag)

const doomed = d.entries.filter(toDelete);
console.table(doomed.map(e => ({datum:e.date, beleg:e.reference, text:e.description, betrag:e.amount})));
```

4. Die angezeigte Tabelle kontrollieren: Sie listet genau die Buchungen, die gelöscht würden.
   Bis hierhin wurde noch nichts verändert.

### Schritt 2: Löschen

Stimmt die Auswahl, diesen Code ausführen:

```js
d.entries = d.entries.filter(e => !toDelete(e));
localStorage.setItem(KEY, JSON.stringify(d));
console.log(`${before - d.entries.length} Buchungen gelöscht`);
location.reload();
```

Die Seite lädt neu und zeigt das Journal ohne die gelöschten Buchungen.

**Wichtig:** Zwischen Schritt 1 und Schritt 2 nichts in der Software anklicken oder ändern. Die Software
speichert jede Änderung automatisch und würde dabei die Bearbeitung in der Konsole überschreiben.

### Beispiele für `toDelete`

```js
// Alle Buchungen eines Jahres
const toDelete = e => e.date.startsWith("2025-");

// Alle Buchungen auf ein bestimmtes Konto (Soll oder Haben)
const toDelete = e => String(e.debit) === "1020" || String(e.credit) === "1020";

// Belegnummern in einem Bereich (z. B. 2026-100 bis 2026-199)
const toDelete = e => /^2026-1\d\d$/.test(e.reference || "");
```

### Beispiel: Bereich von Belegnummern löschen

Die automatisch vergebenen Belegnummern haben die Form `JJJJ-NNN` (z. B. `2026-001`). Das folgende
Skript löscht alle Buchungen von einer Belegnummer bis zu einer anderen, **beide eingeschlossen**.
Belegnummern in einem anderen Format (z. B. von Hand erfasste wie `R-17`) werden nie gelöscht.

Schritt 1: Bereich festlegen und Auswahl prüfen (es wird noch nichts verändert):

```js
const KEY = "werckmeister_buch_v1";
const VON = "2026-045";   // erste zu löschende Belegnummer
const BIS = "2026-112";   // letzte zu löschende Belegnummer

const parse = r => {
  const m = /^(\d{4})-(\d+)$/.exec(String(r || "").trim());
  return m ? { jahr: m[1], nr: parseInt(m[2], 10) } : null;
};
const von = parse(VON), bis = parse(BIS);
if (!von || !bis || von.jahr !== bis.jahr || von.nr > bis.nr)
  throw new Error("Ungültiger Bereich: VON und BIS müssen die Form JJJJ-NNN haben, im selben Jahr liegen und VON ≤ BIS sein.");

const toDelete = e => {
  const b = parse(e.reference);
  return !!b && b.jahr === von.jahr && b.nr >= von.nr && b.nr <= bis.nr;
};

const d = JSON.parse(localStorage.getItem(KEY));
const before = d.entries.length;
const doomed = d.entries.filter(toDelete);
console.table(doomed.map(e => ({beleg:e.reference, datum:e.date, text:e.description, betrag:e.amount})));
console.log(`${doomed.length} Buchungen im Bereich ${VON} bis ${BIS} gefunden`);
```

Schritt 2: Stimmt die Liste, löschen und neu laden:

```js
d.entries = d.entries.filter(e => !toDelete(e));
localStorage.setItem(KEY, JSON.stringify(d));
console.log(`${before - d.entries.length} Buchungen gelöscht`);
location.reload();
```

Der Vergleich erfolgt über die Zahl, nicht über den Text. Darum gehören z. B. `2026-45` und
`2026-045` beide in den Bereich ab `2026-045`.

---

## Massenbearbeitung: Soll und Haben in einem Belegnummer-Bereich tauschen

Wurden mehrere Buchungen mit vertauschten Konten erfasst (z. B. ein ganzer Bank-Import mit falscher
Richtung), lassen sich Soll- und Haben-Konto für einen Bereich von Belegnummern in einem Schritt tauschen.
Es gilt dasselbe Vorgehen wie bei Variante B. Vorher also ein **💾 JSON Backup** erstellen und zwischen
den beiden Schritten nichts in der Software anklicken.

Betrag, Datum, Text und Belegnummer bleiben unverändert. Auch Fremdwährungsangaben (Betrag in
Fremdwährung, Kurs) bleiben gültig, denn sie hängen nicht davon ab, auf welcher Seite das
Fremdwährungskonto steht.

Schritt 1: Bereich festlegen und Auswahl prüfen (es wird noch nichts verändert):

```js
const KEY = "werckmeister_buch_v1";
const VON = "2026-045";   // erste zu ändernde Belegnummer
const BIS = "2026-112";   // letzte zu ändernde Belegnummer

const parse = r => {
  const m = /^(\d{4})-(\d+)$/.exec(String(r || "").trim());
  return m ? { jahr: m[1], nr: parseInt(m[2], 10) } : null;
};
const von = parse(VON), bis = parse(BIS);
if (!von || !bis || von.jahr !== bis.jahr || von.nr > bis.nr)
  throw new Error("Ungültiger Bereich: VON und BIS müssen die Form JJJJ-NNN haben, im selben Jahr liegen und VON ≤ BIS sein.");

const toSwap = e => {
  const b = parse(e.reference);
  return !!b && b.jahr === von.jahr && b.nr >= von.nr && b.nr <= bis.nr;
  // optional nur bestimmte Konten, z. B. zusätzlich: && (e.debit === "1020" || e.credit === "1020")
};

const d = JSON.parse(localStorage.getItem(KEY));
const betroffen = d.entries.filter(toSwap);
console.table(betroffen.map(e => ({
  beleg:e.reference, datum:e.date, text:e.description, betrag:e.amount,
  soll_alt:e.debit, haben_alt:e.credit, soll_neu:e.credit, haben_neu:e.debit })));
console.log(`${betroffen.length} Buchungen im Bereich ${VON} bis ${BIS} gefunden`);
```

Schritt 2: Stimmt die Liste, tauschen, speichern und neu laden:

```js
let n = 0;
d.entries = d.entries.map(e => {
  if (!toSwap(e)) return e;
  n++;
  return { ...e, debit: e.credit, credit: e.debit };
});
localStorage.setItem(KEY, JSON.stringify(d));
console.log(`Soll/Haben bei ${n} Buchungen getauscht`);
location.reload();
```

**Achtung:** Das Skript nicht zweimal ausführen. Ein zweiter Durchlauf tauscht die Konten wieder zurück.
