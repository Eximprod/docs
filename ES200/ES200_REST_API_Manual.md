# ES200 - REST API Guide <!-- omit from toc -->

[**Table of Contents:**](#toc)

- [1. ES200 Endpoints](#1-es200-endpoints)
  - [1.1. Authenticate User](#11-authenticate-user)
  - [1.2. Update User Password](#12-update-user-password)
- [2. WebServer Endpoints](#2-webserver-endpoints)
  - [2.1. General](#21-general)
  - [2.2. Login](#22-login)
  - [2.3. Authorize](#23-authorize)
  - [2.4. Logout](#24-logout)
  - [2.5. Fetch Log Files Names](#25-fetch-log-files-names)
  - [2.6. Fetch Log Content](#26-fetch-log-content)
  - [2.7. Get Master Equipments](#27-get-master-equipments)
  - [2.8. Send Commands](#28-send-commands)
  - [2.9. Download Database](#29-download-database)
  - [2.10. Upload Database](#210-upload-database)
  - [2.11. Update Device Certificates](#211-update-device-certificates)
  - [2.12. Update Equipment Certificates](#212-update-equipment-certificates)
  - [2.13. Get Certificates Status](#213-get-certificates-status)
  - [2.14. Delete Equipment Certificates](#214-delete-equipment-certificates)
- [3. WebSocket API](#3-websocket-api)
  - [3.1. Connection](#31-connection)
  - [3.2. Message Format](#32-message-format)
  - [3.3. Subscribing and Unsubscribing](#33-subscribing-and-unsubscribing)
  - [3.4. Subscription Topics](#34-subscription-topics)
    - [3.4.1. entityViewer](#341-entityviewer)
    - [3.4.2. esTimestamp](#342-estimestamp)
    - [3.4.3. esConnectionStatus](#343-esconnectionstatus)
    - [3.4.4. esVersion](#344-esversion)
  - [3.5. Request Messages](#35-request-messages)
    - [3.5.1. command](#351-command)
    - [3.5.2. updatePassword](#352-updatepassword)
  - [3.6. Example](#36-example)

## 1. ES200 Endpoints

These endpoints are served by the ESRemote process on port `1732`. The WebServer uses them internally to verify credentials; they are listed here because the SEED they return is also needed for ESRemote connections.

### 1.1. Authenticate User

-   **Base URL**: `https://localhost:1732`
-   **Endpoint**: `/api/authentication`
-   **Method**: `POST`
-   **Content-Type**: `application/json`
-   **Description**: Authenticates a user to get the SEED value used for establishing ESRemote connections.
-   **Request Body**: `password` is the uppercase hexadecimal SHA-256 of the plain text password.

```json
{
    "username": "<username>", // string
    "password": "<UPPERCASE_SHA256_OF_PASSWORD>" // uppercase SHA-256 string
}
```

-   **Response status codes**:

    -   `200 OK`: Returns the SEED value.
    -   `400 Bad Request`: Malformed request.
    -   `401 Unauthorized`: Wrong credentials.
    -   `411 Length Required`: Empty body (no `Content-Length` header).

-   **Response body** (`text/plain`):

```
<64 uppercase hexadecimal characters>
```

-   **Example**:

```bash
curl -k -X POST https://localhost:1732/api/authentication \
	-H "Content-Type: application/json" \
	-d '{"username": "<username>", "password": "<UPPERCASE_SHA256_OF_PASSWORD>"}'
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
    "username": "<username>", // string
    "currentPassword": "<UPPERCASE_SHA256_OF_PASSWORD>", // uppercase SHA-256 string
    "newPassword": "<UPPERCASE_SHA256_OF_NEW_PASSWORD>" // uppercase SHA-256 string
}
```

-   **Response status codes**:
    -   `200 OK`: Password updated successfully. The body is the SEED value, as for `/api/authentication`.
    -   `400 Bad Request`: Malformed request.
    -   `401 Unauthorized`: Incorrect current password.
    -   `411 Length Required`: Empty body (no `Content-Length` header).
-   **Example**:

```bash
curl -k -X POST https://localhost:1732/api/updateUserPassword \
      -H "Content-Type: application/json" \
      -d '{"username": "<username>", "currentPassword": "<UPPERCASE_SHA256_OF_PASSWORD>", "newPassword": "<UPPERCASE_SHA256_OF_NEW_PASSWORD>"}'
```

## 2. WebServer Endpoints

### 2.1. General

-   **Base URL**: `https://<device>:3000`. The server listens on all IPv4 and IPv6 interfaces over HTTPS, using the device TLS certificate.
-   **Authentication**: every endpoint except `/api/login` and `/api/logout` requires the `sessionId` cookie returned by `/api/login`. A missing, unknown or expired cookie gives `401 Unauthorized` with an empty body.
-   **Session lifetime**: 1800 seconds (30 minutes) from login. The lifetime is fixed, it is not extended by activity.
-   **Request body limit**: 5 MB. Larger bodies are rejected with `413 Payload Too Large`. This applies to database and certificate uploads.
-   **Cookie handling with `curl`**: store the cookie at login with `-c cookies.txt` and send it on later requests with `-b cookies.txt`. Every example below follows this pattern.

### 2.2. Login

-   **Endpoint**: `/api/login`
-   **Method**: `POST`
-   **Content-Type**: `application/json`
-   **Description**: Attempts to log in a client, returning the session cookie upon success. Credentials are verified against the ESRemote authentication server (section 1.1); the WebServer hashes the password before forwarding it.
-   **Request Body**:

```json
{
    "username": "user", // string
    "password": "password" // string (plain text password)
}
```

-   **Response Status Codes**:
    -   `200 OK`: Successful login, returns session cookie.
    -   `400 Bad Request`: Malformed body, or `username` / `password` missing.
    -   `401 Unauthorized`: Incorrect credentials.
    -   `429 Too Many Requests`: The client IP is locked out after too many failed login attempts.
    -   `500 Internal Server Error`: The ESRemote authentication server could not be reached, or the session could not be created.
-   **Response Body**:

```json
{
    "authenticatedUser": "user" // string
}
```

-   **Response Headers**:
    -   `Set-Cookie`: `sessionId=<32 hex characters>; Max-Age=1800; Path=/; HttpOnly; Secure; SameSite=Strict;`
-   **Example**:

```bash
# Making a login request to get the sessionId cookie
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
     -H "Content-Type: application/json" \
     -d '{"username": "<username>", "password": "<password>"}'

# Making a request that includes the sessionId cookie
curl -k -b cookies.txt -X GET https://localhost:3000/api/logs
```

-   **Notes**:
    -   **Lockout**: repeated failed logins from the same address are rejected with `429` for a while, even with correct credentials. A successful login clears the record.
    -   The cookie is `HttpOnly` and `Secure`, so it is only sent over HTTPS and is not readable from browser scripts.

### 2.3. Authorize

-   **Endpoint**: `/api/authorize`
-   **Method**: `GET`
-   **Description**: Checks whether the session cookie is still valid and returns the user it belongs to. Useful to restore a session without logging in again.
-   **Response Status Codes**:
    -   `200 OK`: The session is valid.
    -   `401 Unauthorized`: Missing, unknown or expired session cookie.
-   **Response Body**:

```json
{
    "authenticatedUser": "<username>" // string
}
```

-   **Example**:

```bash
curl -k -b cookies.txt -X GET https://localhost:3000/api/authorize
```

### 2.4. Logout

-   **Endpoint**: `/api/logout`
-   **Method**: `POST`
-   **Description**: Logs out the client and invalidates the session cookie (if present). The request must carry a body or a `Content-Length` header; an empty body is fine.
-   **Response Status Codes**:
    -   `200 OK`: Successful logout. Returned even without a session cookie.
    -   `400 Bad Request`: The request has neither a body nor a `Content-Length` header. The session stays valid.
-   **Example**:

```bash
curl -k -b cookies.txt -X POST https://localhost:3000/api/logout -d ""
```

### 2.5. Fetch Log Files Names

-   **Endpoint**: `/api/logs`
-   **Method**: `GET`
-   **Description**: Retrieves the available log files, grouped by category. The three categories are `General`, `Events` and `Commands`. A category whose directory holds no files is still returned, with an empty `files` array; a category whose directory does not exist is omitted.
-   **Response Status Codes**:
    -   `200 OK`: Successfully retrieved log files names.
    -   `401 Unauthorized`: Invalid session cookie.
-   **Response Body**:

```json
{
    "logCategories": [
        {
            "name": "General", // string: General | Events | Commands
            "files": [
                {
                    "name": "MultiDataMaster", // string: display name
                    "fileName": "MultiDataMaster" // string: value for the fileName parameter of /api/logcontent
                }
            ]
        }
    ]
}
```

-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request
curl -k -b cookies.txt -X GET https://localhost:3000/api/logs
```

```json
// Response
{
    "logCategories": [
        {
            "name": "General",
            "files": [
                { "name": "ESRemote", "fileName": "ESRemote" },
                { "name": "MultiDataMaster", "fileName": "MultiDataMaster" },
                { "name": "Watchdog", "fileName": "Watchdog" },
                { "name": "Broker1", "fileName": "3" }
            ]
        },
        {
            "name": "Events",
            "files": [
                { "name": "MultiDataMaster", "fileName": "MultiDataMaster" },
                { "name": "Watchdog", "fileName": "Watchdog" }
            ]
        },
        {
            "name": "Commands",
            "files": [
                { "name": "ESRemote", "fileName": "ESRemote" },
                { "name": "MultiDataMaster", "fileName": "MultiDataMaster" }
            ]
        }
    ]
}
```

-   **Notes**:
    -   Per-equipment log files are stored under their channel ID. For those files `fileName` is the numeric channel ID and `name` is the equipment name (several equipments on the same channel are joined with `_`). Numeric files are listed only when they match the channel of an active equipment whose process is enabled in the configuration.
    -   Backup log files (containing `_backup_` in the name) are not listed.
    -   Always pass `fileName`, not `name`, to `/api/logcontent`.

### 2.6. Fetch Log Content

-   **Endpoint**: `/api/logcontent`
-   **Method**: `GET`
-   **Description**: Fetches and parses the content of a log file, including its rotated backups. The server resolves the file from the logs directory, the category and the file name; clients do not supply a path.
-   **Query Parameters**:

    -   `fileName` (required, string): The `fileName` value returned by `/api/logs`.
    -   `name` (required, string): The `name` value returned by `/api/logs`.
    -   `category` (required, string): `General`, `Events` or `Commands`. Case sensitive.

-   **Response Status Codes**:
    -   `200 OK`: Successfully retrieved log content.
    -   `400 Bad Request`: A parameter is missing or `category` is not one of the three categories.
    -   `401 Unauthorized`: Invalid session cookie.
    -   `500 Internal Server Error`: The log file could not be read (for example an unknown `fileName`).
-   **Response Body**:

```json
{
    "logContent": {
        "headers": ["Log Type", "System Timestamp", "Message"], // string[]: column names for the category
        "body": [
            // one array per parsed line: one cell per header, then the row index
            [
                "General", // Log Type
                {
                    "timestamp": {
                        "day": "13",
                        "month": "12",
                        "year": "2024",
                        "hours": "09",
                        "minutes": "07",
                        "seconds": "08",
                        "milliseconds": "875"
                    }
                }, // System Timestamp
                "Process started", // Message
                0 // row index
            ]
        ]
    }
}
```

-   **Columns per category**:

| Category   | Headers                                                                                                                          |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `General`  | `Log Type`, `System Timestamp`, `Message`                                                                                        |
| `Events`   | `System Timestamp`, `Event Timestamp`, `Equipment`, `PointAddress`, `Type`, `OldValue`, `NewValue`, `OldValidity`, `NewValidity` |
| `Commands` | `System Timestamp`, `Command Timestamp`, `Equipment`, `Slave`, `PointAddress`, `Type`, `Value`                                  |

-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request
curl -k -b cookies.txt -X GET "https://localhost:3000/api/logcontent?fileName=ESRemote&name=ESRemote&category=Commands"
```

-   **Notes**:
    -   Every `Timestamp` column is returned as the object shown above. `milliseconds` is `"000"` when the log line has no millisecond part.
    -   The `Log Type` cell is one of `Other`, `Error`, `PointChange`, `General`, `Command`, `Database`, `Protocol`, `Test`.
    -   Lines that do not match the category's log format are skipped, so the row index counts parsed rows only.
    -   The query parameters must be URL-encoded when they contain special characters.

### 2.7. Get Master Equipments

-   **Endpoint**: `/api/points`
-   **Method**: `GET`
-   **Description**: Retrieves the list of **Master Equipments** and their associated points. Supports filtering by `equipmentId`, `idDown`, and `pointType`. For live updates use the `entityViewer` WebSocket topic (section 3.4.1).
-   **Query Parameters**:

    -   `equipmentId` (optional, number): The ID of the Master Equipment to filter points by. Must be greater than 0.
    -   `idDown` (optional, number): The register address to filter points (registers) by. Must be greater than 0.
    -   `pointType` (optional, string): The type of point to filter by. Valid point types are:
        -   `Binary Input`
        -   `Binary Output`
        -   `Analog Input`
        -   `Analog Output`
        -   `Double Input`
        -   `Double Output`

-   **Response Status Codes**:

    -   `200 OK`: Successfully retrieved Master Equipments and points.
    -   `204 No Content`: No Master Equipments are available at all (for example ESRemote is not connected yet). The body is `{"masterEquipments": []}`.
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
    -d '{"username": "<username>", "password": "<password>"}'

# Request
curl -k -b cookies.txt -X GET "https://localhost:3000/api/points"
```

2. Retrieve points for a specific `equipmentId`:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request
curl -k -b cookies.txt -X GET "https://localhost:3000/api/points?equipmentId=1"
```

3. Retrieve a specific point by `idDown` and `pointType`:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request
curl -k -b cookies.txt -X GET "https://localhost:3000/api/points?equipmentId=1&idDown=10020&pointType=Binary%20Input"
```

-   **Notes**:
    -   `pointType` must be spelled exactly as listed above, with a single space; a value with different spacing passes validation but matches no point.
    -   An `equipmentId` that matches no equipment gives `200` with an empty `masterEquipments` array. `idDown` and `pointType` never remove an equipment: every equipment that passes the `equipmentId` filter is returned, with only the matching points in `points`, which may be empty.
    -   If no query parameters are provided, all available `Master Equipments` and their `points` will be returned.
    -   The query parameters should be correctly URL-encoded. For example, if the `pointType` is `Binary Input`, it should be encoded as `Binary%20Input`.

### 2.8. Send Commands

-   **Endpoint**: `/api/command`
-   **Method**: `POST`
-   **Content-Type**: `application/json`
-   **Description**: Sends a command to a specific register from a **Master Equipment** in ES200.
-   **Response Status Codes**:
    -   `200 OK`: Command accepted and forwarded to ES200.
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
    Use `/api/points` (section 2.7): the `id` field of each entry in `masterEquipments` is the `equipmentId`. The same value is the `ID` column of the `Equipments` table in the configuration database (`.epgd`, an SQLite3 file).
-   **Request Body**:

```json
{
    "equipmentId": 1, // number: the identifier for the equipment
    "idDown": 150, // number: the register address
    "pointType": "Analog Output", // string: type of point being commanded, must be one of the predefined types
    "value": "22.5" // string: value to be set
}
```

-   **Response Body**: the accepted command, echoed back.

```json
{
    "equipmentId": 1, // number
    "idDown": 150, // number
    "pointType": "Analog Output", // string
    "value": "22.5" // string
}
```

-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request
curl -k -b cookies.txt -X POST https://localhost:3000/api/command \
    -H "Content-Type: application/json" \
    -d '{"equipmentId": 1, "idDown": 150, "pointType": "Analog Output", "value": "22.5"}'
```

-   **Notes**:
    -   `200 OK` means the command was forwarded to ESRemote. The server does not check that the equipment or register exists. The outcome is visible in the point's `status` and `value` returned by `/api/points`, and in the `Commands` log.

### 2.9. Download Database

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
    -d '{"username": "<username>", "password": "<password>"}'

# Request - Download database and save to file
curl -k -b cookies.txt -X GET https://localhost:3000/api/database \
    -o ES200.epgd
```

-   **Notes**:
    -   The downloaded file is an SQLite3 database that can be opened with any SQLite3 viewer such as [DB Browser for SQLite](https://sqlitebrowser.org/).
    -   The database contains the ES200 configuration including equipments, points, and other settings.

### 2.10. Upload Database

-   **Endpoint**: `/api/database`
-   **Method**: `POST`
-   **Content-Type**: `application/octet-stream` or `application/x-sqlite3`
-   **Description**: Uploads a new ES200 configuration database (`.epgd` file) to replace the current configuration. After a successful upload, all ES200 processes will be restarted to apply the new configuration.
-   **Request Body**: Binary content of the SQLite3 database file.
-   **Response Status Codes**:
    -   `200 OK`: Database uploaded successfully. All processes will be restarted.
    -   `401 Unauthorized`: Invalid session cookie.
    -   `413 Payload Too Large`: The file is larger than 5 MB.
    -   `415 Unsupported Media Type`: Invalid or missing Content-Type header.
    -   `500 Internal Server Error`: Empty body, or the database could not be stored.
-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request - Upload database from file
curl -k -b cookies.txt -X POST https://localhost:3000/api/database \
    -H "Content-Type: application/octet-stream" \
    --data-binary @ES200.epgd
```

-   **Notes**:
    -   The uploaded file must be a valid SQLite3 database in the ES200 `.epgd` format.
    -   After a successful upload, the Watchdog process will restart all ES200 processes to apply the new configuration. The WebServer is among them, so the session is lost and the client has to log in again.
    -   **Warning**: Uploading an invalid or corrupted database may cause ES200 to malfunction. Always ensure you have a backup of the current configuration before uploading a new one.

### 2.11. Update Device Certificates

-   **Endpoint**: `/api/certificates`
-   **Method**: `POST`
-   **Content-Type**: `multipart/form-data`
-   **Description**: Replaces the TLS certificate and private key of the device itself. These are the files served by the WebServer (ports 3000 and 8443) and the ESRemote authentication server (port 1732). The files are validated, written to the certificates directory, and the WebServer and ESRemote are restarted so they load the new files.
-   **Request Body** (multipart/form-data):
    -   `cert` (required, file): The certificate in PEM format (must contain a `BEGIN CERTIFICATE` marker).
    -   `key` (required, file): The unencrypted private key in PEM format (must contain a `BEGIN` marker).
    -   `ca` (optional, file): The CA certificate in PEM format.
-   **Response Status Codes**:
    -   `200 OK`: Certificates written. A restart of the WebServer and ESRemote has been requested.
    -   `400 Bad Request`: Missing files, empty file contents, invalid PEM markers, or certificate/key validation failed.
    -   `401 Unauthorized`: Invalid session cookie.
    -   `413 Payload Too Large`: The request is larger than 5 MB.
    -   `500 Internal Server Error`: The certificates directory is missing, or the files could not be written.
-   **Response Body** (on error): Plain text error message describing the failure reason.
-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request - Upload certificate and private key
curl -k -b cookies.txt -X POST https://localhost:3000/api/certificates \
    -F "cert=@certificate.pem" \
    -F "key=@private_key.pem" \
    -F "ca=@ca_certificate.pem"
```

-   **Notes**:
    -   Both the certificate and private key must be in PEM format, and the private key must not be password protected.
    -   The certificate and private key must match (the private key must correspond to the public key in the certificate).
    -   The response is sent before the restart happens, but the restart follows within seconds. Clients should expect the connection to drop shortly after `200 OK`, the session to be lost, and the new certificate to be served on reconnect.
    -   Error messages:
        -   `missing files`: The `cert` or `key` part is absent or is not a file upload.
        -   `empty file contents`: One or more files are empty.
        -   `invalid pem markers`: The files don't contain valid PEM headers.
        -   `validation failed: mismatch`: The certificate and private key don't match.
        -   `validation failed: parse cert` / `validation failed: parse key`: Unable to parse the certificate or key.
        -   `validation failed: the private key is encrypted; upload an unencrypted PEM key`.
        -   `cert directory missing` / `write failed: <reason>` (status `500`).

### 2.12. Update Equipment Certificates

-   **Endpoint**: `/api/certificates`
-   **Method**: `POST`
-   **Content-Type**: `multipart/form-data`
-   **Description**: Uploads the TLS material of one equipment that uses a secure protocol (for example an MQTT broker connection). The request is routed here instead of section 2.11 whenever the form carries a `process` or `equipment` field. Only the certificate slots that the configuration marks as `runtime` (upload at runtime) accept files; slots configured as `embedded` are owned by the configuration and reject uploads. Use `/api/certificates/status` (section 2.13) to see which slots an equipment has.
-   **Request Body** (multipart/form-data):
    -   `process` (required, field): The process the equipment belongs to, for example `MQTTMaster`. Only processes with TLS support are accepted (currently `MQTTSlave`, `MQTTMaster` and `IEC61850E2`); the `400` message lists them.
    -   `equipment` (required, field): The equipment name as configured. Letters, digits, `,`, `_` and `$` are allowed.
    -   `cert`, `key`, `ca` (files): Any subset of the equipment's slots. `cert` and `key` must be uploaded together.
-   **Response Status Codes**:
    -   `200 OK`: Files written.
    -   `400 Bad Request`: Unknown process or equipment, equipment does not use certificates, no parts uploaded, a part that does not match a slot or matches an `embedded` / `notConfigured` slot, `cert` without `key` or vice versa, empty or unparsable parts, or a certificate/key mismatch.
    -   `401 Unauthorized`: Invalid session cookie.
    -   `413 Payload Too Large`: The request is larger than 5 MB.
    -   `500 Internal Server Error`: The files could not be written.
-   **Response Body** (`200`):

```json
{
    "written": ["cert", "key"], // string[]: the parts that were stored
    "restart": "requested" // string: requested | skippedInactiveEquipment | processNotRunning
}
```

-   **Response Body** (on error): Plain text error message describing the failure reason.
-   **Example**:

```bash
# Authentication
curl -k -c cookies.txt -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "<username>", "password": "<password>"}'

# Request - Upload the client certificate and key of one equipment
curl -k -b cookies.txt -X POST https://localhost:3000/api/certificates \
    -F "process=MQTTMaster" \
    -F "equipment=Broker1" \
    -F "cert=@broker1_client.pem" \
    -F "key=@broker1_client.key"
```

-   **Notes**:
    -   The files are stored as `<part>_<process>_<equipment>.pem` (`.crt` for `ca`) in the certificates directory, for example `cert_MQTTMaster_Broker1.pem`.
    -   `restart` tells the client whether a reconnect is coming:
        -   `requested`: The equipment is active and a restart of its process was requested.
        -   `skippedInactiveEquipment`: The equipment is inactive in the configuration. The files are stored and used when it is activated.
        -   `processNotRunning`: The files are stored but the process is not running, so nothing was restarted.
    -   The same PEM checks as for the device certificates apply to each part: the key must be unencrypted, `cert` and `key` must match.

### 2.13. Get Certificates Status

-   **Endpoint**: `/api/certificates/status`
-   **Method**: `GET`
-   **Description**: Reports which certificate files are present on the device and, for every equipment with TLS enabled in the configuration, the state of each of its certificate slots.
-   **Response Status Codes**:
    -   `200 OK`
    -   `401 Unauthorized`: Invalid session cookie.
-   **Response Body**:

```json
{
    "device": {
        "cert": true, // boolean: device certificate file present
        "key": true, // boolean: device private key present
        "ca": false // boolean: device CA file present
    },
    "equipments": [
        {
            "process": "MQTTMaster", // string
            "equipment": "Broker1", // string
            "active": true, // boolean: equipment active in the configuration
            "state": "waiting", // string: ready | inactive | waiting
            "slots": [
                {
                    "slot": "cert", // string: part name used in the upload form
                    "property": "ClientCertificate", // string: the configuration property this slot fills
                    "source": "runtime", // string: notConfigured | embedded | runtime
                    "present": false // boolean: the file exists on the device
                }
                // ... more slots
            ]
        }
        // ... more equipments
    ]
}
```

-   **Example**:

```bash
curl -k -b cookies.txt -X GET https://localhost:3000/api/certificates/status
```

-   **Notes**:
    -   `state` is `inactive` for an inactive equipment regardless of its files, `waiting` for an active equipment with at least one slot whose file is missing, and `ready` otherwise.

### 2.14. Delete Equipment Certificates

-   **Endpoint**: `/api/certificates`
-   **Method**: `DELETE`
-   **Description**: Removes the certificate files previously uploaded for one equipment. No restart is requested; a running process keeps using the material it has loaded until its next reconnect.
-   **Query Parameters**:
    -   `process` (required, string): The process the equipment belongs to.
    -   `equipment` (required, string): The equipment name.
-   **Response Status Codes**:
    -   `200 OK`: Deletion done. `removed` lists the file names that existed and were deleted; it is empty when there was nothing to delete.
    -   `400 Bad Request`: A parameter is missing, or the process/equipment is unknown.
    -   `401 Unauthorized`: Invalid session cookie.
    -   `500 Internal Server Error`: A file could not be deleted.
-   **Response Body**:

```json
{
    "removed": ["cert_MQTTMaster_Broker1.pem", "key_MQTTMaster_Broker1.pem"] // string[]
}
```

-   **Example**:

```bash
curl -k -b cookies.txt -X DELETE "https://localhost:3000/api/certificates?process=MQTTMaster&equipment=Broker1"
```

## 3. WebSocket API

The WebSocket server is what the WebServer's own user interface uses for live data. It is documented here so that clients that already hold a session can use the same channel.

### 3.1. Connection

-   **URL**: `wss://<device>:8443`. Any path is accepted. The server listens on all IPv4 and IPv6 interfaces and uses the device TLS certificate.
-   **Authentication**: the `sessionId` cookie from `/api/login` must be sent in the `Cookie` header of the handshake request. With a missing or invalid cookie the handshake is refused: no upgrade takes place, the body is `Unauthorized`, a `Set-Cookie` header clears the cookie, and the connection is closed. Clients must treat any response other than `101` as a rejection.
-   **Handshake response**: `101 Switching Protocols` on success.
-   **Idle timeout**: 120 seconds. The server sends pings automatically, so an idle client stays connected as long as it answers them (browsers and most libraries do this by default).
-   **Limits**: at most 16 MB per message. Text and binary frames are both parsed as JSON.
-   **Data source**: the server polls ESRemote once per second. Every push described below happens at that cadence at most.

### 3.2. Message Format

Every message in both directions is a JSON object.

-   **Client to server**: a `type` field selects the request. The remaining fields depend on the type.

```json
{ "type": "subscribe", "topic": "entityViewer" }
```

-   **Server to client**: a `topic` field names the message and `data` carries the payload.

```json
{ "topic": "esTimestamp", "data": { "esTimestamp": "09:07:08 13/12/2024" } }
```

-   Messages that are not valid JSON, lack `type`, or use an unknown `type` are logged by the server and ignored. No error is sent back.

### 3.3. Subscribing and Unsubscribing

```json
{ "type": "subscribe", "topic": "entityViewer" }
```

```json
{ "type": "unsubscribe", "topic": "entityViewer" }
```

-   A client receives pushed messages only for topics it has subscribed to. Subscribing twice to the same topic, or unsubscribing from a topic not subscribed, has no effect.
-   Subscribing to `entityViewer` or `esVersion` immediately sends the current data as a first message (if the server has any yet).

### 3.4. Subscription Topics

#### 3.4.1. entityViewer

The Master Equipments and their points, the same structure as `/api/points` (section 2.7) returns in `masterEquipments`. Sent on subscribe and then whenever any value changes.

```json
{
    "topic": "entityViewer",
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
    ]
}
```

#### 3.4.2. esTimestamp

The ES200 system time, pushed once per second.

```json
{
    "topic": "esTimestamp",
    "data": {
        "esTimestamp": "09:07:08 13/12/2024" // string: HH:MM:SS DD/MM/YYYY, or "n/a"
    }
}
```

#### 3.4.3. esConnectionStatus

Whether the WebServer is connected to ESRemote, pushed once per second. When `connected` is `false` no other data topic is updated.

```json
{
    "topic": "esConnectionStatus",
    "data": {
        "connected": true // boolean
    }
}
```

#### 3.4.4. esVersion

The ES200 software version. Sent once, on subscribe.

```json
{
    "topic": "esVersion",
    "data": "<version>" // string
}
```

### 3.5. Request Messages

These messages are handled once, when received. They do not need a subscription. Where a reply is sent, it goes only to the requesting client.

#### 3.5.1. command

Sends a command to a register, the same as `POST /api/command` (section 2.8). Unlike the REST endpoint, the fields are not validated here: the message is forwarded to ESRemote as is, and a malformed one is dropped there. No reply is sent; the result shows up in `entityViewer`.

```json
{
    "type": "command",
    "equipmentId": 1, // number
    "idDown": 150, // number
    "pointType": "Analog Output", // string: one of the valid point types
    "value": "22.5" // string
}
```

#### 3.5.2. updatePassword

Changes the password of a user. Passwords are plain text; the WebServer hashes them and forwards the request to `/api/updateUserPassword` (section 1.2).

```json
{
    "type": "updatePassword",
    "username": "<username>", // string
    "currentPassword": "<password>", // string
    "newPassword": "test" // string
}
```

Reply:

```json
{
    "topic": "updatePassword",
    "data": {
        "updatePasswordStatus": 200 // number: HTTP status from section 1.2 (200, 400, 401), or 500 if ESRemote could not be reached
    }
}
```

### 3.6. Example

Node.js client that logs in, connects, subscribes to `entityViewer` and `esConnectionStatus`, and prints every message:

```javascript
import WebSocket from "ws";
import axios from "axios";
import { Agent } from "https";

const HOST = "<device>";

async function getSessionId() {
    const response = await axios.post(
        `https://${HOST}:3000/api/login`,
        { username: "<username>", password: "<password>" },
        { httpsAgent: new Agent({ rejectUnauthorized: false }) }
    );

    const sessionCookie = response.headers["set-cookie"]?.find((c) => c.startsWith("sessionId="));
    if (!sessionCookie) {
        throw new Error("Session cookie not found");
    }

    return sessionCookie.split(";")[0].split("=")[1];
}

async function connectWebSocket() {
    const sessionId = await getSessionId();

    const ws = new WebSocket(`wss://${HOST}:8443`, {
        headers: { Cookie: `sessionId=${sessionId}` },
        rejectUnauthorized: false,
    });

    ws.on("open", () => {
        ws.send(JSON.stringify({ type: "subscribe", topic: "entityViewer" }));
        ws.send(JSON.stringify({ type: "subscribe", topic: "esConnectionStatus" }));
    });

    ws.on("message", (data) => {
        const json = JSON.parse(data.toString());
        console.log(json.topic, JSON.stringify(json.data, null, 2));
    });

    ws.on("error", (error) => console.error("WebSocket error:", error));
    ws.on("close", (code, reason) => console.log(`WebSocket closed: ${code} ${reason}`));
}

connectWebSocket();
```
