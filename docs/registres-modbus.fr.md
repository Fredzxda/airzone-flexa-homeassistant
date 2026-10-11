# Registres Modbus observés sur FLEXA 1.x

## Registres système

| Adresse | Observation | Statut |
| ---: | --- | --- |
| 0 | Mode global : 0 arrêt, 1 froid, 3 ventilation, 4 chauffage | Confirmé |
| 1 | -22016 | Signification inconnue |
| 2 | 0 | Signification inconnue |
| 3 | 0 | Signification inconnue |
| 4 | Valeur évolutive, hypothèse de compteur de fonctionnement | Non confirmé |
| 5 | 34 | Signification inconnue |
| 6 | 31 | Signification inconnue |
| 7 | 0 | Signification inconnue |
| 8 | 250 | Signification inconnue |
| 9 | 15, potentiellement indicateur de zones présentes | À confirmer |
| 10 | 0 | Signification inconnue |
| 11–12 | Lectures exploratoires | Non interprété |

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
