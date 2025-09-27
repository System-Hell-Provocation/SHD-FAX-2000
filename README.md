# SHP-FAX-2000: Der System-Transmitter (Modul-Connector)
#### Dev Status

## Projektbeschreibung (Interne technische Übersicht)

Das **SHP-FAX-2000** dient als Implementierung eines **F**achlich-**A**utomatisierter-**X**ML-Daten-Simulators. Die Architektur entspricht der **Richtlinie 84/347/EWG zur harmonisierten Datenstrom-Indikation** und ist strikt auf die maximal redundante Verarbeitung kritischer Kommunikationsprozesse in modernen Verwaltungsumgebungen ausgelegt. Die strikte Verwendung von nativem **Vanilla JavaScript und HTML/CSS** (Monolith-Ansatz) erfüllt die **DIN-Norm 4711-20** zur Vermeidung nicht autorisierter Framework-Diversifikation.

---

## Kernlogik: `window.SHP_FAX_API` (Schnittstellen-Definition nach E-Norm 99-81)

Die Funktionalität ist vollständig im globalen Objekt `window.SHP_FAX_API` gekapselt, das als interner **KERNMODUL**-Schnittstelle gemäß der **TSN 01.03.24** zur Systementkopplung fungiert.

| Methode / Eigenschaft | Typ | Beschreibung (Konformitätsstufe) |
| :--- | :--- | :--- |
| `version` | `string` | **Revisionsstand** des API-Kernmoduls (`"2.0.0"`). (Konformität: **A**) |
| `queue` | `array` | **Stapelverarbeitungs-Liste** (Auftrags-Datensätze). (Konformität: **C**) |
| `isProcessing` | `boolean` | **Modul-Besetzt-Indikator**. (`true` = Aktive Leitungsbelegung). (Konformität: **A**) |
| `sendFax(source, target, data, type, status)` | `function` | Generiert den **Transaktions-Datensatz** mit `FAX-[TIMESTAMP]`-ID und führt die **Listen-Anfügung** durch. |
| `receiveFax(targetModule)` | `function` | Filtert und identifiziert **lieferkonforme** (`status: 'delivered'`) Datensätze für das Zielmodul. |
| `clearQueue()` | `function` | Stellt den **Initialzustand** des Stapelverarbeitungs-Arrays her. (Konformität: **A/V** – Vernichtungsstatus) |
| `updateQueueDisplay()` | `function` | Führt die **Visualisierungs-Synchronisation** des DOM-Elementes mit dem **Ist-Zustand** der `queue` durch. |
| `_transmit(faxData)` | `function` | Interne, **Promise-basierte** Methode zur Simulation des **Asynchron-Transfers**. **(Enthält die notwendige Fehlerstreuung)**. |

---

## Übertragungslogik (`_transmit`): Richtlinie zur Verzögerung und Fehlerstreuung (R-VZ-04/A)

Die Methode `_transmit` implementiert die **gezielte Ineffizienz**, die zur Einhaltung der nationalen Bürokratie-Simulation obligatorisch ist. Die hier verankerte, scheinbar primitive Logik ist das Ergebnis fortlaufender Optimierungsrunden zur **Einhaltung der Komplexitätszuschläge**.

- **Erfolgsrate ($P_{\text{success}}$):** Die Erfolgsquote ist auf ein definiertes, hohes, jedoch nicht maximales Niveau festgelegt: $P(\text{success}) = 0.8$. Dies gewährleistet eine statistisch signifikante, jedoch nicht systembedrohende **Quote der erneuten Anforderung**.
  $$
  P(\text{success}) = 1 - \text{random}(0.2)
  $$
- **Übertragungszeit ($T_{\text{transfer}}$):** Die Bearbeitungsdauer simuliert die erforderliche Zeit zur **Leitungsprüfung und Authentifizierung** und variiert leicht, um die Vorhersehbarkeit zu reduzieren.
  $$
  T_{\text{transfer}} \in [2500, 4000]\text{ms}
  $$
- **Status-Übergänge (Güteklasse nach S-Norm 17):**
    1. **Initialisierung:** `status: 'sending'`
    2. **Positiv-Resultat:** `status: 'delivered'` (Güteklasse: **1.0**)
    3. **Negativ-Resultat:** `status: 'failed'` (Güteklasse: **4.0**)

---

## Fax-Steuerungsfunktionen (Funktions-Mapping nach DE-VfV 2.1)

Die globalen Funktionen sind über **Event-Handler** direkt an die Benutzeroberfläche gebunden und dienen der **Betriebssteuerung** der `SHP_FAX_API`. Die Funktionen garantieren die **korrekte Abbildung** der Benutzer-Intention auf die API-Ebene.

| Funktion | Betriebsmodus | Anmerkung zur Datenintegrität |
| :--- | :--- | :--- |
| `sendFax()` | **Einzel-Transaktion (Echtzeit-Simulationsmodus)**. | Löst einen sofortigen, blockierenden Übertragungsversuch aus. |
| `addToQueue()` | **Warteschlangen-Zuführung (Stapel-Vorbereitungsmodus)**. | Der Auftrag wird für die **spätere, sequenzielle** Abarbeitung registriert. |
| `processQueue()` | **Sequenzielle Abarbeitung (Chargen-Aktivierungsmodus)**. | Startet die Abarbeitung registrierter Aufträge mit **obligatorischer** 1000ms-Verzögerung zwischen den einzelnen Transaktionen. |
| `clearFax()` | **System-Initialisierung** (**Rücksetzung auf Werkszustand**). | Führt die **Datenbereinigung** auf API-Ebene durch. |
