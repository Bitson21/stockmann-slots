# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt

„Stockmann Slots“: Browser-Slotmaschine (3 Walzen, 3 waagerechte Gewinnlinien) im Wald-/Groot-Thema. Das Bonussymbol ist das Gesicht eines Freundes („Stockmann“). Das ganze Spiel steckt in einer einzigen Datei, `index.html` (HTML, CSS und JS inline). Es gibt keinen Build, keine Abhängigkeiten, keine Tests und keinen Linter. Töne werden per WebAudio selbst erzeugt (Spielhallen-Stil). Keine fremden Sounds verwenden, z. B. nicht aus „Book of Ra“ (urheberrechtlich geschützt).

**Ausführen:** `index.html` direkt im Browser öffnen.

## Architektur (alles in `index.html` im `<script>`)

- **Einstellungen oben im Script:** `BET` (10, gilt für alle 3 Linien zusammen), `START_BALANCE` (1000), `ROWS` (die 3 Linien), `WEIGHTS` (Grundspiel-Walzen, 9 Stockmänner), `FS_WEIGHTS` (eigene Freispiel-Walzen **ohne** Stockmann), `PAY_3` (Gewinne pro Linie in × Einsatz, auch Kommazahlen), `PAY_3_FACE`, `STICK_PRIZES` (die 5 Stöcke).
- **`FACE_IMAGE`:** das Foto als Base64-JPEG (256×256) in einer sehr langen Zeile. Beim Lesen der Datei diese Zeile überspringen. Das Foto ist absichtlich fest eingebaut und nicht austauschbar.
- **Walzen:** Jede Walze hat `baseStrip` und `fsStrip` (je 48 Felder). `makeStrip()` verteilt die Stockmänner gleichmäßig (nie zwei innerhalb von 3 Feldern), damit pro Dreh höchstens ein Bonus entstehen kann; die übrigen Symbole werden gemischt. Der Getter `strip` wählt automatisch je nach `state.freeSpins`. `reel.shown` merkt sich die 3 sichtbaren Symbole, damit beim Wechsel zwischen Grund- und Freispielwalzen nichts springt.
- **`evaluate(line)`:** wertet eine Linie aus. Es zählen nur 3 gleiche Symbole (Joker ersetzt alles außer Stockmann). 3× Joker zahlt `PAY_3.wild`. 3× Stockmann = Bonus.
- **`evaluateSpin(grid)`:** wertet alle 3 Linien aus (`grid[walze] = [oben, Mitte, unten]`). `highlightLine(row)` markiert eine Gewinnlinie.
- **Ablauf:** `spin()` → `triggerBonus()` (setzt `state.bonusPending`, zeigt `showCelebration`) → Stock-Auswahl `pickStick()` → Freispiele → `endFreeSpins()` → Gewinn-Show `showBonusWin()` (zählt hoch, Stufen aus `WIN_TIERS`).
- **Spannungs-Sound:** `sound.faceLand(1)` wenn Walze 1 mit Kopf stoppt, `sound.faceLand(2)` wenn Walze 2 einen Kopf in derselben Reihe zeigt (nur wenn der Bonus noch möglich ist). `sound.bonus()` ist eine eigene Melodie (E-Hijaz-Tonleiter), keine Kopie.
- **Drama:** Zwei Stockmänner auf Walze 1+2 in derselben Reihe → `startDrama(sekunden, reihe)`/`stopDrama()`, Walze 3 dreht 3 s länger.
- **Geld:** immer über `cents()` runden und über `money()` anzeigen (deutsches Format mit Tausenderpunkt).
- **Test-Hilfe:** `window.__force = ['face','face','face']` erzwingt das nächste Ergebnis auf der mittleren Linie (in der Browser-Konsole).

## Mathematik (Stand: Grundspiel ~20 % Gewinn-Drehs, Bonus alle ~50 Drehs, Ø Bonus ~42×, RTP ~99,6 %)

- RTP = Erwartungswert aller 3 Linien im Grundspiel + P(Bonus) × (10 + Ø(Multi × Freispiele) × Erwartungswert aller 3 Linien pro Freispiel).
- Bei jeder Änderung an Walzen, Gewinnen oder Stöcken den RTP **exakt** über alle 48³ Walzen-Stellungen mit `evaluateSpin()` im Browser nachrechnen (bei 3 Linien zählt die Reihenfolge auf den Walzen für Bonus und Trefferquote), nicht nur schätzen. Simulationen schwanken wegen der großen Bonusgewinne um ±1 %.
- Der Nutzer will ca. 99,5 % Auszahlung und den Bonus ca. alle 50 Drehs (9 Stockmann-Felder von 48 bei 3 Linien).

## Hinweise

- UI-Texte und Kommentare auf Deutsch.
- `gesicht.jpg` (Originalfoto) liegt nur lokal und ist per `.gitignore` ausgeschlossen.
- Änderungen laufen über PRs auf GitHub (Remote `Bitson21/stockmann-slots`, öffentlich).
- Live über GitHub Pages (Branch `main`): https://bitson21.github.io/stockmann-slots/ – jeder Merge auf `main` geht sofort live. Die Seite ist per `<meta name="robots">` für Suchmaschinen gesperrt, weil sie ein Foto einer Privatperson enthält.
