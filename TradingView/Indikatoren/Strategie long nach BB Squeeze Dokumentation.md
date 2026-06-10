# Long BB Squeeze Strategy & Indicator

**Version:** 1.0  
**Erstellt mit Pine Script™ v5**  
**Typ:** Reversal-Strategie / Swing-Trading  

---

## 📖 Zusammenfassung (Executive Summary)

Die **"Long BB Squeeze"** Strategie ist eine prozedurale Trading-Ansatz, der darauf abzielt, **frühe Wende-Momente nach einem Konsolidierungsphase** im Aufwärtstrend zu identifizieren. Sie kombiniert drei technische Ansätze:

1.  **Volatilitäts-Filter:** Nutzung der Bollinger Band Width (BB Width) zur Identifikation von "Squeeze"-Phasen (geringe Volatilität), die oft einer großen Bewegung vorausgehen.
2.  **Preisaktions-Signale (Price Action):** Erkennung spezifischer Kerzenmuster (Hammer, Engulfing, Doji), die auf eine Abwehr des unteren Bollinger Bandes und potenzielle Umkehr hindeuten.
3.  **Trend-Following Filter:** Sicherstellung, dass der overarching Trend aufwärts ist, um Only Long-Positionen in einem bullischen Kontext zu handeln.

**Ziel:** Hohe Gewinnwahrscheinlichkeit durch das "Buy the Dip" Prinzip nach einer Konsolidierung, wobei Risiken durch einen dynamischen Trailing Stop verwaltet werden.

---

## 🧠 Die zugrunde liegende Idee (Market Philosophy)

### 1. Die Logik des "Squeeze"
Bollinger Bänder reagieren auf Volatilität. Wenn die Bänder eng zusammenlaufen ("Squeeze"), bedeutet dies, dass der Markt sich konsolidiert und die Volatilität niedrig ist. Historisch gesehen folgen auf Phasen extremer Konsolidierung oft Ausbrüche mit hoher Volatilität.
*   **Unsere Annahme:** Wir suchen nicht nach dem Ausbruch *nach oben*, sondern nach einer **Fehldrehung nach unten** innerhalb oder am Ende dieser Squeeze-Phase, die sich als Fehlsignal erweist und eine Kaufgelegenheit bietet.

### 2. Der "Lower Band" Touch
Das untere Bollinger Band fungiert als dynamische Unterstützung. In bullischen Märkten berühren Preise oft kurzfristig das untere Band, um Liquidity zu sammeln, bevor sie weiter nach oben steigen. Wir definieren diesen Kontakt durch:
$$ \text{Low} \leq \text{Lower Band} \times 1.005 $$Der kleine Puffer von 0.5% (via `touchesLowerBand`) berücksichtigt Rauschen und verhindert zu frühe Signale, wenn der Preis noch nicht "getestet" hat.

### 3. Reversal Confirmation (Price Action)
Nur weil ein Price das untere Band berührt, muss es kein Kauf sein. Wir benötigen Bestätigung durch Kerzenmuster, die **Kraftverkündigung** zeigen:
*   **Hammer/Pinbar:** Lange untere Docht zeigt, dass Verkäufer den Preis drücken konnten, aber Käufer ihn stark zurücktrieben (Abwehr).
*   **Bullish Engulfing:** Vollständige Übernahme des vorherigen roten Körpers durch einen grünen Körper zeigt starke Kaufmomentum.
*   **Doji:** Indiz für Unentschlossenheit, oft ein Vorbote der Trendumkehr, wenn es nach einem Drop auftritt.

### 4. Kontext ist King (Trend Filter)
Um "Catching Falling Knives" zu vermeiden, wird nur gehandelt, wenn der langfristige Trend (200 EMA) aufwärts ist (`close > emaTrend`). Dies stellt sicher, dass wir gegen den Hauptstrom handeln, aber in Richtung des größeren Trends.

---

## ⚙️ Technische Komponenten & Parameter

