# Airzone FLEXA 1.x - Integration locale Home Assistant

[English README](README.md) | [Configuration de l'EW41](docs/ew41-configuration.fr.md)

Ce projet communautaire permet de piloter une ancienne installation Airzone FLEXA 1.x avec Home Assistant **sans passerelle cloud Airzone**.

## Pourquoi ?

- **100 % local** : la communication entre Home Assistant et FLEXA ne depend d'aucun cloud.
- **Economique** : une passerelle serie/Ethernet **EW41**, d'un prix indicatif d'environ **30 EUR**.
- **Conservation du materiel** : centrale, thermostats radio Tacto et registres motorises existants.
- **Pilotage bidirectionnel** : les commandes depuis les thermostats restent utilisables.
- Google Home est facultatif et n'est pas necessaire au fonctionnement local.

## Architecture

```text
Home Assistant -- Modbus TCP (LAN) -- EW41 (~30 EUR)
                                        |
                                     RS485/RTU
                                        |
                                 Airzone FLEXA 1.x
                                        |
                             Thermostats Tacto / zones
```

## Fonctionnalites testees

Sur une installation de reference a quatre zones : activation des zones, consignes et temperatures, mode nuit, lecture des limites de temperature et selection du mode global (arret, froid, ventilation, chauffage).

## Registres observes

| Fonction | Zone 1 | Zone 2 | Zone 3 | Zone 4 |
| --- | ---: | ---: | ---: | ---: |
| Etat de zone | 256 | 512 | 768 | 1024 |
| Consigne min. | 257 | 513 | 769 | 1025 |
| Consigne max. | 258 | 514 | 770 | 1026 |
| Consigne | 259 | 515 | 771 | 1027 |
| Temperature ambiante | 266 | 522 | 778 | 1034 |

Mode global : registre 0. Echelle des temperatures : 0,1. Bits d'etat observes : bit 0 = nuit, bit 1 = programmation desactivee, bit 2 = zone active. **Preserver les autres bits lors des ecritures.**

## Installation

Commencer par le [guide de configuration EW41](docs/ew41-configuration.fr.md). Les fichiers YAML generiques sont encore en preparation ; ne pas recopier aveuglement les adresses ou les ecritures sur une autre generation de centrale.

## Etat et securite

Projet en cours de documentation, **non encore pret a installer en un clic**. Ne pas exposer le port TCP 502 sur Internet ; limiter les interrogations du bus et tester les lectures avant toute ecriture. Projet independant, non affilie a Airzone.

## Licence

GNU GPL v3.0 - voir [LICENSE](LICENSE).

## Fichiers de configuration et documentation

[Exemple de `configuration.yaml` et inclusions](homeassistant/configuration.example.yaml) : montre comment raccorder les fichiers Modbus, templates et scripts sans remplacer la configuration existante.

Pour reproduire l'installation, consulter les fichiers suivants dans cet ordre :

1. [Configuration de la passerelle EW41](docs/ew41-configuration.fr.md) : câblage RS485, paramètres 19200 8E1, réseau local et port 502.
2. [Registres Modbus système et zones](docs/registres-modbus.fr.md) : valeurs observées, registres confirmés et hypothèses.
3. [Configuration Modbus et capteurs](homeassistant/modbus.yaml) : fragment à intégrer dans `configuration.yaml` sous `modbus:`.
4. [Capteurs templates et sélecteur de mode global](homeassistant/templates.yaml) : fragment à intégrer dans `configuration.yaml` sous `template:`.
5. [Scripts de pilotage](homeassistant/scripts.yaml) : à ajouter à `scripts.yaml`.
6. [Guide de l'intégration HACS Template Climate](docs/template-climate.fr.md) et [options des quatre thermostats](homeassistant/template-climate-options.json) : à configurer via l'interface Home Assistant.

7. [Synchronisation quotidienne de l'horloge Tacto](docs/synchronisation-horloge.fr.md) : automatisation validée pour corriger une dérive, exécutée à 04 h 15 ; passage été/hiver encore à valider.

**Important :** les fichiers sont des exemples anonymisés extraits de l'installation de référence, pas un package prêt à installer. Fusionner les entrées avec les clés YAML existantes ; ne pas dupliquer `modbus:` ou `template:`. Tester les lectures avant d'activer les écritures.
