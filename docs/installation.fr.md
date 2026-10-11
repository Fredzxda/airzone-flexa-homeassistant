# Installation pas à pas — Airzone FLEXA 1.x et Home Assistant

[Retour au README](../README.fr.md)

Ce guide décrit une installation **testée sur une seule centrale FLEXA 1.x à quatre zones**. Ce n'est pas une intégration prête à installer en un clic. Les écritures Modbus sur une autre révision de centrale doivent être validées avant utilisation.

## 1. Préparer et sauvegarder

- Disposer d'une centrale FLEXA 1.x accessible sur son bus RS485, d'une passerelle EW41 et d'un Home Assistant fonctionnel.
- Sauvegarder `configuration.yaml`, `scripts.yaml` et toute configuration existante.
- Repérer le nombre de zones et les commandes Tacto. Ne pas supposer que la zone 1 correspond à une pièce particulière ni qu'elle est maître sur toutes les installations.
- Installer l'intégration personnalisée HACS [Template Climate](https://github.com/holdestmade/template_climate) pour les thermostats virtuels, après avoir validé les lectures Modbus.

## 2. Raccorder et paramétrer l'EW41

Suivre [le guide EW41](ew41-configuration.fr.md) : câblage A/B, alimentation conforme à la notice, Modbus TCP vers RTU, **19200 bauds, 8E1, esclave 1, port 502** sur l'installation testée. Réserver une adresse IP locale à la passerelle ; ne pas exposer le port 502 à Internet.

## 3. Tester la communication en lecture seule

Commencer par l'exemple minimal du [guide EW41](ew41-configuration.fr.md), en remplaçant `192.168.1.100` par **l'IP de votre EW41**. Vérifier la configuration Home Assistant, redémarrer et observer `sensor.airzone_mode_global`. Sa valeur doit correspondre au mode affiché par la commande physique (0 arrêt, 1 froid, 3 ventilation, 4 chauffage sur la centrale testée).

**Ne pas activer les scripts d'écriture tant que cette étape n'est pas concluante.** Si la lecture échoue, consulter le [dépannage](depannage.fr.md).

## 4. Installer les entités Modbus

Le fichier [`homeassistant/modbus.yaml`](../homeassistant/modbus.yaml) contient une **liste de hubs** à placer sous la clé `modbus:`, et non un `configuration.yaml` complet.

Suivre [`configuration.example.yaml`](../homeassistant/configuration.example.yaml). Si vous utilisez `modbus: !include modbus.yaml`, remplacer l'exemple minimal précédent plutôt que de déclarer deux hubs `airzone_flexa`. Modifier l'adresse IP de l'EW41. Après vérification YAML et redémarrage, confirmer la disponibilité des entités `climate.clim_zone_N` et `sensor.airzone_zone_N_etat_brut`.

Le fichier est prévu pour **quatre zones**. Pour moins de zones, supprimer les entrées `climates` et `sensors` des zones absentes, puis les templates et thermostats correspondants. Pour plus de quatre zones, **ne pas extrapoler sans validation** les adresses et capacités de la centrale.

## 5. Installer les templates

Le fichier [`homeassistant/templates.yaml`](../homeassistant/templates.yaml) est une **liste de blocs** à inclure sous `template:`. Fusionner avec les templates existants sans dupliquer la clé. Vérifier les entités `binary_sensor.airzone_zone_N_activee`, `binary_sensor.airzone_zone_N_mode_nuit` et `select.airzone_mode_global`. Les identifiants réels peuvent différer selon les entités déjà créées.

## 6. Installer les scripts

Fusionner [`homeassistant/scripts.yaml`](../homeassistant/scripts.yaml) avec votre `scripts.yaml` existant ; ne pas écraser les autres scripts. Les scripts réalisent des écritures sur les registres d'état en conservant les bits non ciblés. **Ne tester les commandes qu'après avoir vérifié les valeurs et la correspondance des zones sur votre installation.**

Tester d'abord une seule zone, en présence de la commande physique : marche/arrêt, consigne, puis mode nuit. Vérifier le retour d'état dans les deux sens. Le mode global est une commande système distincte ; vérifier qu'il est autorisé par la centrale.

## 7. Créer les thermostats virtuels HACS

Suivre [le guide Template Climate](template-climate.fr.md) et les [options JSON de référence](../homeassistant/template-climate-options.json). Créer **une instance HACS par zone** depuis *Paramètres → Appareils et services* ; **le JSON ne s'importe pas directement**. Il ne remplace pas les entités `climate.clim_zone_N` de Modbus.

## 8. Horloge Tacto (facultatif)

La [synchronisation de l'horloge](synchronisation-horloge.fr.md) est une automatisation distincte, qui utilise `sensor.az_diagnostic_reg_4`. Elle a été validée lors d'une correction manuelle du jour et de l'heure ; son comportement pendant une vraie transition été/hiver reste à confirmer.

## 9. Vérifications finales

- La température et la consigne lues sont plausibles.
- Chaque zone correspond à la bonne commande physique.
- Les changements depuis le Tacto sont visibles dans Home Assistant.
- Les changements depuis Home Assistant sont visibles sur le Tacto.
- Les autres zones ne changent pas de façon inattendue.
- Aucun message d'erreur Modbus récurrent n'apparaît.

En cas d'anomalie, arrêter les essais d'écriture et consulter [le dépannage](depannage.fr.md).
