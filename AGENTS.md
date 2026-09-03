# WB-SG1-8G Keypad Control — Οδηγίες για τον επόμενο agent

## Τι είναι το project

Web front-end (μία σελίδα, καθαρό HTML/CSS/JS — **χωρίς build step, χωρίς framework**) που ελέγχει
τη γεννήτρια συχνοτήτων **BG7TBL WB-SG1-8G Wideband Signal Generator** μέσω **Web Serial API** (USB καλώδιο).

- Ένα μόνο αρχείο: **`index.html`** (όλο το markup, CSS και JS μέσα).
- Φιλοξενείται ως **στατική σελίδα στο GitHub Pages** — σερβίρεται αυτούσιο το `index.html` από το root.
- Repo: `traianos1985-glitch/wbsg1-keypad`.
- Η αρχική εφαρμογή (rotary-knob έκδοση) είναι εδώ: https://katzenjens.github.io/wbsg1/ (WB-SG1-8G Lab Dashboard v1.9).
  Αυτό το project είναι η **keypad έκδοση** — ίδιο πρωτόκολλο, διαφορετική εισαγωγή συχνότητας (αριθμητικό πληκτρολόγιο + επιλογή μονάδας).

## Απαιτήσεις εκτέλεσης

- Web Serial API → μόνο **Chrome/Edge σε desktop ή Android**. Δεν δουλεύει σε Firefox/Safari/iOS.
- Απαιτεί **HTTPS** (ή localhost). Το GitHub Pages είναι HTTPS, οπότε ΟΚ.
- Ο χρήστης πατάει **CONNECT** και επιλέγει τη σειριακή θύρα της γεννήτριας.

## Specs υλικού (WB-SG1-8G)

| | CH1 (BNC) | CH2 (SMA) |
|---|---|---|
| Εύρος | 1 Hz – 250 MHz | 35 MHz – 8 GHz |
| Ανάλυση | 1 Hz | **10 Hz** (ADF5355) |
| Modulation | — | ON/OFF keying μόνο (όχι AM/FM) |

- Reference: 10 MHz OCXO, INT/EXT.
- Power output: ADF5355, τυπικά βήματα -4 / -1 / +2 / +5 dBm (οθόνη δείχνει 0dBm).

## Πρωτόκολλο σειριακής (ASCII, 9600 baud, 8N1)

Όλες οι εντολές: αρχή `$`, τέλος `*`.

| Εντολή | Λειτουργία |
|---|---|
| `$A*` | Query status → η γεννήτρια απαντά κείμενο με `CH1 ... <ψηφία> Hz` / `CH2 ... <ψηφία> Hz` |
| `$N*` | MODE / return to normal (έξοδος από sweep) |
| `$S*` | SAVE ρυθμίσεων |
| `$F{ch}{9ψηφία}*` | Set frequency. **CH1** = Hz. **CH2** = Hz÷10 (10 Hz ανάλυση). Padded σε 9 ψηφία. |
| `$W{kid}{start9}{stop9}{steps9}*` | Sweep. `kid=3`→CH1, `kid=4`→CH2. CH1 σε Hz, CH2 σε Hz÷10. |
| `$C{2ψηφία}*` | Contrast 00–63 |
| `$E3232*` / `$E3333*` | Beeper ON / OFF |
| `$E3434*` / `$E3535*` | CH1 RF ON / OFF |
| `$E3636*` / `$E3737*` | CH2 RF ON / OFF |
| `$E3838*` / `$E3939*` | Modulation ON / OFF |

> ΠΡΟΣΟΧΗ: υπάρχει και ΠΑΛΙΑ μονοκάναλη WB-SG1 (ADF4351) με **διαφορετικό** πρωτόκολλο —
> δυαδικό, 57600 baud, prefix byte `0x8f`. **ΔΕΝ ισχύει εδώ.** Μην το μπερδέψεις.

## Δομή κώδικα (`index.html`)

- **State** (γραμμή ~239): `f1/f2` live τιμές Hz, `entry1/entry2` buffers πληκτρολογίου, `unit1/unit2`.
- **Keypad**: `buildKeypad`, `pressKey`, `renderEntry`, `setUnit`.
- **applyFreq(ch)**: validation ορίων + στρογγυλοποίηση CH2 στα 10 Hz → `sync()`.
- **Serial**: `connect()`, `sendCommand()`, `readLoop()`, `parseIncomingData()`.
- **Debug console**: `#debugPanel` (κρυφό, ανοίγει με κουμπί DEBUG). `sendManual`, `logLine`, `hexSuffix`,
  `clearLog`, `copyLog`. Υποστηρίζει HEX view (TX/RX bytes σε hex) — χρήσιμο για sniffing.

## TODO / Μελλοντικές αναβαθμίσεις (κατά προτεραιότητα)

1. **Χαρτογράφηση `$A*` απάντησης** — τρέξε QUERY STATUS με HEX ON και δες αν επιστρέφει PWR/REF/RF-state.
   Ο parser (`parseIncomingData`) διαβάζει μόνο τις συχνότητες τώρα.
2. **Έλεγχος ισχύος εξόδου (PWR/dBm)** — ΑΓΝΩΣΤΗ ASCII εντολή. Χρειάζεται serial sniffing για επιβεβαίωση
   πριν υλοποιηθεί UI. Μην μαντέψεις εντολή.
3. **Επιλογή REF INT/EXT** από το web — επίσης άγνωστη εντολή, χρειάζεται sniffing.
4. **Ζωντανή γραμμή κατάστασης** (PLL LOCK / REF / MOD / OUT / PWR) όπως η φυσική LCD.
5. **Presets/μνήμες** πέρα από το απλό `$S*`.
6. **Βελτιωμένο sweep** — dwell time, progress bar.

### Κανόνας για νέες εντολές
Μη προσθέτεις UI για εντολή που δεν έχει επιβεβαιωθεί με πραγματικό serial sniffing στη συσκευή.
Το firmware/manual του BG7TBL ΔΕΝ είναι δημόσια διαθέσιμο. Χρησιμοποίησε το DEBUG console για reverse-engineering.

## Deploy

Merge στο `main` → το GitHub Pages ανανεώνεται αυτόματα (σερβίρει το `index.html` από το root του `main`).
Δεν υπάρχει build. Το URL είναι της μορφής `https://<owner>.github.io/wbsg1-keypad/`.
