# HA Utinform kamera integráció

Home Assistant alá a Utinform kameraképének berakása
![HA Utinform kamera](/pictures/HA_Utinform.jpg)

# Előkészületek
- www.utinform.hu
  ![HA Utinform kamera](/pictures/utinform1.jpg)
  1, jobb szélen bekapcsolod a kamerákat, és megkeresed a térképen ami érdekel
  2, klikk a kamerára - megjelenik a kamerák képe
  ![HA Utinform kamera](/pictures/utinform2.jpg)
  3, Chrome F12 (fejlesztői eszközök)
      - disable cache bepipál
      - network és várakozás
      - name alatt egyszer csak megjelennek a kamerák, ha megvan klikk rá és kinyílik egy ablak aminek a generál részében ott a           Requested URL - ez a kamera elérhetősége (mögötte a lid nem kell) 
Pl: https://cdnuiwebcams.utinform.hu/webcamimages/mcs033_2.jpg
 ezt bemásolod a böngészőbe és ellenőrződ, hogy le tudod-e tölteni. ha igen, másik kamera(k) linkjét hasonló módon kinyered.
 ![HA Utinform kamera](/pictures/utinform3.jpg)
# Home Assistant beállítások:

# configuration.yaml
configuration.yaml végére beilleszteni, utána teljes HA restart kell.
Lekéri a kamerák képét és betárolja a /config/www/ könyvtárba utinform_cam1, utinform_cam2 néven
a megszerzett kamera linkeket másold bele a ? elé

```
shell_command:
  utinform_cam1: >
    bash -c 'curl -L -s -A "Mozilla/5.0" -o /config/www/utinform_cam1.jpg
    "https://cdnuiwebcams.utinform.hu/webcamimages/mcs033_1.jpg?lid=$(date +%s%3N)"'
    
  utinform_cam2: >
    bash -c 'curl -L -s -A "Mozilla/5.0" -o /config/www/utinform_cam2.jpg
    "https://cdnuiwebcams.utinform.hu/webcamimages/mcs033_2.jpg?lid=$(date +%s%3N)"'
```

# automatizmusok:
1, 5 percenként frissíti a kamera(k) képeit

```
automation:
  - alias: "Útinform kamerák frissítése 5 percenként"
    mode: single
    trigger:
      - platform: time_pattern
        minutes: "/5"
    action:
      - service: shell_command.utinform_cam1
      - service: shell_command.utinform_cam2
```
2, HA restart után azonnal frissít

```
automation:
  - alias: "HA init – restart utáni feladatok"
    mode: single
    trigger:
      - platform: homeassistant
        event: start
    action:
      # várunk, amíg minden integration feláll
      - delay: "00:00:20"

      # Utinform kamerák frissítése
      - service: shell_command.utinform_cam1
      - service: shell_command.utinform_cam2

```

