# Synchronisation quotidienne de l'horloge FLEXA

Cette automatisation est une transcription anonymisée de l'automatisation Home Assistant testée sur la centrale FLEXA 1.x. Elle a corrigé avec succès le jour et une dérive d'environ une heure lors d'un essai manuel. La transition saisonnière été/hiver n'a pas encore été testée.

- Exécution : chaque jour à 04:15, heure locale de Home Assistant, après les transitions d'heure légale en France.
- Source : `sensor.az_diagnostic_reg_4`, capteur Modbus de l'adresse 4, type `int16`, esclave 1.
- Écriture : `modbus.write_register`, hub `airzone_flexa`, adresse 4.
- Écriture seulement si jour différent ou écart horaire strictement supérieur à 2 minutes.
- Les bits 14 et 15 du registre sont préservés.
- Le fuseau horaire Home Assistant doit être correctement configuré (par exemple `Europe/Paris`).

**Installation :** copier l'automatisation ci-dessous dans votre configuration d'automatisations en adaptant les identifiants du capteur et du hub. Ne pas recopier un identifiant `id` existant. Tester manuellement et vérifier l'heure sur le Tacto avant d'activer la planification.

```yaml
alias: "Airzone FLEXA - Synchronisation quotidienne de l'horloge"
description: "Synchronise le Tacto après les changements d'heure, seulement si nécessaire."
triggers:
  - trigger: time
    at: "04:15:00"
actions:
  - action: homeassistant.update_entity
    target:
      entity_id: sensor.az_diagnostic_reg_4
  - delay:
      seconds: 2
  - variables:
      raw: "{{ states('sensor.az_diagnostic_reg_4') }}"
  - condition: template
    value_template: "{{ is_number(raw) and -32768 <= (raw | int) <= 65535 }}"
  - variables:
      word: "{{ (raw | int) % 65536 }}"
      az_day: "{{ (word | int) % 8 }}"
      az_hour: "{{ ((word | int) // 8) % 32 }}"
      az_minute: "{{ ((word | int) // 256) % 64 }}"
  - condition: template
    value_template: "{{ (az_hour | int) < 24 and (az_minute | int) < 60 }}"
  - variables:
      offset_minutes: "{{ ((((now().hour * 60 + now().minute) - ((az_hour | int) * 60 + (az_minute | int)) + 720) % 1440) - 720) }}"
  - condition: template
    value_template: "{{ (az_day | int) != now().isoweekday() or (offset_minutes | int | abs) > 2 }}"
  - variables:
      new_value: "{{ ((word | int) // 16384) * 16384 + now().minute * 256 + now().hour * 8 + now().isoweekday() }}"
  - action: modbus.write_register
    data:
      hub: airzone_flexa
      slave: 1
      address: 4
      value: "{{ new_value | int }}"
  - delay:
      seconds: 2
  - action: homeassistant.update_entity
    target:
      entity_id: sensor.az_diagnostic_reg_4
mode: single
```

**Limites :** l'automatisation n'est pas exécutée si Home Assistant est arrêté à 04:15. Le contrôle de dérive utilise les minutes et le jour de semaine, sans comparaison d'une date complète. Elle dépend de l'exactitude de l'horloge de Home Assistant. La validation sur le passage effectif à l'heure d'été ou d'hiver reste à réaliser.
