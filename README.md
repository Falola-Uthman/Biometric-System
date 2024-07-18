# Biometric System



## Objective

The primary aim of the Fingerprint Biometric System project was to develop a secure access control system using fingerprint recognition technology. The goal was to design, implement, and test a system that enhances security while providing a seamless and user-friendly experience. This project aimed to address the limitations of traditional access control methods and provide a robust solution for environments requiring stringent security measures.

## Skills Learned
- Biometric Security Implementation: Gained hands-on experience in designing and implementing a biometric security system to control access to secure areas.
- Access Control Management: Improved my understanding of implementing secure access control systems using biometric authentication.
- Threat Modeling: Enhanced my ability to identify potential security threats and vulnerabilities associated with biometric systems spoofing and replay attacks.
- System Integration: Enhanced my skills in integrating multiple hardware components, such as fingerprint scanners, motion sensors, LEDs, and LCD displays, with a central processing unit.
- Testing and Validation: Sharpened my skills in conducting comprehensive testing to validate system performance, accuracy, and reliability.


## Tools Used
- Raspberry Pi: Served as the central processing unit for coordinating the activities of all connected hardware components.
- Python: Utilized for developing the software components, including the enrollment and authentication scripts.
- SQLite: Used for managing the user database and storing fingerprint templates securely.
- PIR Motion Sensor: Integrated to activate the fingerprint scanner only when motion is detected, enhancing security and power efficiency.
- Fingerprint Scanner: Core component for capturing and processing fingerprint images.
- LED Indicators: Provided immediate visual feedback to users about the status of their authentication attempts.
- LCD Display: Guided users through the enrollment and authentication processes with real-time prompts and messages.


## Steps
### Physical Connection

- PIR Motion Sensor:
    --VCC: Connected to the 5V power pin on the Raspberry Pi.
    --GND: Connected to a ground pin on the Raspberry Pi.
    --OUT: Connected to GPIO pin 17 on the Raspberry Pi.
- Fingerprint Scanner:
    --VCC: Connected to the 3.3V or 5V power pin on the Raspberry Pi.
    --GND: Connected to a ground pin on the Raspberry Pi.
    --TX: Connected to the GPIO pin configured for UART RX (GPIO 15 / RXD).
    --RX: Connected to the GPIO pin configured for UART TX (GPIO 14 / TXD).
- LEDs (Red and Green):
    --Red LED:
    --Anode: Connected to GPIO pin 27 on the Raspberry Pi. 
    --Cathode: Connected to a ground pin on the Raspberry Pi through a current-limiting resistor (220 ohms).
    --Green LED:
    --Anode: Connected to GPIO pin 18 on the Raspberry Pi through a current-limiting resistor.
    --Cathode: Connected to a ground pin on the Raspberry Pi.
- LCD Display:
    --VCC: Connected to the 5V power pin on the Raspberry Pi.
    --GND: Connected to a ground pin on the Raspberry Pi.
    --SDA: Connected to the GPIO pin configured for I2C data (GPIO 2 / SDA).
    --SCL: Connected to the GPIO pin configured for I2C clock (GPIO 3 / SCL).

## Visual of Biometric System

