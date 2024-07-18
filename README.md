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

### Imports
-	import serial: For serial communication with the fingerprint scanner.
-	import time: Provides time-related functions for delays and timeouts.
-	import RPi.GPIO as GPIO: Controls the GPIO pins on the Raspberry Pi.
-	import re: Provides regular expression matching operations.
-	from rpi_lcd import LCD: Manages the LCD display connected to the Raspberry Pi.
-	import sqlite3: Handles database operations for storing and retrieving user information.
-	import adafruit_fingerprint: Interfaces with the Adafruit fingerprint sensor.

### Functions
-	initialize_database(): This function creates the SQLite database and users table if they do not already exist. It initializes the storage system for user information and fingerprint data, ensuring the database is ready to store new users.
-	add_user_to_database(user_id, name, fingerprint_id): This function inserts a new user's details into the database. It adds newly enrolled users to the system, associating their fingerprint ID with their personal details for future authentication.
-	delete_user_from_database_by_id(user_id): This function deletes a user from the database using their user ID. It allows the removal of users from the system based on their unique ID.
-	delete_user_from_database_by_name(name): This function deletes a user from the database using their name. It provides flexibility in removing users by allowing deletion based on the user's name.
-	delete_user_from_database_by_fingerprint(fingerprint_id): This function deletes a user from the database using their fingerprint ID. It allows deletion of users by directly using their biometric identifier.
-	view_users(): This function retrieves and displays a list of all users from the database. It provides an overview of all enrolled users, helping in managing and verifying the user database.
-	find_user(): This function searches for a user in the database by ID, name, or fingerprint. It allows administrators to locate user details based on various criteria, enhancing user management capabilities.
-	enroll_finger(user_id, name): This function captures and processes the fingerprint images, creates a fingerprint template, and stores the template along with the user's details in the database. It is the core function for enrolling new users into the system, ensuring that their biometric data is captured accurately and stored securely.
-	delete_user(): This function prompts the administrator to delete a user by ID, name, or fingerprint. It provides a flexible interface for removing users from the system, ensuring that the database can be maintained and updated as needed.
-   Main Loop: The main loop (initialize_database()) provides a menu for administrators to enroll users, delete users, view users, and find users. It facilitates user management by providing easy access to enrollment and administrative functions through a simple interface.


### Authentication Script


### Imports
-	import serial: For serial communication with the fingerprint scanner.
-	import time: Provides time-related functions for delays and timeouts.
-	import RPi.GPIO as GPIO: Controls the GPIO pins on the Raspberry Pi.
-	from rpi_lcd import LCD: Manages the LCD display connected to the Raspberry Pi.
-	import sqlite3: Handles database operations for storing and retrieving user information.
-	import adafruit_fingerprint: Interfaces with the Adafruit fingerprint sensor.

### Functions
-	get_user_from_database(fingerprint_id): This function connects to the SQLite database to retrieve the user details associated with a given fingerprint ID. It is crucial for verifying the identity of the user by comparing the fingerprint ID obtained during the authentication process with stored user data.
-	authenticate(): This function manages the authentication process. It checks for motion using the PIR sensor, captures and processes the fingerprint, searches the database for a match, and provides feedback through the LCD and LEDs. This ensures that only authorized users can gain access by verifying their fingerprint against the stored database while providing real-time feedback and control signals to the user.
-   Main Loop: The main loop continuously calls the authenticate() function to handle ongoing user authentication, keeping the system in a ready state to authenticate users at any time.


