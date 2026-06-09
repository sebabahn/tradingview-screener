# tradingview-screener

Sammlung von TradingView Pine Scripts für effektive Market-Screener.

## BBW Trend-Filter Screener (Letzte 7 Tage)

**enge-bb-mit-sma** – Bollinger BandWidth Indikator mit SMA-Trendfilter und rückwirkender Signalerkennung.

### Features
- Bollinger BandWidth Squeeze & Expansion Erkennung
- Strenger SMA50/SMA20 Aufwärtstrend-Filter
- Signale der **letzten 7 Handelstage** (konfigurierbar)
- Automatische Invalidierung bei Trendbruch
- Screener-kompatible Plot-Ausgaben
- Integriertes Dashboard

### Installation
1. Kopiere den Inhalt von `enge-bb-mit-sma` in ein neues Pine Script in TradingView
2. Füge es zu deinen Charts oder Screenern hinzu

### Screener-Nutzung
Verwende folgende Bedingungen im TradingView Screener:
- `Signal: Squeeze + Trend (7 Tage) = 1`
- `Signal: Expansion + Trend (7 Tage) = 1`

### Backtesting & Weiterentwicklung
Gerne erweitern mit:
- Multi-Timeframe-Unterstützung
- Alert-Funktionen
- Performance-Statistiken