### A. Bollinger Bands & Width Normalization
*   **Länge (`bbLen`):** Standard 20 Kerzen.
*   **Multiplikator (`mult`):** Standard 2.0 (entspricht ~95% der Datenverteilung bei normalverteilter Verteilung).
*   **Normalized BB Index:** 
    Wir berechnen nicht nur die absolute Bandbreite, sondern normalisieren diese über einen Zeitraum von `histLen` (Standard 125 Kerzen).
    $$ \text{Index} = \left( \frac{\text{Current Width}}{\text{Highest Width}_{\text{history}}} \right) \times 100 $$
    *   **Logik:** Ein Index von < 20 bedeutet, dass die aktuelle Volatilität extrem niedrig ist im Vergleich zu den letzten ~6 Monaten (bei täglichen Charts). Dies definiert den "Squeeze".

### B. Trend Filter (EMA)
*   **Länge (`trendEmaLength`):** Standard 200.
*   **Bedingung:** `close > EMA(200)`
    *   Nur Long-Positionen zulässig, wenn der Preis über dem langfristigen Gleichgewicht liegt.

### C. Reversal Pattern Detection (Custom Logic)
Anstatt sich auf starre Indikator-Kreuzungen zu verlassen, nutzen wir mathematische Berechnung der Kerzenkörper und Dochte:

| Muster | Bedingung im Code | Bedeutung |
| :--- | :--- | :--- |
| **Hammer-Like** | `LowerShadow > Body * 2` & `UpperShadow < Body * 0.5` | Starke Abwehr am Tiefpunkt. |
| **Engulfing** | `Close > Open` & `Open < PrevClose` & `Close > PrevOpen` | Käufer übernehmen komplett die Kontrolle. |
| **Doji-Like** | `Body < (High - Low) * 0.1` | Indiz für Erschöpfung der aktuellen Bewegung. |

---

## 📊 Risiko-Management (Stop Loss & Trailing)

Die Strategie verwendet ein dynamisches Risikomanagement, um Gewinne zu sichern und Verluste zu begrenzen.

### 1. Initialer Stop Loss
Beim Entry wird der Stop Loss gesetzt:$$ \text{Stop}_{initial} = \text{Tiefpunkt der letzten } N \text{ Kerzen} - (\text{ATR}(14) \times 0.5) $$*   **`atrLength` (Default 30):** Definiert den Lookback-Perioden für den Swing Low.
*   **ATR-Puffer:** Der ATR-basierte Puffer (50% des ATR-Wertes) sorgt dafür, dass der Stop nicht zu eng an das Minus gesetzt wird und durch normales Markt-Rauschen nicht sofort ausgelöst wird ("Normal Volatility Noise").

### 2. Dynamischer Trailing Stop
Sobald der Trade im Gewinn ist, wandert der Stop Loss mit:$$ \text{Trailing Stop} = \text{Close} - (\text{ATR}(14) \times 2.5) $$
*   **Warum ATR?** Im Gegensatz zu festen Prozentwerten passt sich der Trailing-Stop der aktuellen Marktvolatilität an. Bei hoher Volatilität ist der Abstand größer, bei niedriger Volatilität enger.
*   **Kombination:** Der aktive Stop ist das **Maximum** aus dem `initialStop` (basiert auf Swing Low) und dem `trailingStop`. Dies verhindert, dass der Stop zurückwandert, wenn der Preis stark steigt.

---

## 🚀 Nutzung im Pine Scanner

Dieser Indikator wurde speziell für die Verwendung im **TradingView Pine Scanner** optimiert.

### Wie man es verwendet:
1.  Füge den Indikator `Long BB Squeeze` zu einem Chart hinzu (oder speichere ihn als Indikator).
2.  Öffne den **Pine Scanner**.
3.  Erstelle einen neuen Scan oder bearbeite einen bestehenden.
4.  Filtere nach: **Indicator Name** > `Long BB Squeeze`.
5.  Setze die Bedingung für das Signal (je nach Setup im Scanner oft "Plot Value > 0" oder spezifische Variable, falls als numerischer Plot ausgegeben).

### Visualisierung im Chart:
*   **Grünes Dreieck (unter der Kerze):** Bestätigungssignal für den Long-Entry (alle Kriterien erfüllt).