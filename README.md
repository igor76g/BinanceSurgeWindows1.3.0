# Binance Surge Monitor (Windows)

Scans all Binance USDT-M perpetual futures (public API only, no API key) and shows two lists in one window (drag the divider between them to resize):

1. **List 1 - volume + NATR surge.** Coins where both the trade volume and the NATR of the last 15 minutes are
   more than 4 times higher than their usual level. Conditions (all editable in *Settings*):
   24h volume > 15M USDT, NATR (last window) >= 0.7%, volume ratio > 4, NATR ratio > 4.
   Every coin that newly enters this list triggers a notification (sound + always-on-top popup + Windows toast)
   with the ticker. A coin is not notified again within the cooldown (default 30 minutes); the time of the last notification is stored on disk
   (`alert_times.json`), so Stop/Start and restarting the app do not reset the pause.
2. **List 2 - top 7 coins by number of trades** in the last 15 minutes. Conditions: 24h volume > 30M USDT,
   at least 10,000 trades in the window, NATR >= 0.7%. No notifications (as requested).

Both lists are re-checked every 5 minutes. **Double-click a ticker** in either list to open the coin in Vataga.terminal.

## Definitions (1-minute candles)

- NATR of a candle = true range / close x 100; NATR of the window = average over the last 15 candles.
- "Usual level" = average over the baseline period before the window (default 180 minutes).
- Volume ratio = volume of the window / (average 1-minute volume of the baseline x 15).
- NATR ratio = NATR of the window / average NATR of the baseline.
- Number of trades comes from the same candles (field "number of trades").
Window length, baseline length, the 4x ratios, volume limits, NATR limits, trade limit, list size and the
check interval are separate settings.

## Install

Build the installer with GitHub (free):
1. Create a repository and upload the contents of this folder (the `.github` folder must be included).
2. Open **Actions** > "Build Windows installer" > Run workflow (it also starts after every push).
3. When it finishes (3-5 minutes) download the artifact **BinanceSurgeMonitor-Setup** and run the .exe.
Or run from source: `pip install -r requirements.txt` then `python run.py`.
Tests: `python -m unittest discover -s tests -v`

## Vataga.terminal

1. In the terminal: Settings > General > turn on **Application integration**.
2. Right-click the title of the window with the order book > **Link to window** > **Screener**.
3. Keep the terminal running. In this app: Settings > **Test Vataga** (BTCUSDT should open).
The app sends `POST http://localhost:5005/api/open/BinanceSurgeMonitor/binance/F/BTCUSDT`
(market F = futures, S = spot; the port is editable).

## Traffic and limits

The first check downloads about 200 one-minute candles for every coin above the volume limit (roughly 100-200 coins);
later checks download only the few new candles. Every cycle also loads the 24h ticker list once. If Binance answers
HTTP 429/418 the app pauses as requested by the server. If data cannot be loaded 3 checks in a row, a red
"Connection lost" popup appears (and a green "Connection restored" later).

Files: settings `%APPDATA%\BinanceSurgeMonitor\settings.json`, alert history `alerts.log` (button *Alerts log*).

## 1.2.0
Every row of both lists has a "Copy" cell: one click copies the ticker (e.g. INITUSDT) to the clipboard. Double-click on the ticker still opens it in Vataga.

## 1.3.0
- **Chart** cell in every row (next to Copy): opens a mini chart of the coin. **Mouse wheel = scroll through history**, Ctrl + wheel = zoom, Home / End = oldest / newest, arrows also scroll. 1000 candles are loaded once, the newest are refreshed every 10 seconds; intervals 1m ... 1d.
- `alerts.log` is emptied when the app is closed (Settings > "Empty the alerts log when the app is closed", on by default).
- Compact window (660x540): status next to the buttons, shorter lists and log, narrower columns.
