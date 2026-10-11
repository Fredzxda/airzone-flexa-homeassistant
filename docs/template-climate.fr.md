# Intégration HACS Template Climate

Dépôt : https://github.com/holdestmade/template_climate

Installer cette intégration via HACS. Elle est distincte du Template Climate natif de Home Assistant 2026.10.

Créer **quatre entrées Template Climate** via l'interface Home Assistant. Les options de chacune, extraites de l'installation de référence et anonymisées, figurent dans [`homeassistant/template-climate-options.json`](../homeassistant/template-climate-options.json). Ce fichier JSON contient la **configuration des quatre instances de l'intégration personnalisée HACS `holdestmade/template_climate`** (une instance par zone). Il présente les options utilisées dans l'installation de référence : entités sources, commandes, consignes, modes et presets. **Il ne s'importe pas directement dans Home Assistant** et ne doit pas être collé dans `configuration.yaml` : il sert de **référence pour renseigner manuellement les options de chaque instance** depuis l'interface Home Assistant, dans *Paramètres → Appareils et services*.

Les entités Modbus `climate.clim_zone_1` à `climate.clim_zone_4` fournissent température et consigne ; les entités HACS assurent la marche/arrêt, le preset nuit et l'affichage. Adapter les `entity_id` aux identifiants créés sur votre installation.

`auto` signifie ici **zone activée** : le mode global chaud/froid/ventilation est géré séparément via `select.airzone_mode_global`.

## Exemple de configuration d'une zone

1. Vérifier d'abord la présence de `climate.clim_zone_1`, `sensor.airzone_zone_1_etat_brut`, `binary_sensor.airzone_zone_1_activee` et des scripts `script.airzone_zone_marche_arret` / `script.airzone_zone_mode_nuit`.
2. Dans *Paramètres → Appareils et services*, ajouter une entrée de l'intégration HACS **Template Climate**.
3. Utiliser l'objet `zones[0].options` du [JSON de référence](../homeassistant/template-climate-options.json) pour renseigner les champs de cette instance (nom, modèles de température, mode, preset, limites et actions). Selon la version de l'intégration, les libellés des champs peuvent différer.
4. Pour la zone 1, les températures courante et cible proviennent de `climate.clim_zone_1`. Le mode `auto` correspond à la **zone activée**, et `off` à la zone désactivée : ce n'est pas le mode chaud/froid de la pompe à chaleur.
5. L'action de changement de mode appelle `script.airzone_zone_marche_arret` avec `zone: 1` ; le preset `sleep` appelle `script.airzone_zone_mode_nuit`. Le changement de consigne passe par `climate.set_temperature` sur `climate.clim_zone_1`.
6. Vérifier d'abord l'affichage et les valeurs ; tester ensuite les commandes sur une seule zone après validation des registres.

Pour les autres zones, utiliser respectivement `zones[1].options`, `zones[2].options` et `zones[3].options`, en vérifiant **chaque** `entity_id` et numéro de zone. Ne pas créer d'instance pour une zone absente. Le JSON contient des expressions Jinja et des actions de référence : il n'est **pas** un fichier de configuration importable automatiquement.

Pour un ordre de mise en service sécurisé, suivre le [guide d'installation](installation.fr.md). Pour les entités indisponibles ou les commandes sans effet, voir le [dépannage](depannage.fr.md).
