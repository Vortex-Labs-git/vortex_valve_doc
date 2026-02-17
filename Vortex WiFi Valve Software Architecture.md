# Vortex WiFi Valve Software Architecture

[**Mobile App And WebServer Communication (REST / WebSocket)	2**](#mobile-app-and-webserver-communication-\(rest-/-websocket\))

[User Authentication & Device Data Flow	2](#user-authentication-&-device-data-flow)

[1\. User Credential Provisioning	2](#1.-user-credential-provisioning)

[2\. Mobile App Login (REST API)	2](#2.-mobile-app-login-\(rest-api\))

[3\. WebSocket Connection Initialization	3](#3.-websocket-authentication-using-a-bearer-token-\(jwt\)-connection-initialization)

[4\. Device Data Push (via WebSocket)	3](#4.-device-data-push-\(via-websocket\))

[User Device Adding process	4](#user-device-adding-process)

[WiFi Valve Details Screen \- Data visualizing	5](#wifi-valve-details-screen---data-visualizing-\(-websocket\))

[1\. User Opens Valve Device	5](#1.-user-opens-valve-device)

[2\. Server Response Messages	5](#2.-server-response-messages)

[3\. UI Rendering Logic in the Mobile App	7](#3.-ui-rendering-logic-in-the-mobile-app)

[WiFi Valve Details Screen \- Data editing	8](#wifi-valve-details---data-editing-\(rest-with-jwt-token-\))

[1\. Valve Basic Data Editing	8](#1.-valve-basic-data-editing-via-rest)

[2\. Valve Schedule or Sensor Editing	8](#2.-valve-schedule-or-sensor-editing-via-rest)

[**Mobile App And ESP32 Communication (Websocket)	10**](#mobile-app-and-esp32-communication-\(websocket\))

[Connection Establishment	10](#connection-establishment)

[1\. Identify the Device	10](#1.-identify-the-device)

[2\. ESP32 response	10](#2.-esp32-response)

[WiFi Valve Details Screen \- Data visualizing	11](#wifi-valve-details-screen---data-visualizing)

[1\. User Opens a Valve Device	11](#1.-user-opens-a-valve-device)

[2\. ESP32 Response Messages	11](#2.-esp32-response-messages)

[WiFi Valve Details Screen \- Data editing	12](#wifi-valve-details-screen---data-editing)

[1\. Valve Basic Data Editing	12](#1.-valve-basic-data-editing)

[2\. Configure the Valve STA mode wifi credentials	13](#2.-configure-the-valve-sta-mode-wifi-credentials)

[**WebServer And ESP32 Communication (MQTT)	14**](#webserver-and-esp32-communication-\(mqtt\))

[MQTT topic structure	14](#mqtt-topic-structure)

[Topic Descriptions and Message Flow	15](#topic-descriptions-and-message-flow)

[1\. Device Status Topic	15](#1.-device-status-topic)

[2\. Valve State Data Topic	15](#2.-valve-state-data-topic)

[3\. Device Error Topic	16](#3.-device-error-topic)

[4\. Command Data Topic	17](#4.-command-data-topic)

[5\. Control Data Topic	18](#5.-control-data-topic)

# **Mobile App And WebServer Communication (REST / WebSocket)** {#mobile-app-and-webserver-communication-(rest-/-websocket)}

## User Authentication & Device Data Flow {#user-authentication-&-device-data-flow}

### 1\. User Credential Provisioning {#1.-user-credential-provisioning}

* The **Vortex Network Admin** creates a **username** and **password**.  
* Credentials are sent to the user **via email**.  
* The mobile app uses these credentials for login.


### 2\. Mobile App Login (REST API) {#2.-mobile-app-login-(rest-api)}

Login Request

* When the user enters their username and password and taps **Sign In**, the mobile app sends a REST post request to **login.php** :  
```json
  **{**  
    **"username": "testuser",**  
    **"password": "mypassword123"**  
  **}**
```
Server Response (Successful Authentication)

* If the credentials are valid, the server responds with:  
  **{**  
    **"success": true,**  
    **"message": "Login successful",**  
    **"access\_token": "\<jwt\_token\_here\>",**  
    **"refresh\_token": "no\_refresh",**  
    **"user": {**  
      **"id": "\<user\_id\>",**  
      **"name": "\<user\_name\>",**  
      **"email": "\<user\_email\>",**  
      **"contact": null**  
    **}**  
  **}**  
* The **access token** is used for authenticated API calls.  
* The **refresh token** is used to obtain new access tokens without re-login.

### 3\. WebSocket authentication using a **Bearer** token (JWT) Connection Initialization {#3.-websocket-authentication-using-a-bearer-token-(jwt)-connection-initialization}

* JWT-based authentication during the WebSocket handshake. Once authentication succeeds, the mobile app connects to the server via **WebSocket**. The WebSocket layer already identifies the user (typically via headers, token authentication, or parameters).

### 4\. Device Data Push (via websocket) {#4.-device-data-push-(via-websocket)}

When ui at device list Home page a subscribe msg send to server via websocket.

**{**  
  **"event": "subscribe",**  
  **"process": "device\_list"**  
**}**

* The server determines which devices belong to the authenticated user.  
* It queries the **device info table** based on the user JWT key.  
* It sends the initial device data to the mobile app in a structured WebSocket message:  
   **{**  
      **"event": "devices\_data",**  
      **"user\_id": "uid001",**  
      **"devices": 2,**  
      **"device\_list": \[**  
          **{**  
              **"id": "VA202601001",**  
              **"vwv\_name": "MainValve01"**  
          **},**  
          **{**  
              **"id": "VA202601002",**  
              **"vwv\_name": "test\_2"**  
          **}**  
      **\],**  
      **"timestamp": "2026-02-10T20:27:59+00:00"**  
  **}**

* By extracting device list data Mobile App will display user device list on Home page  
* This msg will send to app at every 2 second by websocket. And this process only trigger when user send that **device\_list** subscribe msg. After send this msg websocket will continuously send **device\_list** msg until the webscoket close or another subscribe msg send (**device\_detail**)

## User Device Adding process {#user-device-adding-process}

* In the current system design, **end-users do not have permission** to add or register Vortex devices to their accounts. Device assignment is managed exclusively by the **Vortex Network Admin** through the administrative interface or backend tools

* When a Mobile app receives the initial device data , the App needs to save them on local space to show the user devices even in an offline state. Future help for operate on ESP32 AP mode

## WiFi Valve Details Screen \- Data visualizing ( websocket) {#wifi-valve-details-screen---data-visualizing-(-websocket)}

### 1\. User Opens Valve Device {#1.-user-opens-valve-device}

* When the user taps a WiFi valve device layer on the Home screen, the mobile app sends a request to the WebSocket server to fetch the valve's detailed information.

   **{**

    **"event": "subscribe",**

    **"process": "device\_detail",**

    **"device\_id": "VA202601003"**

  **}**

### 2\. Server Response Messages {#2.-server-response-messages}

After receiving the request, the server check the user and devices and sends **two types of messages**:

* **Basic Valve Data**

  **{**

      **"event": "device\_basic\_detail",**

      **"device\_id": "VA202601003",**

      **"data": {**

          **"id": "VA202601003",**

          **"user\_id": "uid002",**

          **"vwv\_name": "valve\_1",**

          **"vwv\_version": "v\_1.0",**

          **"vwv\_last\_seen": null,**

          **"vwv\_is\_close": 0,**

          **"vwv\_is\_open": 0,**

          **"vwv\_ol\_active": 0,**

          **"vwv\_ol\_state": 0,**

          **"vwv\_cl\_active": 0,**

          **"vwv\_cl\_state": 0,**

          **"vwv\_pos": 0,**

          **"user\_set\_pos": 0,**

          **"user\_vwv\_pos": 0,**

          **"user\_sensor\_ctrl": 0,**

          **"user\_schedule\_ctrl": 0,**

          **"sensor\_id": null**

      **},**

      **"timestamp": "2026-02-10T20:50:12+00:00"**

  **}**

* **Control Valve Data**

  **{**


  **}**

* When user send **device\_detail** subscribe msg to server via websocket it continuously send following msg at every 2 second. This only stops when user stop the websocket connection or send the **device\_list** msg to server.   
* So when the app start and after the websocket start it continuously send one of the type of data set to the user app.

### 3\. UI Rendering Logic in the Mobile App {#3.-ui-rendering-logic-in-the-mobile-app}

Case 1 — Both `schedule` and `sensor` are FALSE

* **Only basic valve UI is shown**  
* App extracts data from **`valve_basic_data` only**  
* Sections related to scheduling and sensor settings are **hidden**

                                         

Case 2 — Either `schedule` OR `sensor` is TRUE

## WiFi Valve Details \- Data editing (REST with JWT token ) {#wifi-valve-details---data-editing-(rest-with-jwt-token-)}

### 1\. Valve Basic Data Editing via REST {#1.-valve-basic-data-editing-via-rest}

Valve nickname

* User can edit via an **“Edit”** button in the details section.

Valve control

* User can operate the valve using:  
* Fully Open / Fully Close  
* Set a specific angle

When the user edits the valve nickname or changes the angle,  
This msg send to server **control\_device.php** post request  
**{**  
   **"event": "set\_valve\_basic",**  
   **"timestamp": "2025-01-15T10:30:00Z",**  
    **"device\_id": "dev0016",**  
    **"set\_controller": {**  
    	**"schedule": false,**  
    	**"sensor": false**  
    **},**  
    **"valve\_data": {**  
    	**"name": "MainValve01",**  
    	**"set\_angle": true,**  
    	**"angle": 45**  
    **},**  
    **"ota\_update": false**  
**}**

### 2\. Valve Schedule or Sensor Editing  via REST {#2.-valve-schedule-or-sensor-editing-via-rest}

Editable Sections

* Schedule: Add/Edit/Delete schedule entries (day, time, action)  
* Sensor: Update sensor IDs and upper/lower limit thresholds

When the user modifies schedule/sensor settings and taps **“Save”**, this message send to server **control\_device.php** post request

**{**  
    **"event": "set\_valve\_control",**  
	    **"timestamp": "2025-01-15T10:30:00Z",**  
    **"device\_id": "dev0016",**  
    **"set\_controllerdata": {**  
    	**"schedule": true,**  
    	**"sensor": false**  
    **},**  
    **"set\_sheduledata": {**  
    	**"set\_shedule": true,**  
    	**"schedule\_info": \[**  
     	 **{**  
        		**"day": "Monday",**  
        		**"open": "08:00",**  
        		**"close": "08.20"**  
      	**},**  
    	**{**  
        		**"day": "Monday",**  
        		**"open": "18:00",**  
       	 	**"close": "18.20"**  
      	**}**  
    	**\]**  
    **},**  
    **"set\_sensordata": {**  
    	**"sensor\_id": "sensor-01",**  
    	**"upper\_limit": 80,**  
    	**"upper\_limit": 30**  
    **}**  
**}**

If update success this will reply

**{**  
    **"success": true,**  
    **"message": "Valve control configuration updated"**  
**}**

# **Mobile App And ESP32 Communication (Websocket)** {#mobile-app-and-esp32-communication-(websocket)}

## Connection Establishment {#connection-establishment}

### 1\. Identify the Device {#1.-identify-the-device}

* When the ESP32 is in **AP mode**, the user must first connect their mobile device to the ESP32 Wi-Fi network.  
* Then, the user opens the **Vortex Lab mobile app**.  
   The app checks the connected Wi-Fi SSID and identifies whether it is connected to a **Vortex device Wi-Fi**. If confirmed, the app sends a request to the ESP32 asking for device information.

  **{**

  	**"event": "request\_device\_info",**

  	**"timestamp": "2025-01-15T10:30:00Z",**

    	**"user\_id": "user\_id",**

    	**"passkey": "key"**

  **}**

### 2\. ESP32 response {#2.-esp32-response}

* The ESP32 reads the request and, if authorized, responds with the device information.

**{**  
**"event": "device\_info",**  
**"timestamp": "2025-01-15T10:30:00Z",**  
**"device\_id": "dev0016"**  
**}**

* On the mobile app side, the app already knows which **device IDs belong to the user**, as this data is stored locally.  
* On the **Home screen**, devices are displayed:  
  * Devices in **offline mode** are indicated with a **red line**  
  * If the connected ESP32 device belongs to the user, it is indicated with a **green line**  
* After this, the user can control the **Vortex Labs Wi-Fi valve** in **offline mode**.

## WiFi Valve Details Screen \- Data visualizing {#wifi-valve-details-screen---data-visualizing}

### 1\. User Opens a Valve Device {#1.-user-opens-a-valve-device}

* When the user taps a Wi-Fi valve device tile on the Home screen, the mobile app sends a message to the ESP32. Need to send this to get valve data

  **{**

    **"event": "device\_basic\_info",**

    **"timestamp": "2025-01-15T10:30:00Z",**

    **"data": {**

    **"user\_id": "useer001",**

    **"device\_id": "dev0016",**

    **"device\_name": "home valve"**

  	  **}**

  **}**

### 2\. ESP32 Response Messages {#2.-esp32-response-messages}

* After receiving the request, the ESP32 responds with the valve’s current data.

  **{**

      **"event": "valve\_data",**

      **"timestamp": "2025-01-15T10:30:00Z",**

      **"device\_id": "dev0016",**

      **"get\_controller": {**

      	**"schedule": true,**

      	**"sensor": false**

      **},**

      **"get\_valvedata": {**

      	**"angle": 45,**

      	**"is\_open": true,**

      	**"is\_close": false**

      **},**

      **"get\_limitdata": {**

      	**"is\_open\_limit": true,**

      	**"open\_limit": false,**

      	**"is\_close\_limit": false,**

      	**"close\_limit": true**

      **},**

      **"Error": ""                                }**

## WiFi Valve Details Screen \- Data editing {#wifi-valve-details-screen---data-editing}

### 1\. Valve Basic Data Editing {#1.-valve-basic-data-editing}

The same **Valve Details screen** is used, but the user is only allowed to control the valve.

Valve control

* Fully Open / Fully Close  
* Set a specific angle

When the user edits the **valve nickname** or changes the **valve angle**, the following message is sent to the ESP32.

**{**  
   **"event": "set\_valve\_basic",**  
    **"timestamp": "2025-01-15T10:30:00Z",**  
    **"device\_id": "dev0016",**  
    **"set\_controller": {**  
    	**"schedule": false,**  
    	**"sensor": false**  
    **},**  
    **"valve\_data": {**  
    	**"name": "MainValve01",**  
    	**"set\_angle": true,**  
    	**"angle": 45**  
    **},**  
    **"ota\_update": false**  
**}**

### 2\. Configure the Valve STA mode wifi credentials {#2.-configure-the-valve-sta-mode-wifi-credentials}

By clicking **Change Wi-Fi Connection**, the user can update the valve’s **STA mode Wi-Fi credentials**.

A popup appears requesting Wi-Fi details. When the user clicks the **Change** button, the following message is sent to the ESP32.

**{**  
   **"event": "set\_valve\_wifi",**  
    **"timestamp": "2025-01-15T10:30:00Z",**  
    **"device\_id": "dev0016",**  
    **"wifi\_data": {**  
    	**"ssid": "myNetWork",**  
    	**"password":"1234"**  
    **}**  
**}**

# **WebServer And ESP32 Communication (MQTT)** {#webserver-and-esp32-communication-(mqtt)}

 

## MQTT topic structure {#mqtt-topic-structure}

* The base topic for all Wi-Fi valve communication starts with:

  **vortex\_device/wifi\_valve/{device\_id}/**

* Main Valve Topics

  **vortex\_device/wifi\_valve/{device\_id}/status**

  **vortex\_device/wifi\_valve/{device\_id}/state\_data**

  **vortex\_device/wifi\_valve/{device\_id}/error**

  **vortex\_device/wifi\_valve/{device\_id}/cmd\_data**

  **vortex\_device/wifi\_valve/{device\_id}/control\_data**




## Topic Descriptions and Message Flow {#topic-descriptions-and-message-flow}

### 1\. Device Status Topic {#1.-device-status-topic}

**Topic:**  
**vortex\_device/wifi\_valve/{device\_id}/status**

**Published by:** ESP32 device

**Subscribed by:** Web server

**Message**:

**{**  
	    **"event": "valve\_status",**  
	    **"timestamp": "2025-01-15T10:30:00Z",**  
   	    **"device\_id": "dev0016",**  
    	    **"status": "online",**  
	**}**

**Description:**  
 When the ESP32 is running and connected to the MQTT broker, the device publishes its **online status** to this topic

### 2\. Valve State Data Topic {#2.-valve-state-data-topic}

**Topic:**  
**vortex\_device/wifi\_valve/{device\_id}/state\_data**

**Published by:** ESP32 device

**Subscribed by:** Web server

**Message**:

**{**  
    **"event": "valve\_basic\_data",**  
    **"timestamp": "2025-01-15T10:30:00Z",**  
    **"device\_id": "dev0016",**  
    **"get\_controller": {**  
    	**"schedule": true,**  
    	**"sensor": false**  
    **},**  
    **"get\_valvedata": {**  
    	**"angle": 45,**  
    	**"is\_open": true,**  
    	**"is\_close": false**  
    **},**  
    **"get\_limitdata": {**  
    	**"is\_open\_limit": true,**  
    	**"open\_limit": false,**  
    	**"is\_close\_limit": false,**  
    	**"close\_limit": true**  
    **},**  
    **"Error": ""                                }**

**Description:**  
 The ESP32 publishes **basic valve state data** to this topic.  
 Updates can be sent:

* Periodically (for example, every second), or  
* Only when the valve data changes

### 3\. Device Error Topic {#3.-device-error-topic}

**Topic:**  
**vortex\_device/wifi\_valve/{device\_id}/error**

**Published by:** ESP32 device

**Subscribed by:** Web server

**Message:**

**{**  
	    **"event": "valve\_error",**  
	    **"timestamp": "2025-01-15T10:30:00Z",**  
   	    **"device\_id": "dev0016",**  
    	    **"error": "",**  
	**}**

**Description:**  
 If an error occurs on the ESP32 device, error information is published to this topic.

###  4\. Command Data Topic {#4.-command-data-topic}

**Topic:**  
**vortex\_device/wifi\_valve/{device\_id}/cmd\_data**

**Published by:** Web server

**Subscribed by:** ESP32 device

**Message:**

**{**  
   **"event": "set\_valve\_basic",**  
    **"timestamp": "2025-01-15T10:30:00Z",**  
    **"device\_id": "dev0016",**  
    **"set\_controller": {**  
    	**"schedule": false,**  
    	**"sensor": false**  
    **},**  
    **"valve\_data": {**  
    	**"name": "MainValve01",**  
    	**"set\_angle": true,**  
    	**"angle": 45**  
    **},**  
    **"ota\_update": false**  
**}**

**Description:**  
 Basic valve command data from the **database or application logic** is published to this topic. The ESP32 listens to this topic and executes the received commands.

### 5\. Control Data Topic {#5.-control-data-topic}

**Topic:**  
**vortex\_device/wifi\_valve/{device\_id}/control\_data**

**Published by:** Web server

**Subscribed by:** ESP32 device

**Message:**

**{**  
    **"event": "set\_valve\_control",**  
    **"timestamp": "2025-01-15T10:30:00Z",**  
    **"device\_id": "dev0016",**  
    **"set\_controllerdata": {**  
    	**"schedule": true,**  
    	**"sensor": false**  
    **},**  
    **"set\_sheduledata": {**  
    	**"set\_shedule": true,**  
    	**"schedule\_info": \[**  
     	**{**  
        		**"day": "Monday",**  
        		**"open": "08:00",**  
        		**"close": "08.20"**  
      	**},**  
    	**{**  
        		**"day": "Monday",**  
        		**"open": "18:00",**  
       	 	**"close": "18.20"**  
      	**}**  
    	**\]**  
    **},**  
    **"set\_sensordata": {**  
    	**"sensor\_id": "sensor-01",**  
    	**"upper\_limit": 80,**  
    	**"lower\_limit": 30**  
    **}**  
**}**

**Description:**  
 Control-related data from the **database or user actions** is published to this topic. This includes valve control operations such as open, close, or setting a specific angle.
