# Airzone FLEXA 1.x — registre 0 et passage Froid vers Chauffage

Relevés du 29 octobre au 1er novembre 2026. Toutes les heures ci-dessous sont en
**Europe/Paris** (UTC+1 pour ces dates). Installation de référence : FLEXA ancienne
génération, thermostat Tacto maître, quatre zones. Observations locales, sans
généralisation aux autres générations ni assimilation aux codes API/BACnet.

## Résultat principal

Le passage Froid vers Chauffage demandé par Modbus est suivi d'une lecture `2`,
puis d'un retour à `4` sans nouvelle commande volontaire de notre part. Le même
phénomène existe en écrivant directement le registre, sans les scripts Airzone.
Le Tacto maître continue d'afficher Chaud pendant le relevé `2` (confirmation
utilisateur lors du premier essai direct du 1er novembre).

Le délai **observé par Home Assistant n'est pas fixe** : cinq séquences ont un
écart d'environ 30 secondes entre première lecture `2` et première lecture `4`,
contre environ 125 secondes pour une autre. Ce n'est pas une mesure précise de
la durée interne : le capteur Modbus est interrogé normalement toutes les 30 s,
avec des lectures forcées au cours des essais. Une période courte peut donc
passer inaperçue ; des lectures retardées peuvent aussi modifier l'écart observé.
Les raisons de la séquence et de sa durée variable ne sont pas établies.

## Commande du dashboard : chaîne vérifiée

```text
carte Air Zone
  -> script.airzone_basculer_froid_chaud
  -> select.airzone_mode_global (option Chauffage)
  -> script.airzone_mode_global (mode = 4)
  -> modbus.write_register (adresse 0, valeur = 4 dans ces essais)
```

La trace du 31 octobre à 14:54:54 prouve une **écriture de 4**, pas de 2.
Le script calcule `(ancienne_valeur // 512) * 512 + nouveau_mode`, pour préserver
les bits supérieurs à la zone des bits 0 à 8 ; l'ancienne valeur était `1`, donc
la valeur effectivement écrite était `4`. Il attend 500 ms puis force une lecture
du capteur de mode, sans attendre une confirmation du résultat demandé.

Le sélecteur et le capteur textuel de l'installation reconnaissent `0`, `1`, `3`,
`4`, mais pas `2`. Une lecture `2` donne donc un sélecteur `unknown` et une icône
inconnue dans les cartes de mode et de zones qui se basent sur ce sélecteur.
Un nouveau clic pendant cet état est ignoré par la condition initiale du script.
Les scripts se sont terminés sans erreur d'écriture dans les traces examinées.

Le masque des zones présentes est resté `15` pendant les essais étudiés. Cela
ne correspond pas à une disparition des quatre zones. Des erreurs Modbus de
lecture existent par ailleurs, mais les journaux agrégés ne permettent pas de
les attribuer au clic précis. Ne pas confondre `2` reçu et `unavailable` dû à une
lecture en échec.

## Mesures des passages 2 vers 4

Durée = première publication de `4` moins première publication de `2` dans
l'historique de `sensor.airzone_mode_global`. Les valeurs sont conservées dans
[le CSV de mesures](data/register-0-transitions-2026-11-01.csv).

| Date | Première lecture 2 | Première lecture 4 | Écart observé | Origine connue |
| --- | --- | --- | ---: | --- |
| 29/10/2026 | 12:12:59.896 | 12:13:29.931 | 30,035 s | Historique, origine non vérifiée |
| 29/10/2026 | 17:02:56.857 | 17:03:26.940 | 30,082 s | Historique, origine non vérifiée |
| 29/10/2026 | 19:19:43.890 | 19:20:13.946 | 30,057 s | Dashboard, traces disponibles |
| 31/10/2026 | 14:54:54.986 | 14:57:00.299 | 125,312 s | Dashboard, traces disponibles |
| 01/11/2026 | 11:40:25.182 | 11:40:55.229 | 30,047 s | Écriture directe, commandes espacées d'environ 5 s |
| 01/11/2026 | 11:44:49.866 | 11:45:19.914 | 30,048 s | Écriture directe, commandes espacées de 120 s |

Les heures ultérieures auxquelles l'utilisateur a demandé une nouvelle lecture
ne sont **pas** les heures du retour effectif enregistré : celui-ci était déjà
visible dans l'historique. En particulier, le second essai n'est pas resté à `2`
pendant onze minutes : le retour `4` est enregistré à 11:45:19.914, avant notre
vérification manuelle à 11:55:47.

## Comparaison avec le Tacto maître

Le 1er novembre, l'utilisateur a sélectionné Chaud, puis Froid, puis Chaud depuis
le Tacto maître. Aucune exécution du script de mode HA pendant cette manipulation.

