Introduction
========

The umbrella term *Security* in Signal K server refers to the difference between running an *unsecured server*, that everybody who has network access to it can access and alter at will, and a *secured server* that has one or more security restrictions enabled.

The security options are related to
* **authentication**: the users are authenticated, for example but not limited to username & password
* **access control**: based on authentication we can grant / limit access to Signal K data and server configuration
* **encryption**: is the network traffic protected against eavesdropping by using encryption
* **active network services**: which of the server's services/interfaces are configured and active, for example does it allow unsecured read/write of NMEA0183 data

Enabling Security
=======

You can tell that a server does not have security turned on from not having `Login` option at the top right corner of the admin UI.

You can enable security in several ways:
- by accessing the Security settings pages in the admin UI
- starting the server with the `--securityenabled` option
- adding the following section in the settings file

```
"security": {
    "strategy": "./tokensecurity",
  }
```

When security is enabled the admin UI will prompt you to create the admin account in the admin UI.

Security configuration is stored in file called `security.json` which will be located in the configuration directory.

Disabling Security / Lost Admin Credentials
==========

**In case you accidentally lose your admin credentials you can remove `security.json` and restart.**

Access Control
====

Access control lists allow fine grained control of access to specific data in SignalK. The acls are a list which allow specifiying controls for specifc contexts and it goes in the security.json file mentioned above.

The following example defines acls for the self context. It allows anyone to read the paths `"steering.*"`, `"navigation.*"`, `"name"`, `"design.aisShipType"` and allows the admin user permission to write (update) those paths. 

The second entry allows the user _john_ to read any data coming from the `actisense.35` $source.

The last entry covers all other paths, allowing only the admin user to read and no one can write.

If there is no match is found for a specific path in the acl list, then permission will be denied to that path.

```
  "acls": [
    {
      "context": "vessels.self",
      "resources": [
        {
          "paths": ["steering.*", "navigation.*", "name", "design.aisShipType"],
          "permissions": [
            {
              "subject": "any",
              "permission": "read"
            },
            {
              "subject": "admin",
              "permission": "write"
            }
          ]
        },
        {
          "sources": [ "actisense.35" ],
          "permissions": [
            {
              "subject": "john",
              "permission": "read"
            }
          ]
        },
        {
          "paths": ["*"],
          "permissions": [
            {
              "subject": "admin",
              "permission": "read"
            }
          ]
        }
      ]
    }
  ]
  ```

Active network services 
=====

Signal K server's main networks services are
- the primary http / WebSocket interface, with options to use TLS encryption and certificate verification and authentication, with read and write capabilities
- read only NMEA0183 data available for clients over TCP on port 10110
- Signal K over TCP on port 8375, with read and write capabilities
- manually configured TCP and UDP connections

The security implication of these is that with no security options turned on clients that have network access to the server have both read and write access to all of its data.

People often dismiss this by saying that their boat's local network is secure enough. But one very common scenario is connect your SK server as a client to a marina wifi. Many marina wifis allow communication between all connected computers, with your the server advertising its services over MDNS. If there is a manually configured connection for NMEA0183 over UDP other computers broadcasting data can easily write data to the server. 

NMEA0183 connections over TCP and UDP are inherently not safe: there are no options for authentication and secure communication. In comparison Signal K over TLS and HTTP / WebSockets can provide secure, authentication read and write access to your data.

