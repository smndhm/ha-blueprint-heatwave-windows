# Fenêtres Canicule / Heatwave Windows

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fsmndhm%2Fha-blueprint-heatwave-windows%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fheatwave_windows.yaml)

---

**FR** — Blueprint Home Assistant pour vous alerter d'ouvrir ou fermer vos fenêtres pendant une canicule, en comparant les températures extérieures et intérieures via plusieurs capteurs.

**EN** — Home Assistant Blueprint to alert you to open or close your windows during a heatwave, by comparing indoor and outdoor temperatures across multiple sensors.

---

## Logique de déclenchement / Trigger logic

| Trigger | Condition | Action |
|---------|-----------|--------|
| **Ouvrir / Open** | `max(ext) < min(int)` — Il fait plus frais dehors que dedans / It's cooler outside than inside | Notification : ouvrir les fenêtres |
| **Fermer / Close** | `max(ext) > min(int)` — Il fait plus chaud dehors que dedans / It's warmer outside than inside | Notification : fermer les fenêtres |

La logique est conservatrice : on n'ouvre que quand **tous** les capteurs extérieurs indiquent une température inférieure à **tous** les capteurs intérieurs. Les capteurs indisponibles (`unavailable`, `unknown`) sont ignorés automatiquement.

The logic is conservative: windows are only suggested open when **all** outdoor sensors read lower than **all** indoor sensors. Unavailable sensors (`unavailable`, `unknown`) are automatically ignored.

---

## Installation

### Via le bouton d'import (recommandé / recommended)

Cliquez sur le badge en haut de cette page / Click the badge at the top of this page.

### Manuellement / Manually

1. Copiez l'URL du fichier raw `heatwave_windows.yaml`
2. Dans Home Assistant : **Paramètres → Automatisations → Blueprints → Importer un blueprint**
3. Collez l'URL et confirmez

---

## Configuration

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| **Capteurs extérieurs** | Un ou plusieurs capteurs `device_class: temperature` en extérieur | — |
| **Capteurs intérieurs** | Un ou plusieurs capteurs `device_class: temperature` en intérieur | — |
| **Service de notification** | Service à appeler, ex: `notify.mobile_app_mon_telephone` | — |
| **Délai** | Durée (en minutes) pendant laquelle la condition doit être vraie avant déclenchement | 10 min |
| **Langue** | Langue des notifications : Français ou English | `fr` |

---

## Contributing

Les langues supplémentaires sont les bienvenues ! Pour ajouter une langue :

1. Forkez ce dépôt
2. Dans `blueprints/automation/heatwave_windows.yaml`, ajoutez votre langue dans le sélecteur `langue` et les templates de messages
3. Ouvrez une Pull Request

Additional languages are welcome! To add a language:

1. Fork this repository
2. In `blueprints/automation/heatwave_windows.yaml`, add your language to the `langue` selector and the message templates
3. Open a Pull Request

---

## License

MIT — see [LICENSE](LICENSE)
