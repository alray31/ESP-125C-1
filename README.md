# ESP-125C-1

[English](#english) | [Français](#français)

---

## English

Replace your ACM-125C-1 RF433 remote with an ESP32 + Qiachip WL102-341 transmitter module to control pool lights (skimmer or return jet) using ESPHome. This project transmits the RF signals captured from an original ACM-125C-1 remote, allowing integration with Home Assistant.

Original ACM-125C-1 remote:

<img width="230" height="406" alt="image" src="https://github.com/user-attachments/assets/b28b058f-0d4f-45b9-8b9d-f4bb4036e94f" />

Used to control skimmer light:

<img width="270" height="182" alt="image" src="https://github.com/user-attachments/assets/b23a936e-f089-4bd3-bd39-f6049aea3096" />

And/or return jet light:

<img width="270" height="271" alt="image" src="https://github.com/user-attachments/assets/bb05db57-c002-4275-9b05-a271657ca118" />

---

### Part List

Here's what you'll need to replicate this project:

| Quantity | Component | Notes |
|----------|-----------|-------|
| 1        | ESP32 Dev Board | Any ESP32 board compatible with ESPHome |
| 1        | Qiachip WL102-341 RF433 Transmitter Module | To transmit RF 433.92 MHz codes |
| 1        | Jumper Wires | For connections between ESP32 and transmitter |
| 1        | USB Cable | To power and flash the ESP32 |
| Optional | 3D-printed enclosure | To house the electronics safely |

ESP32 Dev board:  
<img width="273" height="272" alt="image" src="https://github.com/user-attachments/assets/2b5acb89-069d-4b25-9b7b-e92e0d80850e" />

RF433 Transmitter:  
<img width="181" height="172" alt="image" src="https://github.com/user-attachments/assets/65407214-096f-4e60-aab9-93035d3a9f96" />

---

### Wiring Diagram

<img width="584" height="547" alt="image" src="https://github.com/user-attachments/assets/619c494c-9e98-438e-abf1-fa1421785298" />

**Note:** EN contact from RF transmitter is not used.

---

### Proof of Concept

This project has been tested and successfully transmits the captured RF signals from an ACM-125C-1 remote. The ESP32, running ESPHome, can toggle the pool lights (skimmer or return jet) from Home Assistant.

Steps verified in proof of concept:

1. Captured the RF codes from the original ACM-125C-1 remote using an RTL-SDR dongle.
2. Analyzed captured codes using Universal Radio Hacker software.
3. Programmed the ESP32 using ESPHome to transmit these codes.
4. Verified successful control of pool lights through Home Assistant.
5. Tested repeated transmissions for reliability.

---

### Code Analysis

The ACM-125C-1 remote transmits RF signals as a pulse train. The structure is as follows:

1. **Leading Pulse / Preamble** – An initial pulse signals the start of the transmission (preamble). The preamble (pulse length + pause length) is consistent for all commands except the On/Off commands:
   - **ON:** pulse of 23100µs, pause of 7500µs
   - **OFF:** pulse of 11610µs, pause of 30440µs
   - **All other commands:** pulse of 263µs, pause of 7520µs

2. **25-bit Data Transmission** – The actual command is encoded as a 25-bit sequence of high and low pulses.

3. **Repeated Transmission** – The 25-bit signal is repeated 10 times for all commands, with a pause of 7250µs between each repetition, except the pair command, which is repeated 110 times.

Typical pulse train:  
<img width="874" height="149" alt="image" src="https://github.com/user-attachments/assets/ace97d7a-4200-41c9-8f98-026032245a89" />
<img width="1309" height="330" alt="image" src="https://github.com/user-attachments/assets/e8e0f3f1-22c8-46a7-9dca-55a90c6831e2" />

Typical 25-bit sequence:  
<img width="958" height="155" alt="image" src="https://github.com/user-attachments/assets/4625414a-4c45-4003-9139-8389a9c9b14c" />

**Pulse analysis and binary translation:**

1. **Base pulse length** – The base pulse length (T) is approximately 260µs.

2. **Protocol** – The protocol uses a 4T sampling interval (1040µs) to determine if the bit is low (0) or high (1):
   - 1T high (a high pulse of 260µs) followed by 3T low (a pause of 780µs) = **0**
   - 3T high (a high pulse of 780µs) followed by 1T low (a pause of 260µs) = **1**

<img width="599" height="408" alt="image" src="https://github.com/user-attachments/assets/f2227939-a3ed-4188-a02f-d3ab57bb40ea" />

This pattern of 3T high : 1T low / 1T high : 3T low matches RC-Switch protocol #1 of the Arduino library.

Translating 25-bit analog pulse train to binary:

<img width="954" height="181" alt="image" src="https://github.com/user-attachments/assets/40d0c449-b087-47fb-8afe-ee5338555fd2" />

**Decoding the 25-bit sequence:**

1. **Remote unique address** – The first 16 bits encode the remote's unique address and remain the same for all transmissions.

2. **Data block** – Bits 17 to 24 encode the command sent by the remote:
   - Bit 17 is high for effect speed/light intensity commands and low for all other commands (ON/OFF/PAIR/EFFECT/WHITE/COLOR).
   - Bit 18 is high when sending a color wheel command, and remains low for all other commands.

3. **End bit/Stop bit** – Bit 25 is always high to confirm the end of the data block.

<img width="1087" height="465" alt="image" src="https://github.com/user-attachments/assets/4e80ecd9-931f-47e7-bf7d-e4755f59b1a4" />

**Binary Codes Table:**

| ADDRESS          | DATA       | END | DESCRIPTION                               |
|------------------|------------|-----|-------------------------------------------|
| 1111110100100110 | 0 0000111  | 1   | ON                                        |
| 1111110100100110 | 0 0111000  | 1   | OFF                                       |
| 1111110100100110 | 0 0110001  | 1   | PAIR                                      |
| 1111110100100110 | 0 0001110  | 1   | GRADUAL EFFECT                            |
| 1111110100100110 | 0 0010101  | 1   | WAVE EFFECT                               |
| 1111110100100110 | 0 0011100  | 1   | JUMPING EFFECT                            |
| 1111110100100110 | 0 0100011  | 1   | FADING EFFECT                             |
| 1111110100100110 | 0 0101010  | 1   | WAVE + JUMPING EFFECT                     |
| 1111110100100110 | 0 0001001  | 1   | STEADY WHITE MODE                         |
| 1111110100100110 | 0 0110110  | 1   | COLOR WHEEL MODE                          |
| 1111110100100110 | 0 1000000  | 1   | COLOR WHEEL 000 DEGREE                    |
| 1111110100100110 | 0 1******  | 1   | 62 OTHER COLOR WHEEL CODES IN BETWEEN     |
| 1111110100100110 | 0 1111111  | 1   | COLOR WHEEL 364 DEGREE                    |
| 1111110100100110 | 1 1000111  | 1   | EFFECT SPEED / LIGHT INTENSITY 1          |
| 1111110100100110 | 1 1001111  | 1   | EFFECT SPEED / LIGHT INTENSITY 2          |
| 1111110100100110 | 1 1010111  | 1   | EFFECT SPEED / LIGHT INTENSITY 3          |
| 1111110100100110 | 1 1011111  | 1   | EFFECT SPEED / LIGHT INTENSITY 4          |
| 1111110100100110 | 1 1100111  | 1   | EFFECT SPEED / LIGHT INTENSITY 5          |
| 1111110100100110 | 1 1101111  | 1   | EFFECT SPEED / LIGHT INTENSITY 6          |
| 1111110100100110 | 1 1110111  | 1   | EFFECT SPEED / LIGHT INTENSITY 7          |
| 1111110100100110 | 1 1111111  | 1   | EFFECT SPEED / LIGHT INTENSITY 8          |

---

### Installation

1. Connect the Qiachip WL102-341 transmitter to the ESP32 following the wiring diagram.
2. Flash the ESP32 with the ESPHome configuration from this repository.
3. Add the ESP32 as a device in Home Assistant.
4. Control your pool lights using Home Assistant or automations.

---

### How to Use

The pool light acts as a receiver only. Therefore, no feedback is sent from the light to Home Assistant after a selection is made.

1. **Modes Dropdown – Replaces the yellow M and C buttons of the remote**
   - Used to change the operation mode of the light(s).
   - Seven modes can be selected (five dynamic modes, steady white, and steady color).
   - Use the toggle buttons or switches to turn the skimmer or return jet lights on/off.

2. **Light Intensity or Effect Speed Dropdown – Replaces the yellow + and - buttons of the remote**
   - In dynamic modes, this controls the speed of the scene mode (1 = slowest, 8 = fastest).
   - Each dynamic mode remembers its last speed. For example, if "Jumping" mode is set to speed 8 and you switch to "Fading" mode at speed 5, returning to "Jumping" mode will still be at speed 8.
   - In steady white or steady color mode, this controls light intensity (1 = dimmest, 8 = brightest).

3. **On/Off Switch – Replaces the red power button of the remote**
   - Turns the light(s) on and off.
   - Works in optimistic mode: the switch status may not reflect the actual light state since the light does not send feedback. Like the original remote, you may need to press twice to ensure the desired command is sent.

4. **Pair Button – Replaces the simultaneous press of M and C buttons on the original remote**
   - Used to pair the ESP to your light. If your lights are already paired to an ACM-125C-1 remote, pairing may not be required, as the light's RF receiver already recognizes these RF codes.
   - Pressing once momentarily simulates a long press of the M + C buttons to transmit the pairing code.
   - For these pool lights, "pairing" simply tells the receiver: "Listen for these RF codes." Multiple remotes of the same type can operate the same light. However, pairing too many different remote types may cause the first type to be ignored. In this project, the ESP acts as an ACM-125C-1 remote, so your light will recognize it automatically.

---

## Français

Remplacez votre télécommande RF433 ACM-125C-1 par un module ESP32 + émetteur Qiachip WL102-341 pour contrôler les lumières de piscine (skimmer ou buse de refoulement) à l'aide d'ESPHome. Ce projet transmet les signaux RF capturés d'une télécommande ACM-125C-1 d'origine, permettant l'intégration avec Home Assistant.

Télécommande ACM-125C-1 d'origine :

<img width="230" height="406" alt="image" src="https://github.com/user-attachments/assets/b28b058f-0d4f-45b9-8b9d-f4bb4036e94f" />

Utilisée pour contrôler la lumière du skimmer :

<img width="270" height="182" alt="image" src="https://github.com/user-attachments/assets/b23a936e-f089-4bd3-bd39-f6049aea3096" />

Et/ou la lumière de la buse de refoulement :

<img width="270" height="271" alt="image" src="https://github.com/user-attachments/assets/bb05db57-c002-4275-9b05-a271657ca118" />

---

### Liste des composants

Voici ce dont vous aurez besoin pour reproduire ce projet :

| Quantité | Composant | Notes |
|----------|-----------|-------|
| 1        | Carte de développement ESP32 | N'importe quelle carte ESP32 compatible avec ESPHome |
| 1        | Module émetteur RF433 Qiachip WL102-341 | Pour transmettre les codes RF 433.92 MHz |
| 1        | Fils de connexion | Pour les connexions entre l'ESP32 et l'émetteur |
| 1        | Câble USB | Pour alimenter et flasher l'ESP32 |
| Optionnel | Boîtier imprimé en 3D | Pour loger l'électronique en toute sécurité |

Carte de développement ESP32 :  
<img width="273" height="272" alt="image" src="https://github.com/user-attachments/assets/2b5acb89-069d-4b25-9b7b-e92e0d80850e" />

Émetteur RF433 :  
<img width="181" height="172" alt="image" src="https://github.com/user-attachments/assets/65407214-096f-4e60-aab9-93035d3a9f96" />

---

### Schéma de câblage

<img width="584" height="547" alt="image" src="https://github.com/user-attachments/assets/619c494c-9e98-438e-abf1-fa1421785298" />

**Note :** Le contact EN de l'émetteur RF n'est pas utilisé.

---

### Preuve de concept

Ce projet a été testé et transmet avec succès les signaux RF capturés d'une télécommande ACM-125C-1. L'ESP32, exécutant ESPHome, peut basculer les lumières de piscine (skimmer ou buse de refoulement) depuis Home Assistant.

Étapes vérifiées dans la preuve de concept :

1. Capture des codes RF de la télécommande ACM-125C-1 d'origine à l'aide d'une clé RTL-SDR.
2. Analyse des codes capturés à l'aide du logiciel Universal Radio Hacker.
3. Programmation de l'ESP32 à l'aide d'ESPHome pour transmettre ces codes.
4. Vérification du contrôle réussi des lumières de piscine via Home Assistant.
5. Test de transmissions répétées pour la fiabilité.

---

### Analyse du code

La télécommande ACM-125C-1 transmet des signaux RF sous forme de train d'impulsions. La structure est la suivante :

1. **Impulsion principale / Préambule** – Une impulsion initiale signale le début de la transmission (préambule). Le préambule (durée de l'impulsion + durée de la pause) est cohérent pour toutes les commandes sauf les commandes On/Off :
   - **ON :** impulsion de 23100µs, pause de 7500µs
   - **OFF :** impulsion de 11610µs, pause de 30440µs
   - **Toutes les autres commandes :** impulsion de 263µs, pause de 7520µs

2. **Transmission de données de 25 bits** – La commande réelle est codée sous forme de séquence de 25 bits d'impulsions hautes et basses.

3. **Transmission répétée** – Le signal de 25 bits est répété 10 fois pour toutes les commandes, avec une pause de 7250µs entre chaque répétition, à l'exception de la commande d'appairage, qui est répétée 110 fois.

Train d'impulsions typique :  
<img width="874" height="149" alt="image" src="https://github.com/user-attachments/assets/ace97d7a-4200-41c9-8f98-026032245a89" />
<img width="1309" height="330" alt="image" src="https://github.com/user-attachments/assets/e8e0f3f1-22c8-46a7-9dca-55a90c6831e2" />

Séquence typique de 25 bits :  
<img width="958" height="155" alt="image" src="https://github.com/user-attachments/assets/4625414a-4c45-4003-9139-8389a9c9b14c" />

**Analyse des impulsions et traduction binaire :**

1. **Durée d'impulsion de base** – La durée d'impulsion de base (T) est d'environ 260µs.

2. **Protocole** – Le protocole utilise un intervalle d'échantillonnage de 4T (1040µs) pour déterminer si le bit est bas (0) ou haut (1) :
   - 1T haut (une impulsion haute de 260µs) suivi de 3T bas (une pause de 780µs) = **0**
   - 3T haut (une impulsion haute de 780µs) suivi de 1T bas (une pause de 260µs) = **1**

<img width="599" height="408" alt="image" src="https://github.com/user-attachments/assets/f2227939-a3ed-4188-a02f-d3ab57bb40ea" />

Ce schéma de 3T haut : 1T bas / 1T haut : 3T bas correspond au protocole RC-Switch #1 de la bibliothèque Arduino.

Traduction du train d'impulsions analogique de 25 bits en binaire :

<img width="954" height="181" alt="image" src="https://github.com/user-attachments/assets/40d0c449-b087-47fb-8afe-ee5338555fd2" />

**Décodage de la séquence de 25 bits :**

1. **Adresse unique de la télécommande** – Les 16 premiers bits codent l'adresse unique de la télécommande et restent identiques pour toutes les transmissions.

2. **Bloc de données** – Les bits 17 à 24 codent la commande envoyée par la télécommande :
   - Le bit 17 est haut pour les commandes de vitesse d'effet/intensité lumineuse et bas pour toutes les autres commandes (ON/OFF/PAIR/EFFECT/WHITE/COLOR).
   - Le bit 18 est haut lors de l'envoi d'une commande de roue chromatique, et reste bas pour toutes les autres commandes.

3. **Bit de fin/Bit d'arrêt** – Le bit 25 est toujours haut pour confirmer la fin du bloc de données.

<img width="1087" height="465" alt="image" src="https://github.com/user-attachments/assets/4e80ecd9-931f-47e7-bf7d-e4755f59b1a4" />

**Tableau des codes binaires :**

| ADRESSE          | DONNÉES    | FIN | DESCRIPTION                               |
|------------------|------------|-----|-------------------------------------------|
| 1111110100100110 | 0 0000111  | 1   | ON                                        |
| 1111110100100110 | 0 0111000  | 1   | OFF                                       |
| 1111110100100110 | 0 0110001  | 1   | APPAIRAGE                                 |
| 1111110100100110 | 0 0001110  | 1   | EFFET GRADUEL                             |
| 1111110100100110 | 0 0010101  | 1   | EFFET VAGUE                               |
| 1111110100100110 | 0 0011100  | 1   | EFFET SAUTILLANT                          |
| 1111110100100110 | 0 0100011  | 1   | EFFET FONDU                               |
| 1111110100100110 | 0 0101010  | 1   | EFFET VAGUE + SAUTILLANT                  |
| 1111110100100110 | 0 0001001  | 1   | MODE BLANC FIXE                           |
| 1111110100100110 | 0 0110110  | 1   | MODE ROUE CHROMATIQUE                     |
| 1111110100100110 | 0 1000000  | 1   | ROUE CHROMATIQUE 000 DEGRÉS               |
| 1111110100100110 | 0 1******  | 1   | 62 AUTRES CODES DE ROUE CHROMATIQUE       |
| 1111110100100110 | 0 1111111  | 1   | ROUE CHROMATIQUE 364 DEGRÉS               |
| 1111110100100110 | 1 1000111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 1   |
| 1111110100100110 | 1 1001111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 2   |
| 1111110100100110 | 1 1010111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 3   |
| 1111110100100110 | 1 1011111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 4   |
| 1111110100100110 | 1 1100111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 5   |
| 1111110100100110 | 1 1101111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 6   |
| 1111110100100110 | 1 1110111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 7   |
| 1111110100100110 | 1 1111111  | 1   | VITESSE D'EFFET / INTENSITÉ LUMINEUSE 8   |

---

### Installation

1. Connectez l'émetteur Qiachip WL102-341 à l'ESP32 en suivant le schéma de câblage.
2. Flashez l'ESP32 avec la configuration ESPHome de ce dépôt.
3. Ajoutez l'ESP32 comme appareil dans Home Assistant.
4. Contrôlez vos lumières de piscine à l'aide de Home Assistant ou d'automatisations.

---

### Comment utiliser

La lumière de piscine agit uniquement comme récepteur. Par conséquent, aucun retour d'information n'est envoyé de la lumière vers Home Assistant après qu'une sélection a été effectuée.

1. **Menu déroulant des modes – Remplace les boutons jaunes M et C de la télécommande**
   - Utilisé pour changer le mode de fonctionnement de la/des lumière(s).
   - Sept modes peuvent être sélectionnés (cinq modes dynamiques, blanc fixe et couleur fixe).
   - Utilisez les boutons à bascule ou les interrupteurs pour allumer/éteindre les lumières du skimmer ou de la buse de refoulement.

2. **Menu déroulant Intensité lumineuse ou Vitesse d'effet – Remplace les boutons jaunes + et - de la télécommande**
   - En modes dynamiques, cela contrôle la vitesse du mode de scène (1 = le plus lent, 8 = le plus rapide).
   - Chaque mode dynamique se souvient de sa dernière vitesse. Par exemple, si le mode "Sautillant" est réglé sur la vitesse 8 et que vous passez au mode "Fondu" à la vitesse 5, le retour au mode "Sautillant" sera toujours à la vitesse 8.
   - En mode blanc fixe ou couleur fixe, cela contrôle l'intensité lumineuse (1 = le plus faible, 8 = le plus lumineux).

3. **Interrupteur On/Off – Remplace le bouton d'alimentation rouge de la télécommande**
   - Allume et éteint la/les lumière(s).
   - Fonctionne en mode optimiste : l'état de l'interrupteur peut ne pas refléter l'état réel de la lumière puisque la lumière n'envoie pas de retour d'information. Comme avec la télécommande d'origine, vous devrez peut-être appuyer deux fois pour vous assurer que la commande souhaitée est envoyée.

4. **Bouton d'appairage – Remplace l'appui simultané des boutons M et C sur la télécommande d'origine**
   - Utilisé pour appairer l'ESP à votre lumière. Si vos lumières sont déjà appairées à une télécommande ACM-125C-1, l'appairage peut ne pas être nécessaire, car le récepteur RF de la lumière reconnaît déjà ces codes RF.
   - Appuyer une fois simule momentanément un appui long sur les boutons M + C pour transmettre le code d'appairage.
   - Pour ces lumières de piscine, « l'appairage » indique simplement au récepteur : « Écoute ces codes RF. » Plusieurs télécommandes du même type peuvent contrôler la même lumière. Cependant, l'appairage de trop de types de télécommandes différents peut faire en sorte que le premier type soit ignoré. Dans ce projet, l'ESP agit comme une télécommande ACM-125C-1, donc votre lumière le reconnaîtra automatiquement.

---

## License