![CIrcuit diagram](https://github.com/user-attachments/assets/132c9792-1d32-49c8-98ee-ba5386771ef5)

Ref 1: Circuit Diagram of Biometric System.

### Biometric System Script


![Scrsht](https://github.com/user-attachments/assets/54da2f07-1707-496e-9d2e-4682862a0419)

Ref 2: Biometric System GUI.

![Scrsht2](https://github.com/user-attachments/assets/7c72d984-de33-4f5c-863f-456643a38e4f)

Ref 3: Biometric System Logs.

### Enrollment Script


import serial
import time
import RPi.GPIO as GPIO
import re
from rpi_lcd import LCD
import sqlite3
import adafruit_fingerprint


PIR_PIN = 17
RED_LED_PIN = 27
GREEN_LED_PIN = 18


GPIO.setmode(GPIO.BCM)
GPIO.setup(RED_LED_PIN, GPIO.OUT)
GPIO.setup(GREEN_LED_PIN, GPIO.OUT)


lcd = LCD()


uart = serial.Serial("/dev/serial0", baudrate=57600, timeout=1)
finger = adafruit_fingerprint.Adafruit_Fingerprint(uart)


def initialize_database():
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, fingerprint_id INTEGER)''')
    conn.commit()
    conn.close()

def add_user_to_database(user_id, name, fingerprint_id):
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    cursor.execute('''INSERT INTO users (id, name, fingerprint_id) VALUES (?, ?, ?)''', (user_id, name, fingerprint_id))
    conn.commit()
    conn.close()

def delete_user_from_database_by_id(user_id):
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    cursor.execute('''DELETE FROM users WHERE id = ?''', (user_id,))
    conn.commit()
    conn.close()

def delete_user_from_database_by_name(name):
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    cursor.execute('''DELETE FROM users WHERE name = ?''', (name,))
    conn.commit()
    conn.close()

def delete_user_from_database_by_fingerprint(fingerprint_id):
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    cursor.execute('''DELETE FROM users WHERE fingerprint_id = ?''', (fingerprint_id,))
    conn.commit()
    conn.close()

def view_users():
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    cursor.execute('''SELECT id, name FROM users''')
    users = cursor.fetchall()
    conn.close()
    if users:
        print("Current users:")
        for user in users:
            print(f"ID: {user[0]}, Name: {user[1]}")
        for user in users:
            lcd.text(f"ID: {user[0]}", 1)
            lcd.text(f"Name: {user[1]}", 2)
            time.sleep(2)
            lcd.clear()
    else:
        print("No users found.")
        lcd.text("No users found.", 1)
        time.sleep(2)
        lcd.clear()

def find_user():
    option = input("Search by ID (i), Name (n), or Fingerprint (f)? ")
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    if option == 'i':
        user_id = int(input("Enter user ID: "))
        cursor.execute('''SELECT id, name FROM users WHERE id = ?''', (user_id,))
    elif option == 'n':
        name = input("Enter user name: ")
        cursor.execute('''SELECT id, name FROM users WHERE name = ?''', (name,))
    elif option == 'f':
        lcd.text("Place finger", 1)
        while finger.get_image() != adafruit_fingerprint.OK:
            pass
        if finger.image_2_tz(1) != adafruit_fingerprint.OK:
            lcd.text("Error reading", 2)
            return
        if finger.finger_search() != adafruit_fingerprint.OK:
            lcd.text("Not recognized", 2)
            return
        cursor.execute('''SELECT id, name FROM users WHERE fingerprint_id = ?''', (finger.finger_id,))
    user = cursor.fetchone()
    conn.close()
    if user:
        print(f"ID: {user[0]}, Name: {user[1]}")
        lcd.text(f"ID: {user[0]}", 1)
        lcd.text(f"Name: {user[1]}", 2)
    else:
        print("User not found.")
        lcd.text("User not found.", 1)
    time.sleep(2)
    lcd.clear()

def enroll_finger(user_id, name):
    # Enroll the fingerprint
    for fingerimg in range(1, 3):
        if fingerimg == 1:
            lcd.text("Place finger", 1)
        else:
            lcd.text("Place same finger", 1)
        while True:
            i = finger.get_image()
            if i == adafruit_fingerprint.OK:
                lcd.text("Image taken", 2)
                break
            elif i == adafruit_fingerprint.NOFINGER:
                lcd.text("No finger detected", 2)
            else:
                lcd.text("Other error", 2)

        lcd.text("Templating...", 2)
        i = finger.image_2_tz(fingerimg)
        if i == adafruit_fingerprint.OK:
            lcd.text("Templated", 2)
        else:
            lcd.text("Error templating", 2)
            GPIO.output(RED_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.LOW)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.LOW)
            return False

        lcd.text("Remove finger", 2)
        time.sleep(1)
        while i != adafruit_fingerprint.NOFINGER:
            i = finger.get_image()

    lcd.text("Creating model...", 2) gggggggggg
    i = finger.create_model()
    if i == adafruit_fingerprint.OK:
        lcd.text("Model created", 2)
    else:
        lcd.text("Prints did not match", 2)
        GPIO.output(RED_LED_PIN, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(RED_LED_PIN, GPIO.LOW)
        time.sleep(1)
        GPIO.output(RED_LED_PIN, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(RED_LED_PIN, GPIO.LOW)
        return False

    fingerprint_id = user_id  # Assuming user_id is used as fingerprint_id for simplicity

    lcd.text("Storing model...", 2)
    i = finger.store_model(fingerprint_id)
    if i == adafruit_fingerprint.OK:
        lcd.text("Stored", 2)
        add_user_to_database(user_id, name, fingerprint_id)
        GPIO.output(GREEN_LED_PIN, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(GREEN_LED_PIN, GPIO.LOW)
        time.sleep(1)
        GPIO.output(GREEN_LED_PIN, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(GREEN_LED_PIN, GPIO.LOW)
        return True
    else:
        lcd.text("Failed to store", 2)
        GPIO.output(RED_LED_PIN, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(RED_LED_PIN, GPIO.LOW)
        time.sleep(1)
        GPIO.output(RED_LED_PIN, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(RED_LED_PIN, GPIO.LOW)
        return False

def delete_user():
    option = input("Delete by ID (i), Name (n), or Fingerprint (f)? ")
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    user = None
    if option == 'i':
        user_id = int(input("Enter user ID: "))
        cursor.execute('''SELECT id, name FROM users WHERE id = ?''', (user_id,))
        user = cursor.fetchone()
    elif option == 'n':
        name = input("Enter user name: ")
        cursor.execute('''SELECT id, name FROM users WHERE name = ?''', (name,))
        user = cursor.fetchone()
    elif option == 'f':
        lcd.text("Place finger", 1)
        while finger.get_image() != adafruit_fingerprint.OK:
            pass
        if finger.image_2_tz(1) != adafruit_fingerprint.OK:
            lcd.text("Error reading", 2)
            return
        if finger.finger_search() != adafruit_fingerprint.OK:
            lcd.text("Not recognized", 2)
            return
        cursor.execute('''SELECT id, name FROM users WHERE fingerprint_id = ?''', (finger.finger_id,))
        user = cursor.fetchone()
    if user:
        user_id, name = user
        confirm = input(f"Confirm deletion of user with ID {user_id} and name {name}? (y/n): ")
        if confirm.lower() == 'y':
            cursor.execute('''DELETE FROM users WHERE id = ?''', (user_id,))
            conn.commit()
            GPIO.output(GREEN_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(GREEN_LED_PIN, GPIO.LOW)
            time.sleep(1)
            GPIO.output(GREEN_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(GREEN_LED_PIN, GPIO.LOW)
            print("User deleted.")
            lcd.text("User deleted.", 1)
        else:
            GPIO.output(RED_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.LOW)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.LOW)
            print("Deletion cancelled.")
            lcd.text("Deletion cancelled.", 1)
    else:
        print("User not found.")
        lcd.text("User not found.", 1)
    conn.close()

initialize_database()

while True:
    print("e) Enroll")
    print("d) Delete")
    print("v) View users")
    print("f) Find user")
    option = input("Select option: ")
    if option == 'e':
        user_id = int(input("Enter ID: "))
        name = input("Enter name: ")
        if enroll_finger(user_id, name):
            print("Enrollment successful.")
            lcd.text("Enroll successful", 1)
            GPIO.output(GREEN_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(GREEN_LED_PIN, GPIO.LOW)
            time.sleep(1)
            GPIO.output(GREEN_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(GREEN_LED_PIN, GPIO.LOW)
        else:
            print("Enrollment failed.")
            lcd.text("Enroll failed", 1)
            GPIO.output(RED_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.LOW)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.HIGH)
            time.sleep(1)
            GPIO.output(RED_LED_PIN, GPIO.LOW)
    elif option == 'd':
        delete_user()
    elif option == 'v':
        view_users()
    elif option == 'f':
        find_user()
    time.sleep(2)
    lcd.clear()

### Authentication Script


import serial
import time
import RPi.GPIO as GPIO
from rpi_lcd import LCD
import sqlite3
import adafruit_fingerprint


PIR_PIN = 17
RED_LED_PIN = 27
GREEN_LED_PIN = 18


GPIO.setmode(GPIO.BCM)
GPIO.setup(RED_LED_PIN, GPIO.OUT)
GPIO.setup(GREEN_LED_PIN, GPIO.OUT)
GPIO.setup(PIR_PIN, GPIO.IN)


lcd = LCD()


uart = serial.Serial("/dev/serial0", baudrate=57600, timeout=1)
finger = adafruit_fingerprint.Adafruit_Fingerprint(uart)


def get_user_from_database(fingerprint_id):
    conn = sqlite3.connect('fingerprints.db')
    cursor = conn.cursor()
    cursor.execute('''SELECT name FROM users WHERE fingerprint_id = ?''', (fingerprint_id,))
    user = cursor.fetchone()
    conn.close()
    return user

def authenticate():
    while True:
        if GPIO.input(PIR_PIN):
            lcd.text("Welcome", 1)
            GPIO.output(RED_LED_PIN, GPIO.LOW)
            lcd.text("Place finger", 2)
            while finger.get_image() != adafruit_fingerprint.OK:
                pass
            if finger.image_2_tz(1) != adafruit_fingerprint.OK:
                lcd.text("Error reading", 2)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.LOW)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.LOW)
                return
            if finger.finger_search() != adafruit_fingerprint.OK:
                lcd.text("Not recognized", 2)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.LOW)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.LOW)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                lcd.clear()
                time.sleep(1)
                return
            user = get_user_from_database(finger.finger_id)
            if user:
                lcd.text(f"Welcome, {user[0]}", 1)
                GPIO.output(GREEN_LED_PIN, GPIO.HIGH)
                lcd.text("Access Granted", 2)
                time.sleep(5)
                GPIO.output(GREEN_LED_PIN, GPIO.LOW)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                lcd.clear()
                time.sleep(1)
            else:
                lcd.text("User not found", 2)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.LOW)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.HIGH)
                time.sleep(1)
                GPIO.output(RED_LED_PIN, GPIO.LOW)

GPIO.output(RED_LED_PIN, GPIO.HIGH)

while True:
    authenticate()


