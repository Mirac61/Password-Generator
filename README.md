# Passwort Generator

Ein schlichter, clientseitiger Passwort-Generator mit Echtzeit-Entropie-Visualisierung — gebaut mit Vue 3 und Tailwind CSS.

---

## Was es macht

- Generiert kryptografisch zufällige Passwörter mit bis zu 32 Zeichen
- Zeigt die Passwortstärke in Echtzeit anhand einer Entropie-Berechnung
- Schätzt, wie lange ein moderner Angreifer zum Knacken bräuchte
- Erklärt die Mathematik hinter der Stärkebewertung — live, während man die Einstellungen ändert

## Technologien

- **Vue 3** (Composition API + `<script setup>`)
- **TypeScript**
- **Tailwind CSS**
- **Vite**

---

## Lokale Installation

```bash
# Abhängigkeiten installieren
npm install

# Entwicklungsserver starten
npm run dev

# Produktions-Build erstellen
npm run build
```

---

## Wie die Stärkeberechnung funktioniert

Die Passwortstärke wird in **Entropie-Bits** gemessen, nach folgender Formel:

```
Entropie = Länge × log₂(Zeichenpool)
```

Der Zeichenpool wächst je nachdem, welche Zeichentypen aktiviert sind:

| Typ             | Zeichen |
|-----------------|---------|
| Großbuchstaben  | 26      |
| Kleinbuchstaben | 26      |
| Zahlen          | 10      |
| Sonderzeichen   | 27      |

Die Cracking-Zeit-Schätzung geht von **1 Milliarde Versuchen pro Sekunde** aus — ein realistischer Wert für moderne Offline-Angriffe.

| Entropie   | Stärke     |
|------------|------------|
| < 40 Bits  | Schwach    |
| 40–60 Bits | Okay       |
| 60–80 Bits | Stark      |
| > 80 Bits  | Sehr Stark |

---

## Projektstruktur

```
src/
└── components/
    └── PasswordGenerator.vue   # Hauptkomponente, gesamte Logik
```

---

## Lizenz

MIT
