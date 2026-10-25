# Airzone FLEXA 1.x — hypothèse sur le registre système Modbus 20

Date des essais : **25 octobre 2026** (heure locale Europe/Paris).

## Hypothèse issue des essais

La valeur du registre système **20** semble représenter **1280 + le nombre de registres motorisés sollicités / ouverts**. Ce comptage ne doit pas être confondu avec le nombre de zones en mode `auto` : une zone peut être en `auto` sans demande thermique.

Configuration connue :
- Salon : **2** registres motorisés.
- Chambre Tamara : **1** registre.
- Chambre Briséïs : **1** registre.
- Chambre des parents : **1** registre.

## Mesures et actions observées

| Action / situation | Valeur du registre 20 | Différence | Interprétation |
| --- | ---: | ---: | --- |
| Aucune demande active | 1280 (`0x0500`) | — | Base |
| Salon activé, demande de chauffage (10:08:23) | 1282 (`0x0502`) | +2 | Deux volets du Salon |
| Tamara ajoutée au Salon (10:10:24) | 1283 (`0x0503`) | +1 | Un volet de Tamara |
| Salon désactivé (10:12:37) | 1281 (`0x0501`) | -2 | Retrait des deux volets du Salon |
| Tamara désactivée (10:13:37) | 1280 (`0x0500`) | -1 | Retour à zéro demande |

Autres observations corroborantes :
- Le 25/10 à 09:49:55, Briséïs seule en demande probable : `1280 → 1281` ; retour à `1280` à 09:59:02.
- Le 24/10 à 17:41:17, Tamara seule en fonctionnement : `1280 → 1281`.
- Le 23/10 à 07:53, activation du Salon : `1280 → 1282`.

## Décodage provisoire

```text
nombre_de_volets_sollicites = registre_20 - 1280
```

**Attention : hypothèse fortement étayée, pas une spécification constructeur.** Les valeurs de base `0x0500` (bits 8 et 10) ne sont pas expliquées. Les essais ne prouvent pas que les volets sont *physiquement* ouverts ni que ce registre pilote directement la puissance de la PAC ou la ventilation. Il peut refléter des commandes de volets ou une information de charge. Le registre 20 n'a pas été identifié dans le tableau Modbus constructeur consulté.

## Vérifications restantes

1. Ajouter Briséïs et/ou Parents aux demandes pour vérifier les valeurs `1284` et `1285`.
2. Vérifier le comportement en mode refroidissement.
3. Comparer, si disponibles, les positions réelles des registres motorisés avec ce compteur.
4. Identifier le sens de la base `1280` et la réaction aux pertes de communication / redémarrages.

Aucune modification de configuration Home Assistant n'est requise pour cette documentation.
