# WB-SG1-8G Keypad — Προτεινόμενες Αναβαθμίσεις (Roadmap)

Φύλλο οδηγιών με τις μελλοντικές αναβαθμίσεις της εφαρμογής, χωρισμένες σε αυτές
που υλοποιούνται **άμεσα** (γνωστό πρωτόκολλο ASCII `$...*`, 9600 8N1) και σε αυτές
που **απαιτούν πρώτα serial sniffing** μέσω του DEBUG console.

> Πρωτόκολλο & specs: δες `AGENTS.md`. Κανόνας: **μην προσθέτεις UI για εντολή
> που δεν έχει επιβεβαιωθεί με πραγματικό serial sniffing στη συσκευή.**

---

## A. Άμεσα υλοποιήσιμα (γνωστό πρωτόκολλο)

### A1. Ζωντανή γραμμή κατάστασης (σαν τη φυσική LCD)  — ΠΡΟΤΕΡΑΙΟΤΗΤΑ 1
Αναπαραγωγή της κάτω γραμμής της οθόνης: `PLL LOCK · REF INT · MOD OFF · OUT ON · PWR 0dBm`.
- Πλήρες parse της απάντησης `$A*` στο `parseIncomingData()` (τώρα διαβάζει μόνο συχνότητες).
- Auto-refresh με polling (π.χ. `$A*` κάθε 2–3 s) όσο υπάρχει σύνδεση.
- Οπτικά indicators (πράσινο/γκρι) πάνω από το keypad.
- **Bonus:** ετοιμάζει το parsing που χρειάζεται για το sniffing των PWR/REF (§B).

### A2. Presets / μνήμες συχνοτήτων  — ΠΡΟΤΕΡΑΙΟΤΗΤΑ 1
Αποθήκευση αγαπημένων setups σε `localStorage`, φόρτωση με ένα κλικ.
- Πεδία preset: `label`, `channel`, `freqHz`, `unit`.
- Παραδείγματα: "WiFi 2.4 GHz", "GPS L1 1575.42 MHz", "FM 100 MHz".
- CRUD: add / rename / delete / reorder· export/import JSON.

### A3. Βελτιωμένο Sweep UI
Το τρέχον sweep είναι "τυφλό" (`$W...*`). Προσθήκη:
- Progress bar + ζωντανή ένδειξη τρέχουσας συχνότητας.
- Πεδία start / stop / steps / dwell.
- Κουμπί STOP → `$N*` (return to normal).

### A4. Έξυπνη είσοδος & validation μονάδων
- Auto όρια ανά κανάλι: CH1 1 Hz–250 MHz, CH2 35 MHz–8 GHz.
- Στρογγυλοποίηση CH2 στα 10 Hz (ήδη γίνεται) με ορατή ένδειξη.
- Καθαρά μηνύματα σφάλματος + οπτική προειδοποίηση εκτός ορίων.

### A5. Keyboard shortcuts (desktop)
- Αριθμοί από το πληκτρολόγιο, `Enter` = apply, `Esc` = clear, `Backspace` = διαγραφή.
- Προσοχή στο CJK IME: αγνόησε `Enter` όταν `isComposing` ή `keyCode===229`.

### A6. Sharing ρυθμίσεων (URL / QR)
- Encode setup (channel/freq/unit) στο query string για share/restore.
- Προαιρετικά QR code για γρήγορο άνοιγμα σε κινητό.

---

## B. Απαιτούν serial sniffing πρώτα (άγνωστες εντολές)

Χρησιμοποίησε το **DEBUG console** (HEX ON → QUERY STATUS `$A*` → COPY) και ζήτησε
από τον χρήστη το raw log πριν υλοποιήσεις οτιδήποτε από τα παρακάτω.

### B1. Έλεγχος ισχύος εξόδου (PWR / dBm)
- ADF5355: τυπικά βήματα -4 / -1 / +2 / +5 dBm (οθόνη δείχνει 0dBm).
- **Άγνωστη ASCII εντολή** — χρειάζεται sniffing από τα φυσικά κουμπιά MODE.

### B2. Επιλογή Reference INT / EXT
- Εναλλαγή reference από το web — **άγνωστη εντολή**, χρειάζεται sniffing.

### B3. Τύπος modulation
- Επιβεβαίωση αν υποστηρίζεται κάτι πέρα από ON/OFF keying (§AGENTS.md).

---

## Προτεινόμενη σειρά υλοποίησης
1. **A1 (γραμμή κατάστασης) + A2 (presets)** — μέγιστο όφελος, 100% γνωστό πρωτόκολλο,
   και προετοιμάζει το parsing για το §B.
2. A3 (sweep) → A4 (validation) → A5 (shortcuts) → A6 (sharing).
3. Μόλις υπάρξει log από τη γεννήτρια: B1 → B2 → B3.

## Deploy
Merge στο `main` → το GitHub Pages ανανεώνεται αυτόματα μέσω του workflow
`.github/workflows/deploy.yml` (σερβίρει το `index.html` από το root). Δεν υπάρχει build step·
το `package.json` υπάρχει μόνο για να τρέχει το v0 preview (στατικός server `serve`).
