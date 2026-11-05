# Cheminée — Capteur de vitesse de refroidissement / Chimney — Cooling rate sensor

Le blueprint `chimney_heating` a besoin d'un capteur qui mesure la **vitesse de
refroidissement** de la température intérieure (en °/heure). Home Assistant fournit
l'intégration `derivative` pour ça.

The `chimney_heating` blueprint needs a sensor that measures the **cooling rate**
of the indoor temperature (in °/hour). Home Assistant provides the `derivative`
integration for this.

---

## Prérequis / Prerequisite

Avoir défini vos capteurs intérieurs dans un groupe (voir [docs/dashboard.md](dashboard.md)),
ou utiliser directement un capteur individuel comme source.

Having defined your indoor sensors in a group (see [docs/dashboard.md](dashboard.md)),
or use a single sensor directly as source.

---

## Capteur de dérivée / Derivative sensor

Dans `configuration.yaml` :

In `configuration.yaml`:

```yaml
sensor:
  - platform: derivative
    source: sensor.temperature_min_interieure
    name: "Vitesse refroidissement intérieur"
    unique_id: vitesse_refroidissement_interieur
    unit_time: h
    time_window: "00:30:00"
    round: 2
```

- **`source`** : le capteur de température intérieure à suivre — idéalement
  `sensor.temperature_min_interieure` (la plus froide) si vous l'avez créé,
  sinon n'importe quel capteur intérieur individuel.
- **`unit_time: h`** : taux en °/heure (ou °F/heure si vos capteurs sont en Fahrenheit).
- **`time_window: "00:30:00"`** : fenêtre glissante de 30 minutes pour lisser
  les fluctuations. Augmentez à `"01:00:00"` si votre température fluctue beaucoup.
- **`source`**: the indoor temperature sensor to track — ideally
  `sensor.temperature_min_interieure` (the coldest) if you created it,
  otherwise any single indoor sensor.
- **`unit_time: h`**: rate in °/hour (or °F/hour if your sensors use Fahrenheit).
- **`time_window: "00:30:00"`**: 30-minute rolling window to smooth fluctuations.
  Increase to `"01:00:00"` if your temperature fluctuates a lot.

Après modification de `configuration.yaml` : **Réglages → Système → Configuration
YAML → Recharger → "Entités de modèle"**, ou redémarrez Home Assistant.

After modifying `configuration.yaml`: **Settings → System → YAML configuration →
Reload → "Template entities"**, or restart Home Assistant.

---

## Vérification / Verification

Dans **Outils de développement → États**, cherchez `sensor.vitesse_refroidissement_interieur`.

- Valeur **négative** (ex. `-0.8`) → la température descend à 0.8°/h ✓
- Valeur **positive** → la température monte (chauffage en cours, cheminée allumée)
- Valeur **0 ou proche** → température stable

In **Developer Tools → States**, search for `sensor.vitesse_refroidissement_interieur`.

- **Negative** value (e.g. `-0.8`) → temperature dropping at 0.8°/h ✓
- **Positive** value → temperature rising (heating on, fireplace lit)
- **0 or near 0** → temperature stable

---

## Logique d'anticipation / Anticipation logic

Le blueprint calcule :

The blueprint computes:

```
temps_restant = (temp_actuelle - temp_confort) / |vitesse|
```

Si `temps_restant ≤ temps_de_chauffe`, la notification part.

If `temps_restant ≤ warm_up_time`, the notification fires.

**Exemple / Example** :
- Temp intérieure / Indoor temp : 20°C
- Vitesse / Rate : -0.8°C/h
- Confort / Comfort : 19°C
- Temps de chauffe / Warm-up : 45 min (0.75 h)

→ `(20 - 19) / 0.8 = 1.25 h` restantes → pas encore / not yet  
→ Notification quand / fires when temp atteint / reaches `19 + 0.8 × 0.75 = 19.6°C`
