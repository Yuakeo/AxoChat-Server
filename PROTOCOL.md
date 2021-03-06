# The AxoChat protocol
The AxoChat protocol is based on websockets.
All packets are sent to the `/ws` endpoint.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [The AxoChat protocol](#the-axochat-protocol)
- [Structures](#structures)
    - [Id](#id)
    - [UserInfo](#userinfo)
- [Packets](#packets)
    - [Client](#client)
        - [Error](#error)
        - [Message](#message)
        - [MojangInfo](#mojanginfo)
        - [NewJWT](#newjwt)
        - [PrivateMessage](#privatemessage)
        - [Success](#success)
        - [UserCount](#usercount)
    - [Server](#server)
        - [BanUser](#banuser)
        - [LoginJWT](#loginjwt)
        - [LoginMojang](#loginmojang)
        - [Message](#message-1)
        - [PrivateMessage](#privatemessage-1)
        - [RequestJWT](#requestjwt)
        - [RequestMojangInfo](#requestmojanginfo)
        - [RequestUserCount](#requestusercount)
        - [UnbanUser](#unbanuser)

<!-- markdown-toc end -->

# Packets
Packets are sent in websocket `text` messages encoded as JSON objects.
They all have a structure like that, with `c` being optional:
```json
{
    "m": "Name",
    "c": {
        "...": "...",
        "...": false
    }
}
```

Not every packet has a body:
```json
{
    "m": "Name"
}
```

## Client
Client Packets are received by the client.

### Error
This packet may be sent at any time,
but is usually a response to a failed action of the client.

**Example**
```json
{
    "m": "Error",
    "c": {
        "message": "LoginFailed"
    }
}
```

### Message
This packet will be sent to every authenticated client,
if another client successfully [sent a message](#message-1) to the server.

- `author_info` is just the name and uuid of the user that sent the message.
- `content` is any message fitting the validation scheme of the server.

**Example**
```json
{
    "m": "Message",
    "c": {
        "author_info": {
            "name": "Notch",
            "uuid": "069a79f4-44e9-4726-a5be-fca90e38aaf5"
        },
        "content": "Hello, World!"
    }
}
```

### MojangInfo
After the client sent the server a [RequestMojangInfo](#requestmojanginfo)
packet, the server will provide the client with a `session_hash`.
A session hash is synonymous with a *server id* in the context of
[authentication with mojang](https://wiki.vg/Protocol_Encryption#Authentication).
The client has to send a [LoginMojang](#loginmojang) packet to the server
after authenticating itself with mojang.

**Example**
```json
{
    "m": "MojangInfo",
    "c": {
        "session_hash": "88e16a1019277b15d58faf0541e11910eb756f6"
    }
}
```

### NewJWT
After the client sent the server a [RequestJWT](#requestjwt)
packet, the server will provide the client with json web token.
This token can be used in the [LoginJWT](#loginjwt) packet.

**Example**
```json
{
    "m": "NewJWT",
    "c": {
        "token": "VGhpcyBjb3VsZCBiZSBhIGpzb24gd2ViIHRva2VuLCBidXQgaXQgaXNuJ3QK"
    }
}
```

### PrivateMessage
The content of this packet will be sent to a authenticated client with `allow_messages` turned on,
if another client successfully [sent a private message](#privatemessage-1).

- `author_info` is just the name and uuid of the user that sent the message.
- `content` is any message fitting the validation scheme of the server.

**Example**
```json
{
    "m": "PrivateMessage",
    "c": {
        "author_info": {
            "name": "Notch",
            "uuid": "069a79f4-44e9-4726-a5be-fca90e38aaf5"
        },
        "content": "Hello, User!"
    }
}
```

### Success
This packet is sent after either
[LoginMojang](#loginmojang), [LoginJWT](#loginjwt),
[BanUser](#banuser) or [UnbanUser](#unbanuser)
were processed successfully.

- `reason` is the reason for the success; it is one of the following possible
  values:
  - `Login`
  - `Ban`
  - `Unban`

**Example**
```json
{
    "m": "Success",
    "c": {
        "reason": "Login"
    }
}
```

### UserCount
This packet is sent after [RequestUserCount](#requestusercount) was received.

- `connections` is the amount of connections this server has open
- `logged_in` is the amount of authenticated connections this server has open

**Example**
```json
{
    "m": "UserCount",
    "c": {
        "connections": 623,
        "logged_in": 531,
    }
}
```

## Server
Server Packets are received by the server.

### BanUser
A client can send this packet to ban other users from using this chat.

- `user` is the uuid of the user to ban.

**Example**
```json
{
    "m": "BanUser",
    "c": {
        "user": "069a79f4-44e9-4726-a5be-fca90e38aaf5"
    }
}
```

### LoginJWT
To login using a json web token, the client has to send a `LoginJWT` packet.
it will send [Success](#success) if the login was successful.

- `token` can be retrieved by sending [RequestJWT](#requestjwt) on an already
- authenticated connection.
- If `allow_messages` is true, other clients may send private messages
  to this client.

**Example**
```json
{
    "m": "LoginJWT",
    "c": {
        "token": "VGhpcyBjb3VsZCBiZSBhIGpzb24gd2ViIHRva2VuLCBidXQgaXQgaXNuJ3QK",
        "allow_messages": true,
    }
}
```

### LoginMojang
After the client received a [MojangInfo](#mojanginfo) packet
and authenticating itself with mojang,
it has to send a `LoginMojang` packet to the server.
After the server receives a `LoginMojang` packet,
it will send [Success](#success) if the login was successful.

- `name` needs to be associated with the uuid.
- `uuid` is not guaranteed to be hyphenated.
- If `allow_messages` is true, other clients may send private messages
  to this client.

**Example**
```json
{
    "m": "LoginMojang",
    "c": {
        "name": "Notch",
        "uuid": "069a79f4-44e9-4726-a5be-fca90e38aaf5",
        "allow_messages": true
   }
}
```

### Message
The `content` of this packet will be sent to every client
as [Message](#message) if it fits the validation scheme.

**Example**
```json
{
    "m": "Message",
    "c": {
        "content": "Hello, World!"
    }
}
```

### PrivateMessage
The `content` of this packet will be sent to the specified client
as [PrivateMessage](#privatemessage) if it fits the validation scheme.

- `receiver` is the name of the receiver.

**Example**
```json
{
    "m": "PrivateMessage",
    "c": {
        "content": "Hello, Notch!",
        "receiver": "Notch"
    }
}
```

### RequestJWT
To login using [LoginJWT](#loginjwt), a client needs to own a json web token.
This token can be retrieved by sending `RequestJWT` as an already authenticated
client to the server.
The server will send a [NewJWT](#newjwt) packet to the client.

This packet has no body.

**Example**
```json
{
    "m": "RequestJWT"
}
```

### RequestMojangInfo
To login via mojang, the client has to send a `RequestMojangInfo` packet.
The server will then send a [MojangInfo](#mojanginfo) to the client.

This packet has no body.

**Example**
```json
{
    "m": "RequestMojangInfo"
}
```

### RequestUserCount
After receiving this packet, the server will then send a [UserCount](#usercount)
packet to the client.

This packet has no body.

**Example**
```json
{
    "m": "RequestUserCount"
}
```

### UnbanUser
A client can send this packet to unban other users.

- `user` is the uuid of the user to unban.

**Example**
```json
{
    "m": "UnbanUser",
    "c": {
        "user": "069a79f4-44e9-4726-a5be-fca90e38aaf5"
    }
}
```
