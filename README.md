# Tellodrone-opgave

## Links:
https://miro.com/app/board/uXjVHb_ljOQ=/

---

## Idégenerering
<img width="2582" height="774" alt="billede" src="https://github.com/user-attachments/assets/9e060b80-0335-4b95-86b8-16429f27e039" />

---

## Problemformulering
Problemet er, at når du er sengelæggende, kan man have svært ved at komme ud af sengen (det kræver meget energi). Så hvis du skal have en dejlig lille forfriskning i form af en kop te, kan det være umuligt, ellers skal du vente længe på en sygeplejerske eller SOSU-arbejder.

Løsningen er, at sætte en krog på Tello-dronen, der så kan samle tebrevet op, og føre det over i en kop med varmt vand.

---

## Grund ide af dronens funktion
<img width="1778" height="310" alt="image" src="https://github.com/user-attachments/assets/b4f50268-66fb-4aa5-a69d-9bf31204d162" />
<br>

---

## Tellodrone-specifikationer
<img width="1468" height="1246" alt="image" src="https://github.com/user-attachments/assets/afc79d20-1a79-4e28-a407-8aad5dd1a679" />
<br>

---

## Blockdiagram over montering dele
<img width="4218" height="948" alt="image" src="https://github.com/user-attachments/assets/f8b1417e-9511-47fc-9560-d525e21b5809" />
<br>

---

## Trelags-arkitektur
<img width="356" height="665" alt="image" src="https://github.com/user-attachments/assets/58a824bf-5c41-454b-a4f5-caf01fc96e46" />
<br>

---

<details>
  <summary><h1>10 Usability Heuristics</h1></summary>

## 1. Visibility of System Status
The design should always keep users informed about what is going on through clear and timely feedback.

---

## 2. Match Between the System and the Real World
The system should use language, concepts, and conventions familiar to the user.

---

## 3. User Control and Freedom
Users should be able to easily undo actions, exit unwanted states, and stay in control of the system.

---

## 4. Consistency and Standards
The interface should follow consistent patterns and established conventions.

---

## 5. Error Prevention
The design should prevent problems from occurring before users make mistakes.

---

## 6. Recognition Rather than Recall
The interface should minimize memory load by making options and information visible.

---

## 7. Flexibility and Efficiency of Use
The system should support both inexperienced and experienced users through efficient and flexible interaction.

---

## 8. Aesthetic and Minimalist Design
Interfaces should contain only relevant information and avoid unnecessary complexity.

---

## 9. Help Users Recognize, Diagnose, and Recover from Errors
Error messages should clearly explain problems and help users recover from them.

---

## 10. Help and Documentation
Help and documentation should be easy to find, concise, and focused on the user’s tasks.
</details>

<details>
<summary><h1>Logbog</h1></summary>
  
## Logbog 28/04/26
I dag er vi startet op på det nye emne i informatik: Droner. Vi er blevet introduceret til droner generelt, lidt kort om forløbet, og har fået udleveret et ny projekt, hvor vi skal lave noget med droner.
Vi oprettede på en github og et miroboard og gik i gang med at teste dronen. Derefter lavede vi lidt idégenerering og lavede vores problemformulering.

---

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

---

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

---

## Logbog 07/05/26
I dag arbejdede vi med at få dronen til at virke effektiv med python, og få kameratet til at vise et klart billede <br>
Hovedmålet var at lave koden så dronen fungere lidt ligsom den fra Palantir, hvor den kan jagte efter personen hoved og derefter styrte ind i dem. 
  <details>
  <summary><h2>Kode - 07/05/26</h2></summary>

