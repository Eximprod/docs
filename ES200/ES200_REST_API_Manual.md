# ES200 - REST API Guide <!-- omit from toc -->

[**Table of Contents:**](#toc)

- [1. ES200 Endpoints](#1-es200-endpoints)
  - [1.1. Authenticate User](#11-authenticate-user)
  - [1.2. Update User Password](#12-update-user-password)
  - [1.3. Get Users](#13-get-users)
- [2. WebServer Endpoints](#2-webserver-endpoints)
  - [2.1. Login](#21-login)
  - [2.2. Logout](#22-logout)
  - [2.3. Fetch Log Files Names](#23-fetch-log-files-names)
  - [2.4. Fetch Log Content](#24-fetch-log-content)
  - [2.5. Get Master Equipments](#25-get-master-equipments)
    - [2.5.1. Via Rest API](#251-via-rest-api)
    - [2.5.2. Via WebSockets](#252-via-websockets)
  - [2.6. Send Commands](#26-send-commands)
  - [2.7. Download Database](#27-download-database)
  - [2.8. Upload Database](#28-upload-database)
  - [2.9. Update Certificates](#29-update-certificates)

## 1. ES200 Endpoints

### 1.1. Authenticate User

-   **Base URL**: `https://localhost:1732`
-   **Endpoint**: `/api/authentication`
-   **Method**: `POST`
-   **Content-Type**: `application/json`
-   **Description**: Authenticates a user to get the SEED value used for establishing ESRemote connections.
-   **Request Body**:

```json
{
    "username": "admin", // string
    "password": "8C6976E5B5410415BDE908BD4DEE15DFB167A9C873FC4BB8A81F6F2AB448A918" // uppercase SHA-256 string
}
```

-   **Response status codes**:

    -   `200 OK`: Returns SEED value.
    -   `400 Bad Request`: Malformed request.
    -   `401 Unauthorized`: Wrong credentials.
    -   `411 Length Required`: Empty body.

-   **Response body**:

```json
5ADC69F6C5AADF8220850C627E367293ECE4F2C1B469BD0D519E101AEB01AB29 // 64 characters Base64 string
```

-   **Example**:

```bash
curl -k -X POST https://localhost:1732/api/authentication \
	-H "Content-Type: application/json" \
	-d '{"username": "admin", "password": "8C6976E5B5410415BDE908BD4DEE15DFB167A9C873FC4BB8A81F6F2AB448A918"}'
```

### 1.2. Update User Password

-   **Base URL**: `https://localhost:1732`
-   **Endpoint**: `/api/updateUserPassword`
-   **Method**: `POST`
-   **Content-Type**: `application/json`
-   **Description**: Updates the password for an existing user.
-   **Request Body**:

```json
{
    "username": "admin", // string
    "currentPassword": "CURRENT_UPPERCASE_SHA256_HASH", // uppercase SHA-256 string
    "newPassword": "NEW_UPPERCASE_SHA256_HASH" // uppercase SHA-256 string
}
```

-   **Response status codes**:
    -   `200 OK`: Password updated successfully.
    -   `400 Bad Request`: Malformed request.
    -   `401 Unauthorized`: Incorrect current password.
    -   `411 Length Required`: Empty body.
-   **Example**:

```bash
# Update the current password (admin) of the current user (admin), with the new password (test)
curl -k -X POST https://localhost:1732/api/updateUserPassword \
      -H "Content-Type: application/json" \
      -d '{"username": "admin", "currentPassword": "8C6976E5B5410415BDE908BD4DEE15DFB167A9C873FC4BB8A81F6F2AB448A918", "newPassword": "9F86D081884C7D659A2FEAA0C55AD015A3BF4F1B2B0B822CD15D6C15B0F00A08"}'
```

### 1.3. Get Users

-   **Base URL**: `https://localhost:1732`
-   **Endpoint**: `/api/getUsers`
-   **Method**: `GET`
-   **Content-Type**: `application/json`
-   **Description**: Retrieves a list of all users.
-   **Response status codes**:
    -   `200 OK`: Successfully retrieved user list.
    -   `500 Internal Server Error`: Error in retrieving users.
-   **Response body**:

```json
[
    {
        "username": "admin" // string
    },
    {
        "username": "admin2" // string
    }
    // ... more users
]
```

-   **Example**:

```bash
curl -k -X GET https://localhost:1732/api/getUsers
```

## 2. WebServer Endpoints

### 2.1. Login

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/login`
-   **Method**: `POST`
-   **Content-Type**: `application/json`
-   **Description**: Attempts to log in a client, returning the session cookie upon success.
-   **Request Body**:

```json
{
    "username": "user", // string
    "password": "password" // string (plain text password)
}
```

-   **Response Status Codes**:
    -   `200 OK`: Successful login, returns session cookie.
    -   `429 Too Many Requests`: Too many failed login attempts.
    -   `401 Unauthorized`: Incorrect credentials.
    -   `500 Internal Server Error`: Server error occurred.
-   **Response Body**:

```json
{
    "authenticatedUser": "user" // string
}
```

-   **Example**:

```bash
curl -k -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'
```

-   **Note**:
    -   The session cookie is valid for 1800 seconds (30 minutes) by default. After this period, the session will expire, and the user will need to log in again to obtain a new session cookie.
    -   When you use `curl` to make a login request that sets an HTTP-only cookie (containing the session ID) upon successful login, this cookie is indeed managed by `curl` for the duration of the session. However, it's important to note that `curl` does not automatically include this cookie in subsequent requests unless explicitly instructed to do so.
    -   To ensure the cookie set by the `/api/login` endpoint is included in subsequent requests, such as `/api/logs`, you need to use `curl`'s cookie handling capabilities:
        -   Store the Cookie: Make the login request and save the cookie to a file. This can be done using the `-c` or `--cookie-jar` option in `curl`.
        -   Reuse the Cookie: In subsequent requests, you need to include this cookie for the server to recognize the session. This is done using the `-b` or `--cookie` option in `curl`.

```bash
# Making a login request to get the sessionId cookie
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
     -H "Content-Type: application/json" \
     -d '{"username": "admin", "password": "admin"}'

# Making a request that includes the sessionId cookie
curl -k -b cookies.txt -X GET https://localhost:3000/api/logs
```

### 2.2. Logout

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/logout`
-   **Method**: `POST`
-   **Description**: Logs out the client and invalidates the session cookie (if present)
-   **Response Status Codes**:
    -   `200 OK`: Successful logout.
-   **Example**:

```bash
curl -k -X POST https://localhost:3000/api/logout
```

### 2.3. Fetch Log Files Names

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/logs`
-   **Method**: `GET`
-   **Description**: Retrieves the names of available log files.
-   **Response Status Codes**:
    -   `200 OK`: Successfully retrieved log files names.
    -   `204 No Content`: No log files available.
    -   `401 Unauthorized`: Invalid session cookie.
-   **Response Body**:

```json
{
	logCategories": ["category1", "category2"] // string[]
}
```

-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request
curl -k -b cookies.txt -X GET https://localhost:3000/api/logs
```

```json
// Response
{
    "logCategories": [
        {
            "id": 1,
            "name": "General",
            "files": [
                {
                    "id": 2,
                    "name": "ESRemote",
                    "path": "/mnt/ramdisk/ESRemote"
                },
                {
                    "id": 3,
                    "name": "MultiDataMaster",
                    "path": "/mnt/ramdisk/MultiDataMaster"
                },
                {
                    "id": 4,
                    "name": "Watchdog",
                    "path": "/mnt/ramdisk/Watchdog"
                }
            ]
        },
        {
            "id": 5,
            "name": "Events",
            "files": [
                {
                    "id": 6,
                    "name": "MultiDataMaster",
                    "path": "/mnt/ramdisk/Events/MultiDataMaster"
                },
                {
                    "id": 7,
                    "name": "Watchdog",
                    "path": "/mnt/ramdisk/Events/Watchdog"
                }
            ]
        },
        {
            "id": 8,
            "name": "Commands",
            "files": [
                {
                    "id": 9,
                    "name": "ESRemote",
                    "path": "/mnt/ramdisk/Commands/ESRemote"
                },
                {
                    "id": 10,
                    "name": "MultiDataMaster",
                    "path": "/mnt/ramdisk/Commands/MultiDataMaster"
                }
            ]
        }
    ]
}
```

### 2.4. Fetch Log Content

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/logcontent`
-   **Method**: `GET`
-   **Description**: Fetches the content of a specified log file.
-   **Query Parameters**:

    -   `path` (required, string): The absolute path of the log file.
    -   `name` (required, string): The name of the log file.
    -   `category` (required, string): The category to which the log file belongs.

-   **Response Status Codes**:
    -   `200 OK`: Successfully retrieved log content.
    -   `204 No Content`: Log content not available.
    -   `400 Bad Request`: Missing or incorrect query parameters.
    -   `401 Unauthorized`: Invalid session cookie.
-   **Response Body**:

```json
{
    "logContent": "Log file content here..." // string
}
```

-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request
# The query parameters must be correctly URL-encoded
curl -k -b cookies.txt -X GET https://localhost:3000/api/logcontent?name=ESRemote&path=%2Fmnt%2Framdisk%2FCommands%2FESRemote&category=Commands
```

-   **Notes**:
    -   The `path` query parameter must be correctly URL-encoded. For example, if the path is `/mnt/ramdisk/Commands/ESRemote`, it should be encoded as `%2Fmnt%2Framdisk%2FCommands%2FESRemote`.
    -   The `name` and `category` query parameters should match the log file name and category returned by the `/api/logs` endpoint.
    -   The log content is returned as a single string, which may contain line breaks or other formatting characters.

### 2.5. Get Master Equipments

#### 2.5.1. Via Rest API

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/points`
-   **Method**: `GET`
-   **Description**: Retrieves the list of **Master Equipments** and their associated points. Supports filtering by `equipmentId`, `idDown`, and `pointType`.
-   **Query Parameters**:

    -   `equipmentId` (optional, number): The ID of the Master Equipment to filter points by.
    -   `idDown` (optional, number): The register address to filter points (registers) by.
    -   `pointType` (optional, string): The type of point to filter by. Valid point types are:
        -   `Binary Input`
        -   `Binary Output`
        -   `Analog Input`
        -   `Analog Output`
        -   `Double Input`
        -   `Double Output`

-   **Response Status Codes**:

    -   `200 OK`: Successfully retrieved Master Equipments and points.
    -   `204 No Content`: No data available for the given filters.
    -   `400 Bad Request`: Invalid query parameters.
    -   `401 Unauthorized`: Invalid session cookie.

-   **Response Body**:

```json
{
    "masterEquipments": [
        {
            "id": 1, // number
            "name": "MultiDataMaster", // string
            "points": [
                {
                    "description": "Restart_MultiDataMaster", // string
                    "forcedValueFlag": 0, // number
                    "idDown": 10011, // number
                    "internalTimestamp": "09:07:08:875 13/12/2024", // string
                    "pointType": "Binary Output", // string
                    "protocolTimestamp": "09:07:08:875 13/12/2024", // string
                    "status": 1, // number
                    "value": "0", // string
                    "valueType": 0 // number
                }
                // ... more points
            ],
            "process": "MultiDataMaster" // string
        }
        // ... more equipments
    ]
}
```

-   **Examples**:

1. Retrieve all Master Equipments:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request
curl -k -b cookies.txt -X GET "https://localhost:3000/api/points"
```

2. Retrieve points for a specific `equipmentId`:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request
curl -k -b cookies.txt -X GET "https://localhost:3000/api/points?equipmentId=1"
```

3. Retrieve a specific point by `idDown` and `pointType`:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request
curl -k -b cookies.txt -X GET "https://localhost:3000/api/points?equipmentId=1&idDown=10020&pointType=Binary%20Input"
```

-   **Notes**:
    -   When filtering by `pointType`, ensure the value matches one of the valid point types listed above.
    -   The `idDown` parameter represents the register address of the point. Ensure it is a valid number greater than 0.
    -   If no query parameters are provided, all available `Master Equipments` and their `points` will be returned.
    -   The query parameters should be correctly URL-encoded. For example, if the `pointType` is `Binary Input`, it should be encoded as `Binary%20Input`.

#### 2.5.2. Via WebSockets

This section outlines how to connect to the WebSocket server, authenticate using a session cookie, subscribe/unsubscribe from topics, and the expected JSON message format for data updates.

- **URL**: `wss://localhost:8443`
- **Protocol**: WebSocket Secure (WSS)
- **Authentication**: Requires a session cookie obtained from the `/api/login` HTTP endpoint.
- **Handshake**: During the WebSocket handshake, the `sessionId` cookie must be included in the request headers.
- **Subscribing/Unsubscribing to Topics**

```json
{
    "type": "subscribe",
    "topic": "entityViewer"
}
```
```json
{
    "type": "unsubscribe",
    "topic": "entityViewer"
}
```

- **Receiving Data**

The WebSockets server pushes real-time updates when subscribed to topics.

- **Response Example**:
```json
{
  "data": [
        {
            "id": 1, // number
            "name": "MultiDataMaster", // string
            "points": [
                {
                    "description": "Restart_MultiDataMaster", // string
                    "forcedValueFlag": 0, // number
                    "idDown": 10011, // number
                    "internalTimestamp": "09:07:08:875 13/12/2024", // string
                    "pointType": "Binary Output", // string
                    "protocolTimestamp": "09:07:08:875 13/12/2024", // string
                    "status": 1, // number
                    "value": "0", // string
                    "valueType": 0 // number
                }
                // ... more points
            ],
            "process": "MultiDataMaster" // string
        }
        // ... more equipments
    ],
    "topic": "entityViewer"
}
```

- **Resonse Status Codes** (HTTP Upgrade Response):
    - `200 OK`: Successfully connected to the WebSocket server.
  - `401 Unauthorized`: Missing or invalid session cookie.

- **Example** (Node.js):

```javascript
import WebSocket from "ws";
import axios from "axios";
import { Agent } from "https";

const TOPICS = {
    ENTITY_VIEWER: "entityViewer",
    SUBSCRIBE: "subscribe",
};

async function getSessionId() {
    try {
        const response = await axios.post(
            "https://10.10.31.192:3000/api/login",
            { username: "admin", password: "admin" },
            { httpsAgent: new Agent({ rejectUnauthorized: false }) }
        );

        const setCookieHeader = response.headers["set-cookie"];
        const sessionCookie = setCookieHeader?.find((c) =>
            c.startsWith("sessionId=")
        );
        if (!sessionCookie) throw new Error("Session cookie not found");

        return sessionCookie.split(";")[0].split("=")[1];
    } catch (error) {
        console.error("Failed to obtain session ID:", error);
        process.exit(1);
    }
}

async function connectWebSocket() {
    const sessionId = await getSessionId();

    const ws = new WebSocket("wss://10.10.31.192:8443", {
        headers: { Cookie: `sessionId=${sessionId}` },
        rejectUnauthorized: false,
    });

    ws.on("open", () => {
        console.log("Connected to WebSocket server");
        ws.send(
            JSON.stringify({
                type: TOPICS.SUBSCRIBE,
                topic: TOPICS.ENTITY_VIEWER,
            })
        );
        console.log("Subscribed to entityViewer topic");
    });

    ws.on("message", (data) => {
        try {
            const json = JSON.parse(data.toString());
            if (!json.data || !json.topic)
                throw new Error("Invalid JSON structure");
            console.log(JSON.stringify(json, null, 2));
        } catch (error) {
            console.error("Error processing message:", error);
        }
    });

    ws.on("error", (error) => console.error("WebSocket error:", error));
    ws.on("close", (code, reason) =>
        console.log(`WebSocket closed: ${code} ${reason}`)
    );
}

connectWebSocket();
```

### 2.6. Send Commands

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/command`
-   **Method**: `POST`
-   **Content-Type**: `application/json`
-   **Description**: Sends a command to a specific register from a **Master Equipment** in ES200.
-   **Response Status Codes**:
    -   `200 OK`: Command sent successfully.
    -   `400 Bad Request`: Malformed request body or invalid point type.
    -   `401 Unauthorized`: Invalid session cookie.
-   **Valid Point Types**:
    -   `Binary Input`
    -   `Binary Output`
    -   `Analog Input`
    -   `Analog Output`
    -   `Double Input`
    -   `Double Output`
-   **Retrieving the ID of an equipment in ES200**:
    To retrieve the equipmentId, you need to create a configuration and then access the `.epgd` file, which is an SQLite3 database. Use an appropriate [SQLite3 viewer](https://sqlitebrowser.org/) to open the file, select the equipments table, and retrieve the ID field from there.
-   **Request Body**:

```json
{
    "equipmentId": 1, // number: the identifier for the equipment
    "idDown": 150, // number: the register address
    "pointType": "Analog Input", // string: type of point being commanded, must be one of the predefined types
    "value": "22.5" // string: value to be set
}
```

-   **Response Body**:

```json
{
    "equipmentId": 1, // number: the identifier for the equipment
    "idDown": 150, // number: the register address
    "pointType": "Analog Input", // string: type of point being commanded, must be one of the predefined types
    "value": "22.5" // string: value to be set
}
```

Modificari 
-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request
curl -k -b cookies.txt -X POST https://localhost:3000/api/command \
    -H "Content-Type: application/json" \
    -d '{"equipmentId": 123, "idDown": 456, "pointType": "Analog Input", "value": "22.5"}'
```

### 2.7. Download Database

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/database`
-   **Method**: `GET`
-   **Description**: Downloads the current ES200 configuration database (`.epgd` file) as a binary file.
-   **Response Status Codes**:
    -   `200 OK`: Successfully downloaded the database file.
    -   `204 No Content`: The current database is empty.
    -   `401 Unauthorized`: Invalid session cookie.
    -   `500 Internal Server Error`: Failed to retrieve the database.
-   **Response Headers**:
    -   `Content-Type`: `application/x-sqlite3`
    -   `Content-Disposition`: `attachment; filename="ES200.epgd"`
-   **Response Body**: Binary content of the SQLite3 database file.
-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request - Download database and save to file
curl -k -b cookies.txt -X GET https://localhost:3000/api/database \
    -o ES200.epgd
```

-   **Notes**:
    -   The downloaded file is an SQLite3 database that can be opened with any SQLite3 viewer such as [DB Browser for SQLite](https://sqlitebrowser.org/).
    -   The database contains the ES200 configuration including equipments, points, and other settings.

### 2.8. Upload Database

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/database`
-   **Method**: `POST`
-   **Content-Type**: `application/octet-stream` or `application/x-sqlite3`
-   **Description**: Uploads a new ES200 configuration database (`.epgd` file) to replace the current configuration. After a successful upload, all ES200 processes will be restarted to apply the new configuration.
-   **Request Body**: Binary content of the SQLite3 database file.
-   **Response Status Codes**:
    -   `200 OK`: Database uploaded successfully. All processes will be restarted.
    -   `401 Unauthorized`: Invalid session cookie.
    -   `415 Unsupported Media Type`: Invalid or missing Content-Type header.
    -   `500 Internal Server Error`: Failed to upload the database.
-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request - Upload database from file
curl -k -b cookies.txt -X POST https://localhost:3000/api/database \
    -H "Content-Type: application/octet-stream" \
    --data-binary @ES200.epgd
```

-   **Notes**:
    -   The uploaded file must be a valid SQLite3 database in the ES200 `.epgd` format.
    -   After a successful upload, the Watchdog process will restart all ES200 processes to apply the new configuration.
    -   **Warning**: Uploading an invalid or corrupted database may cause ES200 to malfunction. Always ensure you have a backup of the current configuration before uploading a new one.

### 2.9. Update Certificates

-   **Base URL**: `https://localhost:3000`
-   **Endpoint**: `/api/certificates`
-   **Method**: `POST`
-   **Content-Type**: `multipart/form-data`
-   **Description**: Updates the SSL/TLS certificates used by ES200 for secure communications. The endpoint accepts a certificate file and a private key file in PEM format, validates them, and forwards the update to the authentication server.
-   **Request Body** (multipart/form-data):
    -   `cert` (required, file): The SSL certificate file in PEM format (must contain `BEGIN CERTIFICATE` marker).
    -   `key` (required, file): The private key file in PEM format (must contain `BEGIN` marker).
-   **Response Status Codes**:
    -   `200 OK`: Certificates updated successfully.
    -   `400 Bad Request`: Missing files, empty file contents, invalid PEM markers, or certificate/key validation failed.
    -   `401 Unauthorized`: Invalid session cookie.
-   **Response Body** (on error): Plain text error message describing the failure reason.
-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'

# Request - Upload certificate and private key
curl -k -b cookies.txt -X POST https://localhost:3000/api/certificates \
    -F "cert=@certificate.pem" \
    -F "key=@private_key.pem"
```

-   **Notes**:
    -   Both the certificate and private key must be in PEM format.
    -   The certificate and private key must match (the private key must correspond to the public key in the certificate).
    -   The endpoint validates the certificate and key before applying them.
    -   Common error messages include:
        -   `missing files`: One or both form fields are missing.
        -   `certificate file missing` / `private key file missing`: The file field exists but has no filename.
        -   `empty file contents`: One or both files are empty.
        -   `invalid pem markers`: The files don't contain valid PEM headers.
        -   `validation failed: mismatch`: The certificate and private key don't match.
        -   `validation failed: parse cert` / `validation failed: parse key`: Unable to parse the certificate or key.
