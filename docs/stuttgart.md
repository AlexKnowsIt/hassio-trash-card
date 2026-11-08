# Abfallwirtschaft Stuttgart (AWS) mit Home Assistant

Der AWS-Kalenderlink (iCal) wird in Home Assistant als Kalender eingebunden. Die Karte zeigt ihn an, Automatisierungen nutzen dieselbe Kalender-Entität. Es ist keine eigene Integration nötig.

## 1. Kalender abonnieren

1. Auf <https://www.stuttgart.de/organigramm/leistungen/abfallkalender> Adresse eingeben und den iCal-Link kopieren. Aus `webcal://` wird `https://`.
2. *Einstellungen → Geräte & Dienste → Integration hinzufügen → **Remote Calendar*** (ab HA 2025.7), Link einfügen. Es entsteht z. B. `calendar.abfall_stuttgart`.
3. Unter *Entwicklerwerkzeuge → Aktionen* `calendar.get_events` für diese Entität aufrufen und die exakten Titel (`summary`) der Termine notieren, z. B. `Restmüll`, `Bioabfall`, `Papier`, `Gelbe Säcke`.

Hinweis: Der AWS stellt die Termine des neuen Jahres erst ab Dezember bereit.

## 2. Karte

Die `pattern` müssen im Titel der Termine vorkommen (Groß-/Kleinschreibung egal). Anpassen, falls die Titel aus Schritt 3 anders lauten.

```yaml
type: custom:trash-card
entities:
  - calendar.abfall_stuttgart
next_days: 30
day_style: counter
drop_todayevents_from: '23:59:59'
pattern:
  - { type: waste,   label: Restmüll,    pattern: rest,   icon: mdi:trash-can,        color: dark-grey }
  - { type: organic, label: Bio,         pattern: bio,    icon: mdi:flower,           color: brown }
  - { type: paper,   label: Papier,      pattern: papier, icon: mdi:newspaper-variant-multiple-outline, color: indigo }
  - { type: recycle, label: Gelber Sack, pattern: gelb,   icon: mdi:recycle-variant,  color: amber }
```

## 3. Automatisierungen

Der Kalender-Trigger von Home Assistant feuert relativ zum Terminbeginn. Ganztägige Termine beginnen um 00:00, daher gilt:

- `offset: "+06:30:00"` → am Abholtag um 6:30 Uhr
- `offset: "-06:00:00"` → am Vortag um 18:00 Uhr

**Papiermüll: am Abholtag informieren**

```yaml
alias: Papiermüll heute
triggers:
  - trigger: calendar
    event: start
    entity_id: calendar.abfall_stuttgart
    offset: "+06:30:00"
conditions:
  - "{{ 'papier' in trigger.calendar_event.summary | lower }}"
actions:
  - action: notify.notify
    data:
      message: "Heute wird der Papiermüll geleert."
```

**Gelber Sack: am Vortag informieren**

```yaml
alias: Gelber Sack morgen
triggers:
  - trigger: calendar
    event: start
    entity_id: calendar.abfall_stuttgart
    offset: "-06:00:00"
conditions:
  - "{{ 'gelb' in trigger.calendar_event.summary | lower }}"
actions:
  - action: notify.notify
    data:
      message: "Morgen wird der Gelbe Sack abgeholt."
```

`notify.notify` durch den eigenen Dienst ersetzen (z. B. `notify.mobile_app_<gerät>`).
