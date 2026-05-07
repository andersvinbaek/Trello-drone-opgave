# Tellodrone-opgave

## Links:
https://miro.com/app/board/uXjVHb_ljOQ=/

## Idégenerering
<img width="2582" height="774" alt="billede" src="https://github.com/user-attachments/assets/9e060b80-0335-4b95-86b8-16429f27e039" />

## Problemformulering
Problemet er, at når du er sengelæggende, kan man have svært ved at komme ud af sengen (det kræver meget energi). Så hvis du skal have en dejlig lille forfriskning i form af en kop te, kan det være umuligt, ellers skal du vente længe på en sygeplejerske eller SOSU-arbejder.

Løsningen er, at sætte en krog på Tello-dronen, der så kan samle tebrevet op, og føre det over i en kop med varmt vand.

## Grund ide af dronens funktion
<img width="1778" height="310" alt="image" src="https://github.com/user-attachments/assets/b4f50268-66fb-4aa5-a69d-9bf31204d162" />
<br>

## Tellodrone-specifikationer
<img width="1468" height="1246" alt="image" src="https://github.com/user-attachments/assets/afc79d20-1a79-4e28-a407-8aad5dd1a679" />
<br>

## Blockdiagram over montering dele
<img width="4218" height="948" alt="image" src="https://github.com/user-attachments/assets/f8b1417e-9511-47fc-9560-d525e21b5809" />
<br>

<details>
<summary><h1>Logbog</h1></summary>
  
## Logbog 28/04/26
I dag er vi startet op på det nye emne i informatik: Droner. Vi er blevet introduceret til droner generelt, lidt kort om forløbet, og har fået udleveret et ny projekt, hvor vi skal lave noget med droner.
Vi oprettede på en github og et miroboard og gik i gang med at teste dronen. Derefter lavede vi lidt idégenerering og lavede vores problemformulering.

## Logbog 30/04/26
I dag har vi fløjet meget mere med dronen. Vi har også lavet noget kode med python (med meget hjælp fra Claude), så vi kunne få dronen til at flyve vha. tastatur. Vi har lavet en midlertidig løsning for at teste, om dronen kunne flyve med både tepose og krog, hvilket den kan.

  <details>
  <summary><h2>Kode - 30/04/26</h2></summary>

```
"""
Tello Drone - Smooth keyboard styring + kamerafeed
Krav: pip install djitellopy opencv-python keyboard

Forbind til Tello Wi-Fi inden du starter scriptet.
Kør som administrator hvis tastaturet ikke virker!

Taster:
  T       - Tag af (takeoff)
  L       - Land
  Q       - Afslut
  W/S     - Frem / Tilbage
  A/D     - Venstre / Højre
  E/C     - Op / Ned
  Z/X     - Roter venstre / Højre
"""

import cv2
import keyboard
from djitellopy import Tello
import threading
import time


// Indstillinger
 
SPEED = 50   # cm/s (0-100)

 
// Initialisering
 
tello = Tello()
tello.connect()
print(f"Batteri: {tello.get_battery()}%")

tello.streamon()
frame_reader = tello.get_frame_read()

in_flight = False
running   = True


// Styretråd — sender RC-kommandoer 20x/sek
 
def control_loop():
    global in_flight, running
    while running:
        if in_flight:
            lr = fb = ud = yaw = 0

            if keyboard.is_pressed('a'):   lr  = -SPEED
            if keyboard.is_pressed('d'):   lr  =  SPEED
            if keyboard.is_pressed('w'):   fb  =  SPEED
            if keyboard.is_pressed('s'):   fb  = -SPEED
            if keyboard.is_pressed('e'):   ud  =  SPEED
            if keyboard.is_pressed('c'):   ud  = -SPEED
            if keyboard.is_pressed('z'):   yaw = -SPEED
            if keyboard.is_pressed('x'):   yaw =  SPEED

            tello.send_rc_control(lr, fb, ud, yaw)

        time.sleep(0.05)  # 20 gange i sekundet

ctrl_thread = threading.Thread(target=control_loop, daemon=True)
ctrl_thread.start()


// Hoved-loop (video + enkelt-taster)
 
print("T=takeoff  L=land  Q=afslut  WASD=retning  E/C=op/ned  Z/X=roter")

while running:
    frame = frame_reader.frame
    if frame is None:
        continue

    frame = cv2.resize(frame, (960, 720))

    battery = tello.get_battery()
    status  = "I LUFTEN" if in_flight else "PAA JORDEN"
    cv2.putText(frame, f"Batteri: {battery}%", (20, 40),
                cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
    cv2.putText(frame, status, (20, 80),
                cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 0), 2)
    cv2.putText(frame, "T=takeoff L=land Q=afslut", (20, 710),
                cv2.FONT_HERSHEY_SIMPLEX, 0.6, (200, 200, 200), 1)

    cv2.imshow("Tello Drone", frame)
    cv2.waitKey(1)

    # Enkelt-taster (takeoff/land/quit)
    if keyboard.is_pressed('q'):
        print("Afslutter...")
        running = False

    elif keyboard.is_pressed('t') and not in_flight:
        tello.takeoff()
        in_flight = True
        print("Takeoff!")
        time.sleep(0.5)  # undgå dobbelt-registrering

    elif keyboard.is_pressed('l') and in_flight:
        tello.land()
        in_flight = False
        print("Landing!")
        time.sleep(0.5)

// Oprydning
 
if in_flight:
    tello.land()
tello.streamoff()
tello.end()
cv2.destroyAllWindows()
print("Afsluttet.")
```
  </details>

