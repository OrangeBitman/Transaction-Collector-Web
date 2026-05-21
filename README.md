# Transaction Collector Web

> **Bitcoin-Transaktionen aus mempool.space abrufen & als CSV exportieren**

Eine browserbasierte Single-Page-Application (keine Installation, keine Abhängigkeiten), die Bitcoin-Transaktions-IDs einliest, die Details über die [mempool.space](https://mempool.space) API abruft und strukturierte CSV-Dateien für die Buchführung, Steuererklärung oder Compliance exportiert.

---

## Inhaltsverzeichnis

1. [Funktionsübersicht](#funktionsübersicht)
2. [Schnellstart](#schnellstart)
3. [Eingabedatei](#eingabedatei)
4. [Optionen & Schalter](#optionen--schalter)
5. [Ausgabe-Modi](#ausgabe-modi)
6. [Download](#download)
7. [CSV-Feldbeschreibung](#csv-feldbeschreibung)
8. [Technische Hinweise](#technische-hinweise)

---

## Funktionsübersicht

| Funktion | Beschreibung |
|---|---|
| **Drag & Drop Upload** | Eingabedatei per Drag & Drop oder Klick hochladen |
| **Batch-Verarbeitung** | Beliebig viele TX-IDs in einem Durchlauf verarbeiten |
| **mempool.space API** | Abruf von TX-Details, Blockhöhe, Blockhash und Zeitstempel |
| **Fee-Berechnung** | Automatische Gebührenberechnung falls die API keinen Wert liefert |
| **Split-Modus** | Eine CSV-Datei pro Transaktion (mit optionaler Nummerierung) |
| **Combined-Modus** | Alle Transaktionen in einer einzigen CSV-Datei |
| **ZIP-Export** | Im Split-Modus werden mehrere CSVs automatisch als ZIP gebündelt |
| **UTF-8 BOM** | Alle CSV-Dateien enthalten einen UTF-8 BOM für nahtlose Excel-Kompatibilität |
| **Dezimaltrennzeichen** | Wählbar zwischen Komma (Excel DE) und Punkt (international) |
| **Fortschrittsanzeige** | Echtzeit-Fortschrittsbalken mit Zähler und Statistiken |
| **Log-Ausgabe** | Farbkodiertes Live-Log (Info, Erfolg, Warnung, Fehler, Debug) |
| **Scripthash-Auflösung** | Unterstützt P2PKH, P2SH und Bech32 (SegWit v0/v1) Adressen |
| **Offline-fähig** | Läuft vollständig im Browser – keine Server-seitige Logik |
| **Keine Abhängigkeiten** | Reines HTML/CSS/JavaScript, keine externen Bibliotheken |

---

## Schnellstart

1. `index.html` im Browser öffnen (Doppelklick genügt).
2. Eine Textdatei mit TX-IDs per Drag & Drop in die Drop-Zone ziehen.
3. Optionen nach Bedarf anpassen.
4. Auf **Verarbeiten** klicken.
5. Nach Abschluss die CSV-Datei(en) herunterladen.

---

## Eingabedatei

Die Eingabedatei ist eine einfache **Textdatei** (`.txt` oder `.csv`) mit einer TX-ID pro Zeile.

**Regeln:**

- Jede TX-ID muss exakt **64 Hex-Zeichen** lang sein (`[0-9a-fA-F]{64}`).
- Zeilen, die mit `#` beginnen, werden als **Kommentare ignoriert**.
- Leerzeilen werden übersprungen.
- **Duplikate** werden erkannt und übersprungen (nur die erste Nennung wird verarbeitet).

**Beispiel `transactions.txt`:**

| transaction_id                                                       |
|----------------------------------------------------------------------|
| `a1ef55cc86c0596b55ca92360715a57071cb3d598571013b51121eed4d2642c7`   |
| `4d9887ac983f0487176ed76a5e2d8780bd7b89554f2b813f66b48f0093cacc40`   |
| `adf2355ff1e0fb26b86bf2abd8438662e116c11d0257c76f7602c56cf43b4e57`   |


---

## Optionen & Schalter

Alle Optionen befinden sich im Abschnitt **„Optionen"** der Benutzeroberfläche.

### API-Basis-URL (`--mempool-base`)

| | |
|---|---|
| **Typ** | Text |
| **Standard** | `https://mempool.space/api` |
| **Beschreibung** | Die Basis-URL der mempool.space-kompatiblen API. Kann auf eine eigene mempool.space-Instanz oder einen alternativen Esplora-Endpunkt geändert werden (z. B. `https://blockstream.info/api` oder eine lokale Instanz `http://localhost:8999/api`). Abschließende Schrägstriche werden automatisch entfernt. |

---

### Timeout (`--timeout`)

| | |
|---|---|
| **Typ** | Zahl (Sekunden) |
| **Standard** | `20` |
| **Bereich** | `1` – `120` |
| **Beschreibung** | Maximale Wartezeit in Sekunden für eine einzelne API-Anfrage. Bei langsamen Verbindungen oder stark ausgelasteten Servern empfiehlt sich ein höherer Wert (z. B. `60`). Nach Ablauf des Timeouts wird die Anfrage abgebrochen und als Fehler gewertet. |

---

### Ausgabemodus (`--output-mode`)

| | |
|---|---|
| **Typ** | Auswahl |
| **Optionen** | `split` · `combined` |
| **Standard** | `split` |
| **Beschreibung** | Steuert, wie die CSV-Ausgabe strukturiert wird. Siehe [Ausgabe-Modi](#ausgabe-modi). |

---

### Dezimaltrennzeichen (`--decimal-sep`)

| | |
|---|---|
| **Typ** | Auswahl |
| **Optionen** | `,` (Komma) · `.` (Punkt) |
| **Standard** | `,` (Komma) |
| **Beschreibung** | Legt das Dezimaltrennzeichen für alle BTC-Beträge in der CSV fest. `,` (Komma) ist kompatibel mit Microsoft Excel in deutscher Spracheinstellung. `.` (Punkt) ist der internationale Standard und eignet sich für englischsprachige Tools oder die programmatische Weiterverarbeitung. |

---

### Führende Nullen (`--digits`)

| |                                                                                                                                                                                                                                                                                                                   |
|---|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Typ** | Zahl                                                                                                                                                                                                                                                                                                              |
| **Standard** | `0` (keine führenden Nullen)                                                                                                                                                                                                                                                                                      |
| **Bereich** | `0` – `10`                                                                                                                                                                                                                                                                                                        |
| **Beschreibung** | Legt die Mindestbreite des numerischen Präfix im Dateinamen jeder Split-CSV fest. Bei `3` wird die erste Datei z. B. `001_a1b2c3d4e5f6abcd.csv` benannt. Wird `0` angegeben, erscheint keine Auffüllung (`1_a1b2c3d4e5f6abcd.csv`). Hat keine Wirkung, wenn **Auto Nullen** aktiviert ist oder im combined Modus. |

---

### Auto Nullen (`--auto-digits`)

| |                                                                                                                                                                                                                                                                                                              |
|---|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Typ** | Toggle (Ein/Aus)                                                                                                                                                                                                                                                                                             |
| **Standard** | Aus                                                                                                                                                                                                                                                                                                          |
| **Beschreibung** | Wenn aktiviert, wird die Breite der führenden Nullen automatisch aus der Gesamtzahl der TX-IDs berechnet (z. B. 100 TXs → 3 Stellen: `001`, `002`, …, `100`). **Überschreibt den manuell eingestellten Digits-Wert.** Wenn aktiviert, ist das Digits-Feld deaktiviert. Hat keine Wirkung im combined Modus.  |

---

### Debug-Logging (`--debug`)

| | |
|---|---|
| **Typ** | Toggle (Ein/Aus) |
| **Standard** | Aus |
| **Beschreibung** | Aktiviert ausführliche Debug-Ausgaben im Log-Panel (blau/violett hervorgehoben). Dazu gehören berechnete Scripthashes für jeden Input und Output, die TX-Anzahl pro Adresse, Fallback-Strategien zur Blockhöhen-Ermittlung sowie detaillierte Fehlerursachen. Für die normale Nutzung nicht erforderlich; empfohlen bei der Fehleranalyse. |

---

## Ausgabe-Modi

### Split-Modus (Standard)

Pro TX-ID wird eine eigene CSV-Datei erstellt. 

Beispiel: `001_a1b2c3d4e5f6abcd.csv`

- Bei **einer** Datei: Direkter CSV-Download.
- Bei **mehreren** Dateien: ZIP-Archiv (`transactions.zip`) mit allen CSVs, alternativ sequenzieller Einzeldownload per Klick.

### Combined-Modus

Alle Transaktionen werden in einer einzigen Datei `combined.csv` zusammengefasst. Die Zeilen aller Transaktionen folgen direkt aufeinander unter einem gemeinsamen Header.

---

## Download

- Alle CSV-Dateien werden mit einem **UTF-8 BOM** (Byte Order Mark: `0xEF 0xBB 0xBF`) gespeichert, damit Excel die Datei korrekt als UTF-8 erkennt und Sonderzeichen fehlerfrei darstellt.
- Das Trennzeichen zwischen Feldern ist immer ein **Semikolon** (`;`).
- Felder, die Semikolons, Anführungszeichen oder Zeilenumbrüche enthalten, werden in doppelte Anführungszeichen eingeschlossen; enthaltene Anführungszeichen werden verdoppelt (`""`).
- Zeilenenden sind `CRLF` (`\r\n`), wie im CSV-Standard (RFC 4180) vorgeschrieben.

---

## CSV-Feldbeschreibung

Die CSV-Datei enthält immer genau die folgenden **44 Felder** in dieser Reihenfolge, getrennt durch Semikolon (`;`).

| # | Feldname | Beschreibung                                                                                                                                                                                           |
|---|---|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | `event_number` | Laufende Nummer des Ereignisses innerhalb der Verarbeitung (reserviert, aktuell leer).                                                                                                                 |
| 2 | `event` | Art des Ereignisses. Aktuell immer `transaction`.                                                                                                                                                      |
| 3 | `event_date_time_utc` | Zeitstempel des Blocks, in dem die Transaktion bestätigt wurde, in **ISO 8601 UTC** (z. B. `2024-03-15T10:22:45.000Z`). Leer, wenn die Transaktion noch unbestätigt ist.                               |
| 4 | `event_date_time_mez` | Derselbe Zeitstempel, umgerechnet in **Mitteleuropäische Zeit (MEZ/MESZ)**, formatiert als `YYYY-MM-DD HH:MM:SS` für Excel.                                                                            |
| 5 | `transaction_id` | Die vollständige **Transaction ID** (TXID) – 64 hexadezimale Zeichen.                                                                                                                                  |
| 6 | `transaction_explorer_url` | Direkt aufrufbare URL zur Transaktion auf mempool.space (z. B. `https://mempool.space/tx/<txid>`).                                                                                                     |
| 7 | `transaction_explorer_url_link` | Reserviertes Feld für einen klickbaren Hyperlink (z. B. Excel-HYPERLINK-Formel). Aktuell leer.                                                                                                         |
| 8 | `transaction_fee_btc` | Netzwerkgebühr der Transaktion in **BTC** (8 Dezimalstellen). Format abhängig vom eingestellten Dezimaltrennzeichen. Leer bei Coinbase-Transaktionen oder wenn die Gebühr nicht ermittelt werden kann. |
| 9 | `address_index` | Nullbasierter Index des Inputs oder Outputs innerhalb der Transaktion (0, 1, 2, …).                                                                                                                    |
| 10 | `address_type` | Gibt an, ob es sich um einen **Input** (`input`) oder **Output** (`output`) der Transaktion handelt.                                                                                                   |
| 11 | `address` | Die Bitcoin-Adresse des Inputs bzw. Outputs. Leer bei Coinbase-Inputs oder OP_RETURN-Outputs ohne auflösbare Adresse.                                                                                  |
| 12 | `address_value_btc` | Betrag in **BTC**, der an diese Adresse gesendet oder von ihr ausgegeben wurde (8 Dezimalstellen). Format abhängig vom eingestellten Dezimaltrennzeichen.                                              |
| 13 | `address_owner` | Name oder Bezeichnung des Adressinhabers. Aktuell leer – manuell befüllbar.                                                                                                                            |
| 14 | `address_explorer_url` | Direkt aufrufbare URL zur Adresse auf mempool.space (z. B. `https://mempool.space/address/<adresse>`). Leer, wenn keine Adresse ermittelt werden konnte.                                               |
| 15 | `address_explorer_url_link` | Reserviertes Feld für einen klickbaren Hyperlink zur Adresse (z. B. Excel-HYPERLINK-Formel). Aktuell leer.                                                                                             |
| 16 | `blockheight` | Blockhöhe (Block-Nummer) des Blocks, in dem die Transaktion bestätigt wurde. Leer bei unbestätigten Transaktionen.                                                                                     |
| 17 | `blockhash` | Hash des Blocks, in dem die Transaktion enthalten ist (64 Hex-Zeichen).                                                                                                                                |
| 18 | `exchange_name` | Name der Börse / Wallet oder Handelsplattform. Manuell befüllbar.                                                                                                                                      |
| 19 | `exchange_type` | Typ der Börse / Wallet. Manuell befüllbar.                                                                                                                                                             |
| 20 | `wallet_account_number` | Account innerhalb eines Wallets. Manuell befüllbar.                                                                                                                                                    |
| 21 | `wallet_account_name` | Account-Name innerhalb eines Wallets. Manuell befüllbar.                                                                                                                                               |
| 22 | `derivation_path` | BIP32/BIP44/BIP84-Ableitungspfad der verwendeten Adresse (z. B. `m/84'/0'/0'/0/5`). Manuell befüllbar.                                                                                                 |
| 23 | `wallet_account_extended_public_key_1` | Extended Public Key (xPub/zPub/ypub) des Wallet-Kontos (erster Schlüssel bei Multisig). Manuell befüllbar.                                                                                             |
| 24 | `master_fingerprint_1` | Master-Fingerprint des Wallets (1) (4 Byte Hex). Manuell befüllbar.                                                                                                                                    |
| 25 | `wallet_account_extended_public_key_2` | Extended Public Key des zweiten Signierschlüssels (bei Multisig). Manuell befüllbar.                                                                                                                   |
| 26 | `master_fingerprint_2` | Master-Fingerprint des Wallets (2) (bei Multisig). Manuell befüllbar.                                                                                                                                  |
| 27 | `exchange_rate_btc_euro` | Wechselkurs BTC/EUR zum Zeitpunkt der Transaktion. Manuell befüllbar.                                                                                                                                  |
| 28 | `exchange_fee_btc` | Gebühr der Börse in BTC. Manuell befüllbar.                                                                                                                                                            |
| 29 | `exchange_fee_euro` | Gebühr der Börse in Euro. Manuell befüllbar.                                                                                                                                                           |
| 30 | `amount_btc` | Handelsbetrag in BTC. Manuell befüllbar.                                                                                                                                                               |
| 31 | `amount_euro` | Handelsbetrag in Euro. Manuell befüllbar.                                                                                                                                                              |
| 32 | `description_owner` | Freitext-Beschreibung für den Wallet-Inhaber (z. B. Verwendungszweck). Manuell befüllbar.                                                                                                              |
| 33 | `description_authority` | Freitext-Beschreibung für eine Behörde oder externen Prüfer. Manuell befüllbar.                                                                                                                        |
| 34 | `document_1` | Felder für Verweise auf Belege und Nachweisdokumente (z. B. Dateinamen oder URLs). Manuell befüllbar.                                                                                                  |
| 35 | `document_2` | Zweites Dokumentenfeld. Manuell befüllbar.                                                                                                                                                             |
| 36 | `document_3` | Drittes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                             |
| 37 | `document_4` | Viertes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                             |
| 38 | `document_5` | Fünftes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                             |
| 39 | `document_6` | Sechstes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                            |
| 40 | `document_7` | Siebtes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                             |
| 41 | `document_8` | Achtes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                              |
| 42 | `document_9` | Neuntes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                             |
| 43 | `document_10` | Zehntes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                             |
| 44 | `document_11` | Elftes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                              |
| 45 | `document_12` | Zwölftes Dokumentenfeld. Manuell befüllbar.                                                                                                                                                            |

> **Hinweis zu automatisch befüllten vs. manuell befüllbaren Feldern:**
> Die Felder `event`, `event_date_time_utc`, `event_date_time_mez`, `transaction_id`, `transaction_explorer_url`, `transaction_fee_btc`, `address_index`, `address_type`, `address`, `address_value_btc`, `address_explorer_url`, `blockheight` und `blockhash` werden automatisch durch die Anwendung befüllt. Alle übrigen Felder sind für die manuelle Nachbearbeitung (z. B. in Excel) vorgesehen.

---

## Technische Hinweise

### API-Ratenlimit

Die Anwendung wartet zwischen zwei aufeinanderfolgenden API-Anfragen mindestens **250 ms**, um eine Überlastung des mempool.space-Servers zu vermeiden. Bei Fehlern wird einmal automatisch mit exponentiellem Backoff (max. 500 ms) wiederholt.

### Blockhöhen-Ermittlung (Fallback)

Wenn die TX-API keine Blockhöhe liefert, ermittelt die Anwendung diese über die **Scripthash-History** der Ausgabe-Adressen. Dabei wird die Adresse in einen Electrum-kompatiblen Scripthash (SHA-256, Little-Endian) umgerechnet und die TX-Liste der Adresse abgefragt.

### Blockhash und Blockzeit (Fallback)

Fehlen Blockhash oder Blockzeit in der API-Antwort, werden diese über den **Block-Header-Endpunkt** (`/block/<hash>/header`) nachgeladen und per doppeltem SHA-256 selbst berechnet.

### Unterstützte Adresstypen

| Typ | Präfix | Standard |
|---|---|---|
| P2PKH | `1` (Mainnet), `m`/`n` (Testnet) | Legacy |
| P2SH | `3` (Mainnet), `2` (Testnet) | Wrapped SegWit |
| P2WPKH / P2WSH | `bc1q` | Native SegWit v0 |
| P2TR | `bc1p` | Taproot (SegWit v1) |

### Browser-Kompatibilität

Die Anwendung nutzt ausschließlich moderne Web-Standard-APIs:

- `fetch` (Netzwerk)
- `SubtleCrypto` (SHA-256)
- `TextEncoder` / `TextDecoder`
- `URL.createObjectURL`
- `FileReader`

Empfohlen: **Chrome 90+**, **Firefox 90+**, **Safari 15+**, **Edge 90+**.

---

## Lizenz

Siehe [LICENSE](LICENSE).