```
from djitellopy import Tello
import cv2
import mediapipe as mp
import numpy as np
import time

# --- INITIALISERING ---
tello = Tello()

def start_tello():
    try:
        tello.connect()
        print(f"Batteri: {tello.get_battery()}%")
        tello.streamon()
        # Giv den tid til at åbne porten
        time.sleep(2)
        return tello.get_frame_read()
    except Exception as e:
        print(f"Forbindelsesfejl: {e}")
        return None

frame_reader = start_tello()

# MediaPipe Pose - Hurtig konfiguration
mp_pose = mp.solutions.pose
pose = mp_pose.Pose(
    static_image_mode=False,
    model_complexity=0, # 0 er hurtigst, forhindrer frys
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5
)

# --- INDSTILLINGER ---
MAX_SPEED = 100       # Fuld gas fremad
TARGET_SIZE = 35000   # Stop-afstand (ca. 30 cm)
flying = False

print("KLAR! Tryk 'T' for Takeoff (Jagt starter med det samme)")

while True:
    # 1. HENT NYESTE FRAME (Undgå kø/buffer frys)
    frame = frame_reader.frame
    if frame is None:
        continue

    # 2. OPTIMERING: Kør AI på en meget lille kopi (mindsker CPU belastning)
    h, w, _ = frame.shape
    img_display = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)
    small_frame = cv2.resize(frame, (320, 240))
    results = pose.process(small_frame)

    lr, fb, ud, yv = 0, 0, 0, 0

    # 3. JAGTLOGIK
    if results.pose_landmarks:
        lan = results.pose_landmarks.landmark
        
        # Find center mellem skuldre (virker forfra og bagfra)
        l_sh = lan[mp_pose.PoseLandmark.LEFT_SHOULDER]
        r_sh = lan[mp_pose.PoseLandmark.RIGHT_SHOULDER]
        
        # Beregn center (vandret)
        cx = int(((l_sh.x + r_sh.x) / 2) * w)
        
        # Drej mod mål (Yaw)
        yv = int((cx - (w // 2)) * 0.8)
        yv = np.clip(yv, -90, 90)

        if flying:
            # Afstandsberegning
            width = abs(l_sh.x - r_sh.x) * w
            current_size = width * 200
            
            # Flyv frem hvis vi er under target
            if (TARGET_SIZE - current_size) > 1500:
                fb = MAX_SPEED
            
            # Juster højde (kig efter næsen)
            nose_y = int(lan[0].y * h)
            ud = int(((h // 2) - nose_y) * 0.5)
            ud = np.clip(ud, -40, 40)

        # Tegn mål-indikator
        cv2.circle(img_display, (cx, h//2), 20, (0, 0, 255), 2)
        cv2.putText(img_display, "TARGET LOCKED", (20, 40), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 3)
    
    else:
        # SØGE-MODE (Hvis ingen er fundet)
        if flying:
            yv = 50 # Snur rundt
            cv2.putText(img_display, "SEARCHING...", (20, 40), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 255), 2)

    # 4. VISNING
    cv2.imshow("Tello Hunter Pro", img_display)

    # 5. TASTEKONTROL
    key = cv2.waitKey(1) & 0xFF
    if key == 27: # ESC
        break
    elif key == ord('t'):
        print("TAKING OFF...")
        tello.takeoff()
        # FIX FOR VIDEO-FRYS: Rens buffer efter takeoff
        time.sleep(1)
        flying = True
    elif key == ord('l'):
        tello.land()
        flying = False

    # 6. SEND KOMMANDOER
    if flying:
        tello.send_rc_control(lr, fb, ud, yv)

# --- RYD OP ---
tello.land()
tello.streamoff()
cv2.destroyAllWindows()
tello.end()
```
  </details>

---

## Logbog 08/05/26
Vi arbejdede vidre med palentir drone idéen, vi skrev noget nyt kode som benytter Ai til søgning efter personer. <br>
Vi har tilføjet således at dronen ikke styrter ind i folk men derimod holder ca 1 meters afstand fra personen. <br>
  <details>
  <summary><h2>Kode - 08/05/26</summary>

