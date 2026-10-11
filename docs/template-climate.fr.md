# Intégration HACS Template Climate

Dépôt : https://github.com/holdestmade/template_climate

Installer cette intégration via HACS. Elle est distincte du Template Climate natif de Home Assistant 2026.10.

Créer **quatre entrées Template Climate** via l'interface Home Assistant. Les options de chacune, extraites de l'installation de référence et anonymisées, figurent dans [`homeassistant/template-climate-options.json`](../homeassistant/template-climate-options.json). Le JSON est une **référence des champs à saisir dans l'interface**, pas un bloc YAML à coller dans `configuration.yaml`.

Les entités Modbus `climate.clim_zone_1` à `climate.clim_zone_4` fournissent température et consigne ; les entités HACS assurent la marche/arrêt, le preset nuit et l'affichage. Adapter les `entity_id` aux identifiants créés sur votre installation.

`auto` signifie ici **zone activée** : le mode global chaud/froid/ventilation est géré séparément via `select.airzone_mode_global`.

**Correction dans l'exemple :** l'icône de la zone 3 se réfère à la zone 3 (la configuration d'origine faisait référence à la zone 2). Aucun changement n'a été appliqué au Home Assistant de référence.
