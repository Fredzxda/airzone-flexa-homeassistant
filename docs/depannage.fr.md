# Dépannage — Airzone FLEXA 1.x

[Installation pas à pas](installation.fr.md)

## Aucune valeur Modbus ou entités indisponibles

1. Vérifier l'IP locale de l'EW41 dans `homeassistant/modbus.yaml` (l'adresse `192.168.1.100` n'est qu'un exemple).
2. Vérifier la connectivité LAN, le port TCP 502 et le mode de conversion Modbus TCP ↔ RTU.
3. Contrôler les paramètres série testés : 19200 bauds, 8E1, esclave 1, câblage RS485 A/B.
4. Examiner les journaux Home Assistant ; limiter le nombre de lectures simultanées avant de réessayer.
5. Revenir au capteur minimal en lecture seule du [guide EW41](ew41-configuration.fr.md).

## Erreur de configuration YAML

- Ne pas créer deux clés `modbus:` ou `template:` au même niveau.
- Les fichiers `modbus.yaml` et `templates.yaml` commencent par une liste (`- ...`) : les inclure sous les clés correspondantes.
- Fusionner les scripts Airzone dans le `scripts.yaml` existant ; conserver l'inclusion `script: !include scripts.yaml` si elle est déjà utilisée.
- Vérifier la configuration avant chaque redémarrage.

## Templates ou thermostats virtuels sans valeur

- Vérifier que les identifiants `sensor.`, `binary_sensor.` et `climate.` correspondent réellement aux entités créées dans votre Home Assistant.
- Les quatre thermostats HACS sont à créer dans l'interface ; le fichier JSON sert de référence et ne s'importe pas.
- Si le nombre de zones diffère, supprimer ou adapter les références aux zones absentes.

## Commande sans effet ou mauvaise zone

- Arrêter les écritures jusqu'à identification de la zone physique et du registre correct.
- Vérifier les registres en lecture et le comportement des commandes Tacto avant de relancer un script.
- Ne pas remplacer une valeur complète de registre d'état par une constante : les scripts doivent préserver les bits non concernés.
- Le mode global et l'activation individuelle des zones sont deux commandes distinctes.

## Horloge non synchronisée

- Vérifier le fuseau horaire et l'horloge Home Assistant, ainsi que la disponibilité de `sensor.az_diagnostic_reg_4`.
- L'automatisation s'exécute à 04:15 et ne rattrape pas automatiquement un déclenchement manqué.
- Voir [la procédure dédiée](synchronisation-horloge.fr.md). Le passage réel été/hiver n'est pas encore validé.

Si le comportement ne correspond pas à celui décrit, relever la révision de la centrale, la configuration EW41, les journaux Home Assistant et les valeurs lues **sans publier d'adresses IP privées, de secrets ni d'informations personnelles**.
