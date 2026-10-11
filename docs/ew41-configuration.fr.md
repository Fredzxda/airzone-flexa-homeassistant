# Configuration de la passerelle EW41 avec Airzone FLEXA 1.x

[Retour a la documentation francaise](../README.fr.md)

## Objectif

Relier Home Assistant a une ancienne centrale FLEXA **sans cloud** au moyen d'une passerelle EW41 serie/Ethernet (prix indicatif d'environ 30 EUR).

```text
Home Assistant -- Ethernet / Modbus TCP -- EW41 -- RS485 / Modbus RTU -- FLEXA
```

## Reglages valides sur l'installation de reference

| Parametre | Valeur |
| --- | --- |
| Protocole cote Home Assistant | Modbus TCP |
| Port TCP de l'EW41 | 502 |
| Interface serie | RS485 |
| Vitesse | 19200 bauds |
| Bits de donnees | 8 |
| Parite | Paire (Even) |
| Bits de stop | 1 |
| Adresse esclave Modbus | 1 |

**Important :** ce sont les reglages utilises sur la centrale testee, pas une garantie pour toutes les revisions FLEXA ou EW41. Les noms des menus peuvent varier selon le firmware de la passerelle.

## Raccordement

1. Mettre les equipements hors tension avant de modifier le cablage.
2. Reperer la paire de communication **A et B** sur le bornier de communication de la centrale FLEXA, et les bornes RS485 correspondantes de l'EW41.
3. Raccorder A a A et B a B en respectant le reperage du materiel. Ne pas confondre les bornes d'alimentation + / - avec les bornes de donnees A / B.
4. Alimenter l'EW41 conformement a **sa propre notice et sa version materielle**. Ne pas supposer que l'alimentation disponible sur la centrale convient sans verification.
5. Raccorder l'EW41 au reseau local et configurer son adresse IP ; reserver si possible une adresse dans le DHCP du routeur.

Le brochage exact des connecteurs et les caracteristiques d'alimentation doivent etre verifies sur les appareils avant cablage : ils ne sont pas documentes ici comme universels.

## Configuration EW41

Configurer la passerelle en conversion **Modbus TCP vers Modbus RTU** (et non en simple port serie distant si le firmware propose plusieurs modes). Regler l'interface RS485 sur **19200, 8E1** et le service TCP sur le port **502**. Le client Home Assistant utilisera l'identifiant esclave **1**.

## Configuration Home Assistant

**Ajouter le bloc `modbus:` ci-dessous dans le fichier `configuration.yaml` de Home Assistant**, puis adapter l'adresse IP a celle de votre EW41. Si un bloc `modbus:` existe deja, ajouter l'entree `- name: airzone_flexa` a la liste existante : ne pas creer un second bloc `modbus:` au meme niveau.

Apres modification, verifier la configuration Home Assistant puis redemarrer Home Assistant pour charger la configuration Modbus.

Exemple minimal a adapter a votre reseau local :

```yaml
modbus:
  - name: airzone_flexa
    type: tcp
    host: 192.168.1.100  # A remplacer par l'adresse locale de l'EW41
    port: 502
    delay: 1
    timeout: 3
    sensors:
      - name: Airzone Mode Global
        address: 0
        slave: 1
        scan_interval: 30
        data_type: int16
```

Cet exemple effectue **une lecture uniquement**, pour commencer sans action sur la regulation. Verifier que les valeurs se mettent a jour et que les communications restent stables avant d'ajouter les autres registres et surtout les ecritures.

## Depannage

- Absence de communication : verifier adresse IP, port 502, mode de conversion, parametres 8E1, esclave 1 et paire A/B.
- Valeurs indisponibles : verifier que la revision FLEXA prend en charge les registres testes ; ne pas multiplier les lectures trop rapidement.
- Eviter d'exposer Modbus TCP a Internet : le protocole ne fournit pas, a lui seul, d'authentification adaptee.

Pour poursuivre après le test en lecture seule, suivre le [guide d'installation pas à pas](installation.fr.md) et les [exemples Modbus pour quatre zones](../homeassistant/modbus.yaml). Les lectures exploratoires ne font pas partie de la configuration opérationnelle publiée.
