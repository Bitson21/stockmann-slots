# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt

„Stockmann Slots“: Browser-Slotmaschine (3 Walzen, 1 Gewinnlinie) im Wald-/Groot-Thema. Das Bonussymbol ist das Gesicht eines Freundes („Stockmann“). Das ganze Spiel steckt in einer einzigen Datei, `index.html` (HTML, CSS und JS inline). Es gibt keinen Build, keine Abhängigkeiten, keine Tests und keinen Linter. Töne werden per WebAudio erzeugt.

**Ausführen:** `index.html` direkt im Browser öffnen.

## Architektur (alles in `index.html` im `<script>`)

- **Einstellungen oben im Script:** `BET` (10), `START_BALANCE` (1000), `WEIGHTS` (Grundspiel-Walzen), `FS_WEIGHTS` (eigene Freispiel-Walzen **ohne** Stockmann), `PAY_3` (Gewinne in × Einsatz), `PAY_3_FACE`, `STICK_PRIZES` (die 5 Stöcke).
- **`FACE_IMAGE`:** das Foto als Base64-JPEG (256×256) in einer sehr langen Zeile. Beim Lesen der Datei diese Zeile überspringen. Das Foto ist absichtlich fest eingebaut und nicht austauschbar.
- **Walzen:** Jede Walze hat `baseStrip` und `fsStrip` (je 48 Felder). Der Getter `strip` wählt automatisch je nach `state.freeSpins`. `reel.shown` merkt sich die 3 sichtbaren Symbole, damit beim Wechsel zwischen Grund- und Freispielwalzen nichts springt.
- **`evaluate(line)`:** Es zählen nur 3 gleiche Symbole (Joker ersetzt alles außer Stockmann). 3× Joker zahlt `PAY_3.wild`. 3× Stockmann = Bonus.
- **Ablauf:** `spin()` → `triggerBonus()` (setzt `state.bonusPending`, zeigt `showCelebration`) → Stock-Auswahl `pickStick()` → Freispiele → `endFreeSpins()` → Gewinn-Show `showBonusWin()` (zählt hoch, Stufen aus `WIN_TIERS`).
- **Drama:** Zwei Stockmänner auf Walze 1+2 → `startDrama()`/`stopDrama()`, Walze 3 dreht 3 s länger.
- **Geld:** immer über `cents()` runden und über `money()` anzeigen (deutsches Format mit Tausenderpunkt).
- **Test-Hilfe:** `window.__force = ['face','face','face']` erzwingt das nächste Ergebnis (in der Browser-Konsole).

## Mathematik (Stand: Grundspiel ~4,8 % Treffer, Bonus alle ~50 Drehs, RTP ~99,5 %)

- RTP = Linien-Erwartungswert Grundspiel + P(Bonus) × (10 + Ø(Multi × Freispiele) × Erwartungswert pro Freispiel).
- Bei jeder Änderung an Walzen, Gewinnen oder Stöcken den RTP **exakt** über alle 48³ Kombinationen mit der echten `evaluate()`-Funktion im Browser nachrechnen, nicht nur schätzen. Simulationen schwanken wegen der großen Bonusgewinne um ±1 %.
- Der Nutzer will ca. 99,5 % Auszahlung und den Bonus ca. alle 50 Drehs (13 Stockmann-Felder von 48).

## Hinweise

- UI-Texte und Kommentare auf Deutsch.
- `gesicht.jpg` (Originalfoto) liegt nur lokal und ist per `.gitignore` ausgeschlossen.
- Änderungen laufen über PRs auf GitHub (Remote `Bitson21/stockmann-slots`, öffentlich).
- Live über GitHub Pages (Branch `main`): https://bitson21.github.io/stockmann-slots/ – jeder Merge auf `main` geht sofort live. Die Seite ist per `<meta name="robots">` für Suchmaschinen gesperrt, weil sie ein Foto einer Privatperson enthält.
