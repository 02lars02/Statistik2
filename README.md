# Kickbase: Bewertung von Marktwerten mit linearer Regression

Dieses Repository enthält die Projektarbeit (EDS – Statistik II), umgesetzt als **ein einziges, ausführbares Jupyter Notebook**: `Auswertung.ipynb`.

Ziel ist es, Kickbase-Spieler hinsichtlich **Über- bzw. Unterbewertung** zu beurteilen.
Dazu wird ein "fairer" Marktwert mit einem linearen Regressionsmodell geschätzt und mit dem beobachteten Marktwert verglichen.

## Hypothesen

- **H1:** Spieler mit vielen Punkten sind tendenziell überbewertet.
- **H2:** Die Position hat einen Einfluss auf den Marktwert.
- **H3:** Ein positiver Marktwert-Trend geht mit Unterbewertung einher.

## Datenquelle

Die Daten werden live über eine (inoffizielle) API bezogen:

- Endpoint: `https://api.kbstats.de/api/v1/players`
- Verwendete Variablen:
  - `name` (Spielername)
  - `marketValue` (aktueller Marktwert)
  - `totalPoints` (Gesamtpunkte)
  - `position` (Positionscode)
  - `trend` (Marktwert-Trend)

**Hinweis:** Da die Daten live von der API kommen, können sich Werte über Zeit verändern.

## Methodik (kurz)

### 1) Datenaufbereitung
- Laden der Spieler über die API.
- Auswahl relevanter Spalten.
- Entfernen fehlender/unsinniger Werte (z.B. `marketValue > 0`, `totalPoints > 0`).

### 2) "Fairer" Marktwert (Hauptmodell)
Der "faire" Marktwert wird über eine **lineare Regression (OLS)** geschätzt, basierend auf

- Gesamtpunkten (`totalPoints`)
- Position (Dummy-Variablen)
- **positionsspezifischen Bewertungsprinzipien** über Interaktionen `totalPoints × Position`

Damit erhält jede Position eine eigene Steigung (wie stark Punkte den Marktwert beeinflussen).

Im Notebook werden anschließend berechnet:
- `predMarketValue` = vorhergesagter ("fairer") Marktwert
- `diff = marketValue - predMarketValue`
  - `diff > 0`  → tendenziell **überbewertet** ("zu teuer")
  - `diff < 0`  → tendenziell **unterbewertet** ("zu billig")

### 3) Hypothesentests
- **H1:** Regression im Top-20%-Punkte-Segment: Abweichung zu einem Positions-Baseline-Modell (`diff_base`) ~ `totalPoints`.
- **H2:** Modellvergleich (ANOVA/F-Test): Modell ohne Position vs. Modell mit Position (kontrolliert für Punkte).
- **H3:** Regression: `diff` ~ `trend`.

**Wichtig:** `trend` wird im Notebook bewusst **nicht** im Hauptmodell zur Schätzung des fairen Marktwerts verwendet (sondern nur für H3 und die Bereinigung). Die Begründung dafür wird in der Präsentation erläutert.

## Output

Beim Ausführen des Notebooks wird eine Excel-Datei erzeugt:
- `kickbase_output.xlsx`

Enthalten sind u.a.:
- Spielername
- Position (numerisch)
- beobachteter Marktwert
- vorhergesagter Marktwert
- Differenz (`diff`)

## Ausführen des Codes

### Voraussetzungen
- Python (empfohlen: 3.10+)
- Jupyter Notebook / JupyterLab

### Ausführung im Notebook
1. Repository klonen.
2. `Auswertung.ipynb` in Jupyter öffnen.
3. **Alle Zellen ausführen** (`Run All`).

Die erste Code-Zelle installiert die benötigten Pakete per `%pip` automatisch.

## Repository-Struktur

- `Auswertung.ipynb` – komplette, dokumentierte Auswertung (Daten, Modelle, Hypothesentests, Export)
- `Evaluation/` – enthält Unterlagen der 2. Zwischenpräsentation (z.B. `Datenanalyse.ipynb`)
- `kickbase_output.xlsx` – Output (wird beim Lauf erzeugt)

## Hinweise zur Reproduzierbarkeit

- Das Notebook ist so aufgebaut, dass es **nach dem Klonen ohne absolute Pfade** lauffähig ist.
- Resultate können sich ändern, wenn sich die API-Daten ändern (Marktwerte/Trends sind dynamisch).