## Logbog 04/05/26
I dag har vi expermenteret med en kode så dronen ville kunne lande ved at se en specifik farve. Vi har også styrtet dronen et par gange mens vi forsøgte at flyve med krogen, som vi havde forstærket.

  <details>
  <summary><h2>Kode - 04/05/26</h2></summary>

```
from djitellopy import Tello
import cv2
import numpy as np
import time

tello = Tello()

print("Forbinder...")
tello.connect()

print("Batteri:", tello.get_battery())

tello.streamon()
time.sleep(2)

tello.takeoff()
time.sleep(2)

print("Kører... (lander når rød farve opdages)")

red_detected_counter = 0
RED_THRESHOLD_FRAMES = 10  # hvor mange frames i træk der skal være rød

while True:
    frame = tello.get_frame_read().frame
    if frame is None:
        continue

    # Konverter til HSV
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

    # Rød farve (to ranges i HSV)
    lower_red1 = np.array([0, 120, 70])
    upper_red1 = np.array([10, 255, 255])

    lower_red2 = np.array([170, 120, 70])
    upper_red2 = np.array([180, 255, 255])

    mask1 = cv2.inRange(hsv, lower_red1, upper_red1)
    mask2 = cv2.inRange(hsv, lower_red2, upper_red2)

    mask = mask1 + mask2

    # Fjern støj
    mask = cv2.GaussianBlur(mask, (5, 5), 0)

    # Find hvor meget rød der er
    red_pixels = cv2.countNonZero(mask)
    total_pixels = frame.shape[0] * frame.shape[1]

    red_ratio = red_pixels / total_pixels

    # Visualisering
    cv2.imshow("Tello Kamera", frame)
    cv2.imshow("Rød maske", mask)

    # Hvis der er nok rød i billedet
    if red_ratio > 0.08:  # justér denne værdi hvis nødvendigt
        red_detected_counter += 1
        print("Rød set:", red_detected_counter)
    else:
        red_detected_counter = 0

    # Hvis rød ses stabilt → land
    if red_detected_counter >= RED_THRESHOLD_FRAMES:
        print("Rød farve bekræftet - lander!")
        tello.land()
        break

    key = cv2.waitKey(1) & 0xFF

    if key == 27:  # ESC = nødlanding
        tello.land()
        break
        
cv2.destroyAllWindows()
tello.streamoff()
```
  </details>
</details>

<details>
<summary><h1>Bilag</h1></summary>

## Dronen med udstyr
<img width="3024" height="4032" alt="IMG_5469" src="https://github.com/user-attachments/assets/4ca01080-9d02-4b84-858b-e3d60ad6f110" />
<br>
<img width="3024" height="4032" alt="IMG_5470" src="https://github.com/user-attachments/assets/1eed3304-8e7b-4e10-8c10-5e6e75688a49" />
<br>
<img width="3024" height="4032" alt="IMG_5471" src="https://github.com/user-attachments/assets/6da7ed00-ac11-44a5-917e-c1e2ef1fc9c6" />
<br>

</details>
