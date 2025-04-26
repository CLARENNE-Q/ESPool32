# 🌡️ ESPool - ESP32 Pool Temperature Monitoring System

Ce projet DIY permet de mesurer **3 températures clés de votre piscine** à l’aide d’un ESP32, de sondes thermistance 10k AR-PRO et d’une intégration complète dans **Home Assistant** via ESPHome.

---

## 🔧 Objectifs

- Suivi en temps réel de :
  - Température de l’eau de la piscine
  - Température après les panneaux solaires
  - Température après la thermopompe
- Calcul automatique des **deltas de température**
- Affichage dans Home Assistant (graphique, jauges, etc.)
- Détection de performance solaire / thermopompe

---

## 🧰 Matériel requis

| Composant | Quantité | Détail |
|----------|----------|--------|
| ESP32 (DEVKIT v1) | 1 | Avec 3 GPIO analogiques (32, 34, 35) |
| Sondes thermistance AR-PRO 10k (NTC) | 3 | Étanches, connecteurs 3 fils |
| Résistances 10kΩ | 3 | Précision 1% de préférence |
| Boîtier étanche | 1 | Pour protéger l’ESP32 |
| Câbles Dupont ou domotique | Plusieurs | Connexion stable des sondes |

---

## 🧪 Schéma de câblage

            3.3V (ESP32)
               │
      ┌────────┼────────┐
      │        │        │
   [AR-PRO] [AR-PRO] [AR-PRO]
      │        │        │
     ▼▼▼      ▼▼▼      ▼▼▼
    GPIO34   GPIO35   GPIO32  <--- Prise de tension (ADC)
      │        │        │
    [R10kΩ]  [R10kΩ]  [R10kΩ]
      │        │        │
     GND      GND      GND



---

## 📦 Fichier `espool.yaml` ESPHome (extrait)

```yaml
sensor:
  # SONDE 1 - Piscine (GPIO34)
  - platform: resistance
    id: resistance_pool_1
    sensor: adc_pool_1
    resistor: 10kOhm
    configuration: UPSTREAM

  - platform: ntc
    sensor: resistance_pool_1
    name: "Température Piscine"
    calibration:
      b_constant: 3950
      reference_temperature: 25°C
      reference_resistance: 10kOhm

  - platform: adc
    pin: 34
    id: adc_pool_1
    attenuation: 11db
    update_interval: 10s
    filters:
      - median:
          window_size: 7
          send_every: 2
          send_first_at: 1
```

```
---template:
  - sensor:
      - name: "Delta Solaire"
        unit_of_measurement: "°C"
        device_class: temperature
        state_class: measurement
        state: >
          {{ (states('sensor.espool_temp_rature_solaire') | float - states('sensor.espool_temp_rature_piscine') | float) | round(2) }}

      - name: "Delta Thermopompe"
        unit_of_measurement: "°C"
        device_class: temperature
        state_class: measurement
        state: >
          {{ (states('sensor.espool_temp_rature_thermopompe') | float - states('sensor.espool_temp_rature_solaire') | float) | round(2) }}

```
