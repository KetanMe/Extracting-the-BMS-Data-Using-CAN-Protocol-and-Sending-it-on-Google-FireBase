# Extracting-the-BMS-Data-Using-CAN-Protocol-and-Sending-it-on-Google-FireBase

The provided involves sending CAN messages containing various data points to a Firebase Realtime Database (RTDB) using an ESP8266 module.

## Table of Content

1. [Library Inclusions and Setup](#1-library-inclusions-and-setup)
2. [MCP2515 CAN Setup](#2-mcp2515-can-setup)
3. [WiFi and Firebase Configuration](#3-wifi-and-firebase-configuration)
4. [CAN Message Structures and Data Storage](#4-can-message-structures-and-data-storage)
5. [Data Sending Function (`Send_msg`)](#5-data-sending-function-send_msg)
6. [Setup Function](#6-setup-function)
7. [Main Loop](#7-main-loop)
8. [Summary](#summary)


### 1. Library Inclusions and Setup

```cpp
#include <SPI.h>
#include <mcp2515.h>
#include <string.h>
#include <Firebase_ESP_Client.h>
#include <ESP8266WiFi.h>
#include "addons/TokenHelper.h"
#include "addons/RTDBHelper.h"
```

- **Libraries**: 
  - `SPI.h`: Library for SPI communication, used for communication with the MCP2515 CAN controller.
  - `mcp2515.h`: Library for interfacing with the MCP2515 CAN controller.
  - `Firebase_ESP_Client.h`: Firebase client library for ESP8266, enabling interaction with Firebase services.
  - `ESP8266WiFi.h`: Library for WiFi connectivity with ESP8266.
  - `TokenHelper.h` and `RTDBHelper.h`: Additional helper libraries for Firebase authentication and Realtime Database (RTDB) operations.

### 2. MCP2515 CAN Setup

```cpp
MCP2515 mcp2515(D8);  // MCP CS Pin
```

- **MCP2515**: Initialization of the MCP2515 CAN controller object with the chip select pin `D8`.

### 3. WiFi and Firebase Configuration

```cpp
const char *WIFI_SSID = "YourWiFiSSID";
const char *WIFI_PASSWORD = "YourWiFiPassword";
const char *API_KEY = "YourFirebaseAPIKey";
const char *DATABASE_URL = "https://yourfirebaseproject.firebaseio.com/";

FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;
bool signupOK = false;
```

- **WiFi Credentials**: Configuration of WiFi network credentials (`WIFI_SSID` and `WIFI_PASSWORD`) for connecting ESP8266 to the internet.
- **Firebase Configuration**: Setup of Firebase API key (`API_KEY`) and database URL (`DATABASE_URL`) for Firebase RTDB access.
- **Firebase Objects**: `fbdo`, `auth`, and `config` are Firebase client objects used for authentication and database operations.

### 4. CAN Message Structures and Data Storage

```cpp
struct can_frame canMsg2;
struct can_frame canMsg1;

// Define various data variables for CAN messages
uint16_t pressure = 0;
uint16_t acquisition = 0;
int16_t current = 0;
uint16_t soc = 0;
// ... other variables for different data points
```

- **CAN Message Structures**: `canMsg1` and `canMsg2` are structures defined to hold CAN messages.
- **Data Variables**: Variables like `pressure`, `current`, `soc`, etc., store data received from CAN messages before sending to Firebase.

### 5. Data Sending Function (`Send_msg`)

```cpp
void Send_msg(byte index) {
  // Construct CAN ID based on index
  if (index == 0) {
    canMsg2.can_id  = 0x18900140 | CAN_EFF_FLAG;
  }
  // ... other CAN IDs based on index

  canMsg2.can_dlc = 8;
  // Initialize CAN message data
  canMsg2.data[0] = 0x00;
  canMsg2.data[1] = 0x00;
  // ... initialize other data bytes

  // Send CAN message
  mcp2515.sendMessage(&canMsg2);
}
```

- **Send_msg Function**: Sends CAN messages based on the provided index. Constructs the CAN ID and initializes data bytes, then sends the message using `mcp2515.sendMessage()`.

### 6. Setup Function

```cpp
void setup() {
  Serial.begin(9600);
  mcp2515.reset();
  mcp2515.setBitrate(CAN_250KBPS, MCP_8MHZ);
  mcp2515.setNormalMode();

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  // Wait for WiFi connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(300);
  }

  // Firebase initialization
  config.api_key = API_KEY;
  config.database_url = DATABASE_URL;
  // Firebase sign up (authentication)
  if (Firebase.signUp(&config, &auth, "", "")) {
    signupOK = true;
  }

  // Firebase begin and WiFi reconnection
  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
}
```

- **Setup Function**: Initializes serial communication, resets and configures the MCP2515 CAN controller, connects to WiFi, and initializes Firebase authentication and connection.

### 7. Main Loop

```cpp
void loop() {
  // Send CAN messages and process received data
  // Example: Send_msg(0), process response, Send_msg(1), etc.
  // Update Firebase RTDB with received data
}
```

- **Loop Function**: Continuously sends CAN messages (`Send_msg`) and processes the received data. Updates Firebase RTDB with relevant data points received from CAN messages.

### Summary

This code demonstrates how an ESP8266 with MCP2515 CAN controller sends data from various sensors or systems via CAN bus to Firebase's Realtime Database. It handles WiFi connectivity, CAN communication, Firebase authentication, and data transmission efficiently in a loop. Each `Send_msg` function call sends specific data to Firebase, facilitating real-time monitoring and storage of data from remote devices. Adjustments and error handling ensure reliable operation even in potentially unstable network conditions.
