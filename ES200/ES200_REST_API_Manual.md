
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


## 1. ES200 Endpoints

### 1.1. Authenticate User
 - **Base URL**: `https://localhost:1732`
 - **Endpoint**: `/api/authentication`
 - **Method**: `POST`
 - **Description**: Authenticates a user to get the SEED value used for establishing ESRemote connections.
 - **Request Body**:
```json
{
	"username": "admin", // string
	"password": "8C6976E5B5410415BDE908BD4DEE15DFB167A9C873FC4BB8A81F6F2AB448A918" // uppercase SHA-256 string
}
```
 - **Response status codes**:
	- `200 OK`: Returns SEED value.
	- `400 Bad Request`: Malformed request.
	- `401 Unauthorized`: Wrong credentials.
	- `411 Length Required`: Empty body.
  
 - **Response body**:
```json
5ADC69F6C5AADF8220850C627E367293ECE4F2C1B469BD0D519E101AEB01AB29 // 64 characters Base64 string
```
 - **Example**:
```bash
curl -k -X POST https://localhost:1732/api/authentication \
	-H "Content-Type: application/json" \
	-d '{"username": "admin", "password": "8C6976E5B5410415BDE908BD4DEE15DFB167A9C873FC4BB8A81F6F2AB448A918"}'
```

### 1.2. Update User Password
-   **Base URL**: `https://localhost:1732`
-   **Endpoint**: `/api/updateUserPassword`
-   **Method**: `POST`
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
 - **Base URL**: `https://localhost:1732`
 - **Endpoint**: `/api/getUsers`
 - **Method**: `GET`
 - **Description**: Retrieves a list of all users.
 - **Response status codes**:
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
 -   **Description**: Attempts to log in a client, returning the session cookie upon success.
 -   **Request Body**:
```json
{
	"username": "user", // string
	"password": "password" // string (plain text password)
}
```
 - **Response Status Codes**:
	-   `200 OK`: Successful login, returns session cookie.
	-   `429 Too Many Requests`: Too many failed login attempts.
	-   `401 Unauthorized`: Incorrect credentials.
	-   `500 Internal Server Error`: Server error occurred.
 - **Response Body**:
```json
{
	"authenticatedUser": "user" // string
}
```
 - **Example**:
```bash
curl -k -X POST https://localhost:3000/api/login \
    -H "Content-Type: application/json" \
    -d '{"username": "admin", "password": "admin"}'
```
- **Note**:
	- When you use `curl` to make a login request that sets an HTTP-only cookie (containing the session ID) upon successful login, this cookie is indeed managed by `curl` for the duration of the session. However, it's important to note that `curl` does not automatically include this cookie in subsequent requests unless explicitly instructed to do so.
	- To ensure the cookie set by the `/api/login` endpoint is included in subsequent requests, such as `/api/logs`, you need to use `curl`'s cookie handling capabilities:
		- Store the Cookie: Make the login request and save the cookie to a file. This can be done using the `-c` or `--cookie-jar` option in `curl`.
		- Reuse the Cookie: In subsequent requests, you need to include this cookie for the server to recognize the session. This is done using the `-b` or `--cookie` option in `curl`.
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
- **Example**:
```bash
# Request
curl -k -X GET https://localhost:3000/api/logs
```
```json
// Response
{
   "logCategories":[
      {
	     "id":1,
         "name":"General",
         "files":[
            {
               "id":2,
               "name":"ESRemote",
               "path":"/mnt/ramdisk/ESRemote"
            },
            {
               "id":3,
               "name":"MultiDataMaster",
               "path":"/mnt/ramdisk/MultiDataMaster"
            },
            {
               "id":4,
               "name":"Watchdog",
               "path":"/mnt/ramdisk/Watchdog"
            }
         ]
      },
      {
	     "id":5,
         "name":"Events",
         "files":[
            {
               "id":6,
               "name":"MultiDataMaster",
               "path":"/mnt/ramdisk/Events/MultiDataMaster"
            },
            {
               "id":7,
               "name":"Watchdog",
               "path":"/mnt/ramdisk/Events/Watchdog"
            }
         ]
      },
      {
	     "id":8,
         "name":"Commands",
         "files":[
            {
               "id":9,
               "name":"ESRemote",
               "path":"/mnt/ramdisk/Commands/ESRemote"
            },
            {
               "id":10,
               "name":"MultiDataMaster",
               "path":"/mnt/ramdisk/Commands/MultiDataMaster"
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
-   **Query Parameters**: `path`, `name`, `category` for the log file.
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
- **Example**:
```bash
# The query parameters must be correctly URL-encoded
curl -k -X GET https://localhost:3000/api/logcontent?name=ESRemote&path=%2Fmnt%2Framdisk%2FCommands%2FESRemote&category=Commands
```
