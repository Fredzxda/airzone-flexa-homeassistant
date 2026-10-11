# Registres Modbus observés sur FLEXA 1.x

## Registres système

| Adresse | Observation | Statut |
| ---: | --- | --- |
| 0 | Mode global : 0 arrêt, 1 froid, 3 ventilation, 4 chauffage | Confirmé |
| 1 | -22016 | Signification inconnue |
| 2 | 0 | Signification inconnue |
| 3 | 0 | Signification inconnue |
| 4 | Jour et heure, encodés sur 16 bits (vérifié sur Tacto maître) | Confirmé en lecture |
| 5 | 34 | Signification inconnue |
| 6 | 31 | Signification inconnue |
| 7 | 0 | Signification inconnue |
| 8 | 250 | Signification inconnue |
| 9 | 15, potentiellement indicateur de zones présentes | À confirmer |
| 10 | 0 | Signification inconnue |
| 11–12 | Lectures exploratoires | Non interprété |

## Registre 4 — jour et heure du Tacto maître

La documentation Modbus Airzone indique : bits 0–2 = jour (1 lundi à 7 dimanche, 0 sans heure), bits 3–7 = heure (0–23), bits 8–13 = minutes (0–59), bit 15 = changement d'heure automatique ; bit 14 non documenté.

Décodage de la valeur brute `v` (traiter un éventuel `int16` signé comme non signé avec `v & 0xFFFF`) :

```text
jour    = v & 0x0007
heure   = (v >> 3) & 0x001F
minute  = (v >> 8) & 0x003F
heure_auto = (v >> 15) & 1
```

**Vérification réelle :** après réglage du Tacto sur samedi (6) vers 13 h 02, lecture du registre 4 à 12 h 03 côté Home Assistant : `366` = `0x016E` = samedi (6), 13 h 01. L'écart d'une minute peut venir du décalage entre l'affichage et le rafraîchissement Modbus (60 s). L'historique montrait également un retour des minutes de 59 à 0 avec progression du champ heure. Cela confirme le décodage en lecture sur cette installation FLEXA 1.x.

**Évolution envisagée (non implémentée) :** exposer un capteur template Home Assistant donnant le jour et l'heure de la centrale, comparer avec l'heure locale Home Assistant et signaler une dérive. Une synchronisation automatique nécessiterait une écriture du registre 4 (documenté L/E), en préservant le bit 15 et le bit 14 non documenté ; elle devra être testée séparément, avec contrôle du jour de semaine, fuseau horaire, heure d'été et fréquence des écritures. Aucune écriture automatique n'est activée dans ce projet.

Pour écrire le registre 0, préserver les bits supérieurs (voir `airzone_mode_global` dans le fichier des scripts).

## Registres de zones

| Fonction | Zone 1 | Zone 2 | Zone 3 | Zone 4 |
| --- | ---: | ---: | ---: | ---: |
| État de zone | 256 | 512 | 768 | 1024 |
| Consigne min. | 257 | 513 | 769 | 1025 |
| Consigne max. | 258 | 514 | 770 | 1026 |
| Consigne | 259 | 515 | 771 | 1027 |
| Température ambiante | 266 | 522 | 778 | 1034 |
| Registre exploratoire +13 | 269 | 525 | 781 | 1037 |

Températures en dixièmes de degré. Dans le registre d'état : bit 0 = nuit, bit 1 = programmations désactivées, bit 2 = zone activée. **Préserver les autres bits lors des écritures.** Le diagnostic de batterie faible via +13 n'est pas confirmé sur cette génération.
