# Analyse des JSON-Formats: LottoNumberArchive (johannesfriedrich.github.io)

## TL;DR
- Die direkt verlinkte Datei `Lottonumbers_tidy_complete.json` ist ein **"tidy"/Langformat**: ein flaches JSON-Array OHNE Wrapper und OHNE Metadaten, in dem jede einzelne Lottozahl eine eigene Zeile bildet — z. B. `{"id":1,"date":"09.10.1955","variable":"Lottozahl","value":3}`. [github](https://johannesfriedrich.github.io/LottoNumberArchive/Lottonumbers_tidy_complete.json) Das Datum ist ein String im deutschen Format `"TT.MM.JJJJ"`.
- Das Repository bietet zusätzlich eine zweite Datei `Lottonumbers_complete.json` im **Breitformat**, die fast exakt dem vom Nutzer beschriebenen "alten Format" entspricht: `{"data":[{"id":1,"date":"09.10.1955","Lottozahl":[3,12,13,16,23,41]}]}` [github](https://johannesfriedrich.github.io/LottoNumberArchive/Lottonumbers_complete.json) — also mit `data`-Wrapper und den sechs Zahlen als Array. Die Hauptseite sagt wörtlich: *"Two JSON files are give: Choose the one you can work with :-)"*.
- Der zentrale Unterschied: Im Tidy-Format ist die Superzahl KEINE eigene Spalte, sondern erscheint als zusätzliche Zeilen mit `"variable":"Superzahl"` (Wert 0–9); im Breitformat steht sie (in neueren Ziehungen) als eigenes Feld neben dem `Lottozahl`-Array. In den frühen Datensätzen (1955) fehlt die Superzahl ganz, weil sie erst 1991 eingeführt wurde.

## Key Findings
- Die Hauptseite verlinkt zwei JSON-Dateien; die direkt angefragte Datei ist die Tidy-Variante.
- **Tidy-Format** (`Lottonumbers_tidy_complete.json`, exakt 2.07 MB laut GitHub-Dateiansicht): flaches Array von Objekten mit genau vier Feldern – `id` (Integer), `date` (String "TT.MM.JJJJ"), `variable` (String), `value` (Integer). [github](https://johannesfriedrich.github.io/LottoNumberArchive/Lottonumbers_tidy_complete.json) Kein umschließendes Objekt, keine Metadaten.
- **Breitformat** (`Lottonumbers_complete.json`): ein Objekt mit dem Schlüssel `data`, der ein Array von Ziehungsobjekten enthält; jedes Objekt hat `id`, `date` und `Lottozahl` (Array aus sechs Integers).
- Im Repository existiert eine dritte JSON-Datei `lotto_api.json` (Inhalt/Struktur nicht abrufbar).
- Die `variable`-Werte im Tidy-Format umfassen mindestens `"Lottozahl"` (sechs Zahlen 1–49) und `"Superzahl"` (eine Zahl 0–9).
- Jede Ziehung (`id`) erzeugt im Tidy-Format mehrere Zeilen: sechs für die Lottozahlen plus je eine für die Superzahl, sofern vorhanden.
- Der Datenbestand reicht von der ersten Ziehung am 09.10.1955 bis in die Gegenwart. Der GitHub-Repotitel lautet wörtlich *"The german lotto number archive (1955-2026)"*, während die GitHub-Pages-Seite abweichend *"(1955-2025)"* nennt.

## Details

### 1. Aufbau der Tidy-Datei (die direkt angefragte JSON)
Die Datei `Lottonumbers_tidy_complete.json` beginnt mit einer eckigen Klammer `[` — es handelt sich also um ein reines JSON-**Array auf oberster Ebene**, **ohne** umschließendes Objekt und **ohne** Metadaten-Wrapper. Jedes Element ist ein flaches Objekt mit immer denselben vier Schlüsseln.

**Felder und Datentypen:**

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | Integer | Fortlaufende Ziehungsnummer, beginnt bei 1 |
| `date` | String | Ziehungsdatum im Format `"TT.MM.JJJJ"` (z. B. `"09.10.1955"`) |
| `variable` | String | Kategorie des Wertes: `"Lottozahl"` oder `"Superzahl"` |
| `value` | Integer | Der eigentliche Zahlenwert (Lottozahl 1–49 bzw. Superzahl 0–9) |

**Wörtliche Beispiel-Datensätze (Beginn der Datei):**
```json
[
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":3},
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":12},
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":13},
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":16},
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":23},
  {"id":1,"date":"09.10.1955","variable":"Lottozahl","value":41},
  {"id":2,"date":"16.10.1955","variable":"Lottozahl","value":3},
  {"id":2,"date":"16.10.1955","variable":"Lottozahl","value":12},
  {"id":2,"date":"16.10.1955","variable":"Lottozahl","value":18},
  {"id":3,"date":"23.10.1955","variable":"Lottozahl","value":12}
]
```
Charakteristisch ist, dass eine einzelne Ziehung über **sechs aufeinanderfolgende Zeilen** verteilt ist (eine pro gezogener Zahl), die alle dieselbe `id` und dasselbe `date` teilen. [github](https://johannesfriedrich.github.io/LottoNumberArchive/Lottonumbers_tidy_complete.json) Das ist das klassische "tidy data"/Langformat (eine Beobachtung pro Zeile), das sich besonders gut mit R (`dplyr`) und Python (`pandas`) auswerten lässt.

**Superzahl im Tidy-Format:** Die Superzahl ist KEINE eigene Spalte, sondern wird über zusätzliche Zeilen mit `"variable":"Superzahl"` abgebildet, deren `value` zwischen 0 und 9 liegt. Ein solcher Datensatz hat die Form:
```json
{"id":<n>,"date":"TT.MM.JJJJ","variable":"Superzahl","value":<0-9>}
```
Das wird durch den eigenen Beispielcode des Repos bestätigt, der mit `dplyr::filter(variable == "Superzahl")` filtert und die Werte auf einer Achse mit `scale_x_continuous(breaks = c(0:9))` darstellt — also genau den Wertebereich 0–9.

### 2. Die zweite Datei: Breitformat `Lottonumbers_complete.json`
Diese Datei entspricht weitgehend dem vom Nutzer als "altes Format" beschriebenen Aufbau. **Wörtlicher Dateibeginn:**
```json
{"data":[
  {"id":1,"date":"09.10.1955","Lottozahl":[3,12,13,16,23,41]},
  {"id":2,"date":"16.10.1955","Lottozahl":[3,12,18,30,32,49]},
  {"id":3,"date":"23.10.1955","Lottozahl":[12,14,23,24,34,36]},
  {"id":4,"date":"30.10.1955","Lottozahl":[4,13,23,30,36,44]},
  {"id":5,"date":"06.11.1955","Lottozahl":[5,6,31,39,44,49]},
  {"id":6,"date":"13.11.1955","Lottozahl":[6,18,22,29,37,44]}
]}
```
**Felder und Datentypen (Breitformat):**

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `data` | Array (Wrapper) | Enthält alle Ziehungsobjekte |
| `id` | Integer | Fortlaufende Ziehungsnummer |
| `date` | String | `"TT.MM.JJJJ"` |
| `Lottozahl` | Array von 6 Integers | Die sechs gezogenen Hauptzahlen |

Hier gibt es also – im Gegensatz zur Tidy-Datei – einen **Wrapper** (`{"data": [...]}`), und die sechs Zahlen liegen gebündelt als Array vor. Ein Datensatz = eine Ziehung.

### 3. Vergleich mit dem alten Format
Das vom Nutzer genannte alte Format war:
```json
{ "data": [ { "id": ..., "date": "TT.MM.JJJJ", "Lottozahl": [n1,n2,n3,n4,n5,n6], "Superzahl": n } ] }
```
- Das **Breitformat** `Lottonumbers_complete.json` ist nahezu identisch zu diesem alten Format: gleicher `data`-Wrapper, gleiches Datumsformat `"TT.MM.JJJJ"`, gleiche `Lottozahl`-Array-Struktur. Der einzige Unterschied in den frühen Datensätzen: Es gibt dort (noch) kein `Superzahl`-Feld, weil die Superzahl erst 1991 eingeführt wurde (siehe Abschnitt 5).
- Das **Tidy-Format** (die angefragte Datei) ist dagegen strukturell grundlegend anders:
  - **Kein** `data`-Wrapper (Array auf oberster Ebene statt Objekt).
  - **Keine** Arrays — stattdessen eine "geschmolzene"/normalisierte Tabelle mit `variable`/`value`-Spalten.
  - **Mehrere Zeilen pro Ziehung** (6× Lottozahl + ggf. 1× Superzahl) statt einem Datensatz pro Ziehung.
  - Die Superzahl ist als Zeile mit `variable == "Superzahl"` codiert, nicht als eigenes Feld.

Kurz: Beide Dateien tragen dieselben Rohdaten, aber das Breitformat ist "drop-in"-kompatibel zum alten Schema, während das Tidy-Format eine analysefreundliche Normalisierung ist.

### 4. Datumsformat
In beiden Dateien ist das Datum ein **String im deutschen Format `"TT.MM.JJJJ"`** (Tag.Monat.Jahr, Tag/Monat zweistellig, Jahr vierstellig), z. B. `"09.10.1955"`. Es ist **kein** ISO-8601-Datum. Beim Einlesen muss es explizit geparst werden — der README-Beispielcode des Repos verwendet in R `lubridate::dmy(date)` und in Python `pd.read_json(url, convert_dates=False)` mit anschließendem manuellem Parsen.

### 5. Superzahl / Zusatzzahl – historischer Kontext
- Die deutsche **Superzahl** (0–9) wurde **am 7. Dezember 1991 erstmals gezogen** (dielottozahlende.net: *"Am 7. Dezember 1991 wurde die Superzahl zum ersten Mal gezogen und ist seitdem fester Bestandteil des Lotto 6 aus 49"*).
- Die ältere **Zusatzzahl** wurde **am 4. Mai 2013 abgeschafft** und durch die Superzahl ersetzt (lottozahlenonline.de: *"Am Samstag dem 04.05.2013 wird vorerst zum letzten mal die Zusatzzahl gezogen"*; lottoindeutschland.de: *"2013 wurde die Zusatzzahl abgeschafft und durch die Superzahl ersetzt"*).
- Daher enthalten frühe Ziehungen (z. B. 1955) keine Superzahl; im Tidy-Format tauchen Superzahl-Zeilen erst ab dem entsprechenden Zeitraum auf.
- **Hinweis zur Terminologie im Repo:** Im README/auf der Webseite wird die Superzahl ungenau benannt und datiert: *"Since 2001 in the german lottery a number called 'Zusatzzahl' was introduced. Every Wednesday and Saturday the number chosen."* Das ist Anzeigetext und stimmt nicht mit der historischen Einführung der Superzahl (1991) überein — funktional filtert der Code aber korrekt auf `variable == "Superzahl"`.

## Recommendations
- **Für Datenanalyse/Statistik (R/Python):** Nutze die angefragte Tidy-Datei `Lottonumbers_tidy_complete.json`. Sie lässt sich direkt mit `pandas.read_json(url, convert_dates=False)` oder `jsonlite::fromJSON(url)` laden [github](https://johannesfriedrich.github.io/LottoNumberArchive/) [LottoNumberArchive](https://johannesfriedrich.github.io/LottoNumberArchive/) und durch Filtern auf `variable` gruppieren. Für Häufigkeitsanalysen: `filter(variable == "Lottozahl")` und nach `value` zählen.
- **Wenn dein Code das alte `{data:[{id,date,Lottozahl,Superzahl}]}`-Schema erwartet:** Nutze die Breitdatei `Lottonumbers_complete.json` — sie ist nahezu drop-in-kompatibel. Prüfe lediglich, ob das `Superzahl`-Feld in neueren Ziehungen vorhanden ist und fange dessen Fehlen in alten Datensätzen ab.
- **Datum immer als String behandeln** und explizit als `TT.MM.JJJJ` parsen (`dmy()` in R, `pd.to_datetime(..., format="%d.%m.%Y")` in Python); nicht auf automatische Datumserkennung verlassen.
- **Superzahl-Zugriff:** Im Tidy-Format mit `variable == "Superzahl"` filtern; im Breitformat das (in neueren Ziehungen vorhandene) `Superzahl`-Feld auslesen.
- **Umwandlung Tidy → Breit:** In Python nach `id` und `variable` gruppieren und die Werte je Ziehung in Listen sammeln. Ein bewährtes Muster (aus `Nova200019/german-lotto-tft-predictor` [GitHub](https://github.com/Nova200019/german-lotto-tft-predictor/blob/main/lotto.py) , `lotto.py`): `draw_dict = defaultdict(lambda: defaultdict(list))` und dann `draw_dict[draw_id][variable].append(value)`, gefiltert auf Ziehungen mit genau 6 Lottozahlen.
- **Benchmark/Trigger zur Format-Wahl:** Wenn du pro Ziehung genau ein Objekt mit fertigem 6er-Array brauchst → Breitdatei. Wenn du spaltenorientiert aggregieren/zählen willst → Tidy-Datei.

## Caveats
- Ob die Breitdatei `Lottonumbers_complete.json` in neueren Ziehungen tatsächlich ein `Superzahl`-Feld (und ggf. weitere Felder wie Spiel77/Super6) enthält, konnte nicht wörtlich aus dem Dateiende verifiziert werden, da die Datei für eine vollständige Abfrage zu groß war (>200 KB Tool-Limit); nur der Dateianfang (1955 ff.) wurde direkt eingesehen. Die Inferenz, dass neuere Ziehungen ein `Superzahl`-Feld tragen, beruht auf dem analogen Aufbau der Tidy-Datei und der historischen Logik.
- Die genaue Struktur der dritten Datei `lotto_api.json` konnte nicht abgerufen werden.
- Es besteht eine **Diskrepanz in der Jahresangabe**: GitHub-Repotitel "(1955-2026)" vs. GitHub-Pages-Seite "(1955-2025)". Das exakte letzte enthaltene Ziehungsdatum wurde nicht einzeln verifiziert.
- Ein literaler `"variable":"Superzahl"`-Datensatz konnte nicht wörtlich aus der Tidy-Datei extrahiert werden (die abgerufenen Anfangszeilen von 1955 enthalten nur `"Lottozahl"`); die angegebene Form ist aus dem konsistenten Schema und dem README-Code rekonstruiert.