```
import cv2
import numpy as np
from djitellopy import Tello
from ultralytics import YOLO
import time

# =========================
# INDSTILLINGER
# =========================
FOLLOW_SPEED = 100        # Start-hastighed (kan ændres med +/-)
TARGET_AREA = 70000      
TRACKING = False         # Start med tracking slået FRA

# Følsomhed for AI
YAW_GAIN = 0.20
FB_GAIN = 0.0008
UD_GAIN = 0.25

# =========================
# INITIALISERING
# =========================
tello = Tello()
try:
    tello.connect()
    print(f"Batteri: {tello.get_battery()}%")
    tello.streamon()
    time.sleep(1)
except Exception as e:
    print(f"Fejl ved forbindelse: {e}")

# YOLO model
model = YOLO("yolov8n.pt") 

def get_person(results):
    best_box, best_area = None, 0
    for r in results:
        for box in r.boxes:
            if int(box.cls[0]) == 0: 
                x1, y1, x2, y2 = map(int, box.xyxy[0])
                area = (x2 - x1) * (y2 - y1)
                if area > best_area:
                    best_area = area
                    best_box = (x1, y1, x2, y2)
    return best_box, best_area

is_flying = False
keep_running = True

# =========================
# HOVEDLØKKE
# =========================
try:
    while keep_running:
        frame_container = tello.get_frame_read()
        if frame_container is None: continue
        frame_raw = frame_container.frame
        if frame_raw is None: continue

        frame = cv2.cvtColor(frame_raw, cv2.COLOR_RGB2BGR)
        h, w, _ = frame.shape

        # AI Analyse (Kører altid så du kan se boksen)
        results = model(frame, verbose=False)
        person_box, area = get_person(results)

        # Nulstil hastigheder for hvert loop
        lr, fb, ud, yv = 0, 0, 0, 0

        # --- TASTATUR INPUT ---
        key = cv2.waitKey(1) & 0xFF
        
        # Basis kommandoer
        if key == ord('z'):
            tello.takeoff()
            is_flying = True
        elif key == ord('l'):
            tello.land()
            is_flying = False
        elif key == 27: # ESC
            keep_running = False 

        # Manuel styring (W, S, A, D, Q, E, R, F)
        if key == ord('w'): fb = FOLLOW_SPEED
        elif key == ord('s'): fb = -FOLLOW_SPEED
        elif key == ord('a'): lr = -FOLLOW_SPEED
        elif key == ord('d'): lr = FOLLOW_SPEED
        elif key == ord('q'): yv = -FOLLOW_SPEED
        elif key == ord('e'): yv = FOLLOW_SPEED
        elif key == ord('r'): ud = FOLLOW_SPEED
        elif key == ord('f'): ud = -FOLLOW_SPEED

        # Indstillinger
        elif key == ord('p'): 
            TRACKING = not TRACKING
            print(f"AI Tracking: {TRACKING}")
        elif key == ord('+') or key == ord('='):
            FOLLOW_SPEED = min(FOLLOW_SPEED + 10, 100)
            print(f"Hastighed øget til: {FOLLOW_SPEED}")
        elif key == ord('-'):
            FOLLOW_SPEED = max(FOLLOW_SPEED - 10, 10)
            print(f"Hastighed sænket til: {FOLLOW_SPEED}")

        # --- AI LOGIK (Kun hvis ingen manuelle taster trykkes og TRACKING er ON) ---
        manual_active = any([fb, lr, ud, yv])
        
        if TRACKING and is_flying and not manual_active and person_box is not None:
            x1, y1, x2, y2 = person_box
            cx, cy = (x1 + x2) // 2, (y1 + y2) // 2
            
            cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 0, 255), 2)
            
            error_x = cx - w // 2
            error_y = cy - h // 2
            
            yv = int(np.clip(error_x * YAW_GAIN, -FOLLOW_SPEED, FOLLOW_SPEED))
            ud = int(np.clip(-error_y * UD_GAIN, -40, 40))
            
            if area < TARGET_AREA:
                fb = FOLLOW_SPEED
            
            cv2.putText(frame, "AI TRACKING: ACTIVE", (20, 40), 0, 0.8, (0, 0, 255), 2)

        # Send kommandoer til dronen
        if is_flying:
            tello.send_rc_control(lr, fb, ud, yv)

        # --- HUD / VISUALS ---
        cv2.putText(frame, f"Speed: {FOLLOW_SPEED} | AI: {TRACKING}", (10, h-80), 0, 0.6, (0, 255, 0), 2)
        cv2.putText(frame, f"Bat: {tello.get_battery()}% | T: Takeoff | L: Land", (10, h-50), 0, 0.6, (255, 255, 0), 2)
        cv2.putText(frame, "W/S/A/D/Q/E/R/F: Control | ESC: Quit", (10, h-20), 0, 0.6, (255, 255, 255), 1)
        
        if person_box is not None:
             cv2.rectangle(frame, (person_box[0], person_box[1]), (person_box[2], person_box[3]), (0, 255, 0), 2)

        cv2.imshow("Tello Full Control Mode", frame)

except Exception as e:
    print(f"Fejl: {e}")

finally:
    print("Shutting down...")
    if is_flying:
        tello.land()
    tello.streamoff()
    cv2.destroyAllWindows()
```
</details>

---

</details>

<details>
<summary><h1>Bilag</h1></summary>
Med evnen til at AI-genkende personer, kan vi udnytte dette til at genkende objekter, såsom et tebrev.

---

## Dronens AI-tracking
https://github.com/user-attachments/assets/9092c751-88ed-411e-94b7-067bfb7bbe87

---

## Dronen med udstyr
<img width="3024" height="4032" alt="IMG_5469" src="https://github.com/user-attachments/assets/4ca01080-9d02-4b84-858b-e3d60ad6f110" />

---

<img width="3024" height="4032" alt="IMG_5470" src="https://github.com/user-attachments/assets/1eed3304-8e7b-4e10-8c10-5e6e75688a49" />

---

<img width="3024" height="4032" alt="IMG_5471" src="https://github.com/user-attachments/assets/6da7ed00-ac11-44a5-917e-c1e2ef1fc9c6" />

---

</details>