| Heure de publication HA | Valeur | Interprétation locale |
| --- | ---: | --- |
| Avant l'essai | 4 | Chauffage |
| 11:35:15.860 | 1 | Froid |
| 11:35:45.907 | 4 | Chauffage |

Aucune valeur `2` n'a été enregistrée pour ce cycle. La lecture toutes les 30 s
ne permet cependant **pas d'exclure un passage bref par 2**. Ce résultat ne prouve
pas que le Tacto et Modbus utilisent des temporisations différentes.

## Essais directs, sans script Airzone

Service utilisé pour les écritures : `modbus.write_register`, hub local
`airzone_flexa`, esclave `1`, adresse `0`, valeur scalaire `1` puis `4`.
Lectures forcées par `homeassistant.update_entity` sur le capteur de mode, puis
consultation de son état. Les résultats sont donc ceux de la chaîne Modbus de HA,
pas une capture des trames TCP/RTU sur le fil. Le polling normal reste actif.

### Essai 1 : environ 5 s entre les commandes

- Vers 11:40:12 : écriture `1` (Froid), service terminé sans erreur.
- Environ 5 s après le retour du service : écriture `4` (Chauffage).
- Après une nouvelle attente de 5 s : lecture `2`, publiée à 11:40:25.182.
- L'utilisateur indique que le Tacto affiche toujours Chaud.
- Retour spontané enregistré à `4` à 11:40:55.229 ; confirmé par une nouvelle
  lecture à 11:41:43. Aucune commande de correction envoyée entre-temps.

### Essai 2 : deux minutes entre les commandes

- 11:42:43 : écriture `1` (Froid).
- 11:43:41 : lecture intermédiaire `1`, Froid confirmé.
- 11:44:43 environ : écriture `4`, après 120 s mesurées depuis le retour de la
  commande Froid. Les temps de transport du service peuvent ajouter un écart.
- Attente de 5 s puis lecture : `2`, publiée à 11:44:49.866.
- Retour enregistré à `4` à 11:45:19.914 ; lecture manuelle confirmant `4` à
  11:55:47. Aucun nouvel ordre volontaire de mode entre les deux lectures.

Attendre deux minutes entre Froid et Chauffage n'a donc pas évité le relevé `2`.
Cela ne teste pas toutes les durées possibles ni toutes les conditions de demande.

## Recherche documentaire : ne pas mélanger les interfaces

Le manuel Airzone commun `MI_AZ6_MODBUS_A4_MUL.pdf`, partie anglaise page 25,
décrit pour le registre système `00H` : bits 0–8 = mode, bits 9–10 = vitesse,
puis d'autres indicateurs système. Il mentionne notamment `2` pour le chauffage
radiant/plancher et `4` pour le chauffage par air. La table présente également
des variantes selon le système. Ce sont des codes documentés, **pas une preuve
que cette installation ancienne possède un étage radiant**, ni une définition
de `2` comme transition. [Manuel Airzone hébergé par Loxone](https://api.library.loxone.com/downloader/file/83/MI_AZ6_MODBUS_A4_MUL.pdf)

À l'inverse, Aidoo utilise le registre de mode `3` : `2` = Froid, `3` = Chaud,
`4` = Ventilation. Cette table ne doit pas être appliquée au registre système
`0` de l'installation FLEXA étudiée. [Manuel officiel Aidoo Modbus, page 9](https://doc.airzonecloud.com/Documentation/AZ6/AI6/MI_AZAI6_MODBUS_A4_MUL.pdf)

La passerelle BACnet possède encore son propre codage des modes. L'hypothèse
d'un fonctionnement commun aide à rechercher, mais ne justifie pas un décodage
numérique commun à toutes les gammes/interfaces.
[Manuel officiel BACnet](https://doc.airzonecloud.com/Documentation/AZ6/X6/MI_AZX6_BACNET_MUL.pdf)

## Conclusions et suites à vérifier

Établi : la commande transmise par le script vaut `4` ; le retour `2` a été
reproduit sans ce script ; les zones sont restées déclarées présentes ; le mode
revient à `4`. Notre représentation des valeurs lues ne traite pas `2`.

Non établi : raison du retour `2`, rôle d'un étage de chauffage, temporisation
interne fixe, acteur provoquant le retour à `4`, influence éventuelle d'une lecture
très proche de l'écriture ou de la passerelle TCP/RTU. Ne pas remplacer la commande
de chauffage `4` par `2` sur cette seule base et ne pas qualifier le phénomène de
transition normale documentée.

Prochain protocole possible, **pas exécuté** : enregistrer les trames et les lectures
horodatées à intervalle plus fin pendant un cycle commandé au Tacto, puis un cycle
Modbus comparable ; vérifier le code exact de la platine, son firmware et la
configuration des étages. Toute augmentation de polling doit rester limitée pour
ne pas surcharger le bus. Aucun réglage permanent de mode/décodage n'a été modifié
pour rédiger ce document.
