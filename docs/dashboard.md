# Dashboard avec historique / Dashboard with history

Ce guide explique comment recréer un dashboard affichant l'historique des
températures min/max et l'état des fenêtres (Ouvrir/Fermer), en pur YAML
(`configuration.yaml`), sans créer de helpers (`input_select`, `input_text`,
etc.) ni dupliquer la liste de vos capteurs.

This guide shows how to recreate a dashboard displaying min/max temperature
history and the window state (Open/Close), in plain YAML
(`configuration.yaml`), without creating helpers (`input_select`,
`input_text`, etc.) or duplicating your sensor list.

---

## 1. Groupes de capteurs / Sensor groups

Dans `configuration.yaml`, définissez vos capteurs bruts une seule fois,
dans 2 groupes :

In `configuration.yaml`, define your raw sensors once, in 2 groups:

```yaml
group:
  capteurs_exterieurs:
    name: Capteurs extérieurs
    entities:
      - sensor.mon_capteur_exterieur_1
      - sensor.mon_capteur_exterieur_2
  capteurs_interieurs:
    name: Capteurs intérieurs
    entities:
      - sensor.mon_capteur_interieur_1
      - sensor.mon_capteur_interieur_2
```

Ces 2 groupes sont la **source unique** : tous les capteurs ci-dessous s'y
réfèrent. Si vous ajoutez/retirez un capteur physique, vous ne modifiez que
cette liste.

These 2 groups are the **single source of truth**: every sensor below
refers to them. If you add/remove a physical sensor, you only edit this
list.

Vous pouvez aussi sélectionner directement `group.capteurs_exterieurs` /
`group.capteurs_interieurs` comme `capteurs_exterieurs` / `capteurs_interieurs`
dans le blueprint, en éditant l'automation en **YAML** (le sélecteur visuel
filtre par `device_class`, donc il ne proposera pas un groupe générique — il
faut passer par l'éditeur YAML de l'automation pour ce champ précis).

You can also select `group.capteurs_exterieurs` / `group.capteurs_interieurs`
directly as `capteurs_exterieurs` / `capteurs_interieurs` in the blueprint, by
editing the automation in **YAML** (the visual selector filters by
`device_class`, so it won't offer a generic group — you need the automation's
YAML editor for this specific field).

---

## 2. État des fenêtres / Window state sensor

Toujours dans `configuration.yaml`, un capteur template calcule Ouvrir/Fermer
en reproduisant la même comparaison que le blueprint — sans créer de helper.
Il est "sticky" : pendant les périodes ambiguës (températures intérieures et
extérieures qui se chevauchent), il garde sa dernière valeur connue au lieu
de passer à `unknown`/`unavailable` :

Still in `configuration.yaml`, a template sensor computes Open/Close by
mirroring the same comparison as the blueprint — without creating a helper.
It's "sticky": during ambiguous periods (overlapping indoor/outdoor
temperature ranges), it keeps its last known value instead of flipping to
`unknown`/`unavailable`:

```yaml
template:
  - sensor:
      - name: "Fenêtres"
        unique_id: etat_fenetres
        icon: >
          {% if states('sensor.fenetres') == 'Ouvrir' %}
            mdi:window-open-variant
          {% else %}
            mdi:window-closed-variant
          {% endif %}
        state: >
          {% set ext = expand('group.capteurs_exterieurs') | map(attribute='state') | map('float', none) | reject('none') | list %}
          {% set int = expand('group.capteurs_interieurs') | map(attribute='state') | map('float', none) | reject('none') | list %}
          {% set precedent = states('sensor.fenetres') %}
          {% if ext | length > 0 and int | length > 0 %}
            {% if ext | max < int | min %}
              Ouvrir
            {% elif ext | min > int | max %}
              Fermer
            {% else %}
              {{ precedent if precedent not in ['unknown', 'unavailable'] else 'Fermer' }}
            {% endif %}
          {% else %}
            {{ precedent if precedent not in ['unknown', 'unavailable'] else 'Fermer' }}
          {% endif %}
```

`states('sensor.fenetres')` relit la propre valeur actuelle du capteur (par
son `entity_id`) plutôt que la variable `this`, pour rester compatible avec
toutes les versions de Home Assistant. Au tout premier calcul (aucune valeur
précédente), la valeur par défaut est `Fermer` — changez-la si vous préférez
`Ouvrir`.

`states('sensor.fenetres')` reads the sensor's own current value (via its
`entity_id`) instead of the `this` variable, to stay compatible with all
Home Assistant versions. On the very first computation (no previous value
yet), the default is `Fermer` (Close) — change it if you prefer `Ouvrir`
(Open).

