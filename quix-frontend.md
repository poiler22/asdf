# Quix Client REST API
&nbsp; 
### Authenticate  
-----------------------------
&nbsp;
Client  
```
POST /api/v1/client/auth
Content-Type: application/json
```
```js
{
    "username": "quix123",
    "password": "bright01"
}
```
&nbsp;
Server Response  
```js
{
    "token": "fonsdkosdg9wjttj092gf1rj9fm2fe",
    "valid": 86400
}
```
&nbsp; 
### Get Backends
-----------------------------
Client  
```
GET /api/v1/client/backends
Accept: application/json
Authorization: Bearer <token>
```
Response
```js
{
    "msg": [
        {
            "id": 1,
            "name":     "Bob's Server",
            "uplink": {
                "id": 1,
                "host": "th1.s.quix.click"
            },
            "status":   "ok",
            "location": "BKK"
        },
        {
            "id": 2,
            "name":     "Game Server",
            "uplink": {
                "id": 1,
                "host": "th1.s.quix.click"
            },
            "status":   "ok",
            "location": "BKK"
        }
    ]
}
```
&nbsp; 
### Get Backend Detail
-----------------------------
```
GET /api/v1/client/backends?id=1
Accept: application/json
Authorization: Bearer <token>
```
Response
```js
{
    "name": "Bob's Server",
    "uplink": "th1.s.quix.click",
    "status": "ok",
    "location": "bkk",
    "internal_ip": "10.13.14.25",
    "rules": [
        {
            "ext_ip":   "1.2.3.4",
            "proto":    "TCP",
            "eport":    9999,
            "iport":    4444
        },
        {
            "ext_ip":   "1.2.3.4",
            "proto":    "TCP",
            "eport":    9451,
            "iport":    4321
        }
    ]
}
```
&nbsp; 
### Update Backend Information
-----------------------------
Modifiable Parameters  

| Parameter   | Description             | Operations                    | Type                           | 
| ----------- | ----------------------- | ----------------------------- | ------------------------------ |
| `name`      | Server Friendly Name    | Read, Write                   | string, or null if no name.    |
| `location`  | Server Uplink Location  | Read, Write                   | string, or null to disconnect. |
| `rules`     | Server Forwarding Rules | Create, Read, Write, Update   | dict - see below               |

Rule Values

| Rule        | Description             | Acceptable Values      |
| ----------- | ----------------------- | ---------------------- |
| `ext_ip`    | External IP             | Any IPv4 host address  |
| `proto`     | Protocol                | TCP, UDP               |
| `port`      | External Port           | 0 ~ 65535              |
| `iport`     | Internal Port           | 0 ~ 65535              |

>Note: The `ext_ip` parameter must be a valid IP linked to the relevant frontend.

Action Types
| Action         | Description                           |
| -------------- | ------------------------------------- |
| `modify_kv`    | Modify key/value pairs                |
| `delete_rule`  | Delete existing forwarding rule       |
| `create_rule`  | Create new forwarding rule            |
| `modify_rule`  | Modify an existing forwarding rule    |

&nbsp; 
```
PATCH /api/v1/client/backends/modify?id=1
Accept: application/json
Authorization: Bearer <token>
```
```js
{
    "cmd": [
        {
            "op": "modify_kv",
            "data": {
                "name": "Game Server",
                "location": null
            }
        },
        {
            "op": "delete_rule",
            "data": {
                "ext_ip": "1.2.3.4",
                "proto": "TCP",
                "eport": 9999,
                "iport": 4444
            }
        },
        {
            "op": "create_rule",
            "data": {
                "ext_ip": "1.2.3.4",
                "proto": "TCP",
                "eport": 9999,
                "iport": 8888
            }
        },
        {
            "op": "modify_rule",
            "data": {
                "from": {
                    "ext_ip": "1.2.3.4",
                    "proto": "UDP",
                    "eport": 9999,
                    "iport": 4444
                },
                "to": {
                    "ext_ip": "1.2.3.4",
                    "proto": "UDP",
                    "eport": 9999,
                    "iport": 4445
                }   
            }
        }
    ]
}
```
Response Status Code
| Case                                              | Code |
| ------------------------------------------------- | ---- |
| The request was completed successfully.           | 200  |
| The request failed due to invalid auth.           | 401  |
| The request failed due to invalid details         | 400  |
| The forwarding rule was not found. (modify)       | 404  |
| A rule already exists with those details (create) | 409  |


&nbsp;
### Available Address Listings
-----------------------------
```
GET /api/v1/client/extips?backend=1
Accept: application/json
Authorization: Bearer <token>
```
This should return the public IPs of the frontend the backend is currently attached to.  
Note that `public_ip` in the `Frontend` table refers to the frontend-backend uplink IP and should not be included.  
This means if a single IP is used for the uplink and forwarding, it must be included both in `Frontend.public_ip` and the `Addresses.ip` attributes.
```js
{
    "extips": [
        "1.2.3.4",
        "4.3.2.1",
        "5.6.7.8"
    ]
}
```

&nbsp; 
### Backend Token Generation
-----------------------------
```
GET /api/v1/client/pin?type=token
Content-Type: application/json
Authorization: Bearer <token>
```
The token should not sync to the database, and should automatically be cleared after the timestamp specified.  
The typical validity period should only be 3 minutes.  
```js
{
    "pin":      "1dg9x1fz",
    "valid":    1518828488
}
```
&nbsp;
### Backend Token Retrieval
-----------------------------
```
GET /api/v1/backend/token?pin=1dg9x1fz
Content-Type: application/json
```
If creating a backend token with a limited validity period, `valid` should be the expiry timestamp.
```js
{
    "token": "fonsdkosdg9wjttj092gf1rj9fm2fe",
    "valid": null
}
```
Response Status Code
| Case                                              | Code |
| ------------------------------------------------- | ---- |
| The request was completed successfully.           | 200  |
| The PIN is invalid.                               | 410  |