⚠️ Ce capteur réagit instantanément aux températures, contrairement à la
notification du blueprint qui attend le délai configuré (`délai_minutes`,
10 min par défaut) avant de se déclencher. L'historique peut donc varier
légèrement de la notification réelle (le capteur peut changer d'état sans
qu'une notification soit envoyée).

⚠️ This sensor reacts instantly to temperatures, unlike the blueprint's
notification which waits for the configured delay (`délai_minutes`, 10 min
by default) before triggering. The history may therefore differ slightly
from the actual notification (the sensor can change state without a
notification being sent).

---

## 3. (Optionnel) Capteurs agrégés min/max / (Optional) Aggregated min/max sensors

Si vous voulez aussi 4 graphiques séparés (max/min extérieur, max/min
intérieur) plutôt qu'un simple indicateur d'état, ajoutez ces 4 capteurs
template, basés sur les mêmes groupes :

If you also want 4 separate graphs (outdoor max/min, indoor max/min) instead
of a simple state indicator, add these 4 template sensors, based on the same
groups:

```yaml
template:
  - sensor:
      - name: "Température max extérieure"
        unique_id: temperature_max_exterieure
        device_class: temperature
        state_class: measurement
        unit_of_measurement: "°C"
        state: >
          {% set vals = expand('group.capteurs_exterieurs') | map(attribute='state') | map('float', none) | reject('none') | list %}
          {{ vals | max | round(1) if vals | length > 0 else none }}
      - name: "Température min extérieure"
        unique_id: temperature_min_exterieure
        device_class: temperature
        state_class: measurement
        unit_of_measurement: "°C"
        state: >
          {% set vals = expand('group.capteurs_exterieurs') | map(attribute='state') | map('float', none) | reject('none') | list %}
          {{ vals | min | round(1) if vals | length > 0 else none }}
      - name: "Température max intérieure"
        unique_id: temperature_max_interieure
        device_class: temperature
        state_class: measurement
        unit_of_measurement: "°C"
        state: >
          {% set vals = expand('group.capteurs_interieurs') | map(attribute='state') | map('float', none) | reject('none') | list %}
          {{ vals | max | round(1) if vals | length > 0 else none }}
      - name: "Température min intérieure"
        unique_id: temperature_min_interieure
        device_class: temperature
        state_class: measurement
        unit_of_measurement: "°C"
        state: >
          {% set vals = expand('group.capteurs_interieurs') | map(attribute='state') | map('float', none) | reject('none') | list %}
          {{ vals | min | round(1) if vals | length > 0 else none }}
```

(Si vous avez déjà la section `template:` ci-dessus pour "Fenêtres", ajoutez
ces 4 entrées dans la même liste `sensor:` plutôt que de dupliquer la clé
`template:`.)

(If you already have the `template:` section above for "Fenêtres", add
these 4 entries to the same `sensor:` list rather than duplicating the
`template:` key.)

---

## 4. Cartes dashboard / Dashboard cards

```yaml
type: vertical-stack
cards:
  - type: tile
    entity: sensor.fenetres
    features_position: bottom
  - type: history-graph
    title: Fenêtres
    hours_to_show: 48
    entities:
      - entity: sensor.fenetres
  - type: history-graph
    title: Température max extérieure
    entities:
      - entity: sensor.temperature_max_exterieure
  - type: history-graph
    title: Température min extérieure
    entities:
      - entity: sensor.temperature_min_exterieure
  - type: history-graph
    title: Température max intérieure
    entities:
      - entity: sensor.temperature_max_interieure
  - type: history-graph
    title: Température min intérieure
    entities:
      - entity: sensor.temperature_min_interieure
```

⚠️ Si votre vue est de type `sections` (nouveau format de dashboard), une
**section** ne doit pas avoir `type: vertical-stack` (réservé aux cartes) —
mettez juste `cards:` directement sous la section, qui empile déjà
verticalement par défaut :

⚠️ If your view is of type `sections` (new dashboard format), a **section**
must not have `type: vertical-stack` (reserved for cards) — just put
`cards:` directly under the section, which already stacks vertically by
default:

```yaml
sections:
  - cards:
      - type: tile
        entity: sensor.fenetres
        features_position: bottom
      - type: history-graph
        title: Fenêtres
        hours_to_show: 48
        entities:
          - entity: sensor.fenetres
```

Après toute modification de `configuration.yaml` : **Réglages → Système →
Configuration YAML → Recharger → "Entités de modèle"** (et "Groupes" si
besoin), ou redémarrez Home Assistant.

After any change to `configuration.yaml`: **Settings → System → YAML
configuration → Reload → "Template entities"** (and "Groups" if needed), or
restart Home Assistant.
