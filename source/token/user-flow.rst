=========
User Flow
=========

Here's the proposed two-step flow (with Browser ID):

1. the client trades a browser id assertion for an :term:`Auth token` and
   corresponding secret
2. the client uses the auth token to sign subsequent requests using
   :term:`MAC Access Auth`.


Getting an :term:`Auth token` ::


    Client                       Login Server                  BID         User DB          Node Assignment Server
    ===========================================================================================================
                                    |                          |             |                    |
    request token ---- [1] -------->|------> verify --- [2] -->|             |                    |
                                    |       get node -- [3] ---|------------>|--> lookup          |
                                    |                          |             |<-- return node     |
                                    |  attribute node --[4]----|-------------|------------------->|--> set node
                                    |                          |             |                    |<-- node
                                    |<--- build token   [5]    |             |                    |
    keep token <-------- [6] -------|                          |             |                    |

Calling the :term:`Service` ::

    Client                                          Service Node
    ============================================================
    create signed auth header [7]    |                 |
    call node --------------- [8] ---|---------------->|--> verify token [9]
                                     |                 |    verify request signature [10]
                                     |                 |<-- process request [11]
    get response <-------------------|-----------------|


Detailed steps:

- the client requests a token, giving its browser id assertion [1]::

     GET /1.0/sync/request_token HTTP/1.1
     Host: token.services.mozilla.com
     Authorization: Browser-ID <assertion>

- the :term:`Login Server` checks the browser id assertion [2] **this step will be
  done locally without calling an external browserid server -- but this could
  potentially happen** (we can use pyvep + use the BID.org certificate)

- the :term:`Login Server` asks the Users DB if the user is already allocated to a
  :term:`Node` [3]

- If the user is not allocated to a :term:`Node`, the :term:`Login Server` asks a
  new one to the :term:`Node Assignment Server` [4]

- the :term:`Login Server` creates a response with an :term:`Auth Token` and
  corresponding :term:`Token Secret` [5] and sends it back to the user.

  The :term:`Auth Token` contains the user id and a timestamp, and is signed
  using the :term:`Signing Secret`. The :term:`Token Secret` is derived from
  the :term:`Master Secret` and :term:`Auth Token` using :term:`HKDF`.

  It also adds the :term:`Node` url in the response under
  *api_endpoint* [6]

  ::

    HTTP/1.1 200 OK
    Content-Type: application/json

    {'id': <token>,
     'secret': <derived-secret>,
     'uid': 12345,
     'api_endpoint': 'https://example.com/app/1.0/users/12345',
    }

- the client saves the node location and oauth parameters to use in subsequent
  requests. [6]

- for each subsequent request to the :term:`Service`, the client calculates a
  special Authorization header using :term:`MAC Access Auth` [7] and sends
  the request to the allocated node location [8]::

    POST /request HTTP/1.1
    Host: some.node.services.mozilla.com
    Authorization: MAC id=<auth-token>
                       ts="137131201",   (client timestamp)
                       nonce="7d8f3e4a",
                       mac="bYT5CMsGcbgUdFHObYMEfcx6bsw="

- the node uses the :term:`Signing Secret` to validate the :term:`Auth Token` [9].  If invalid
  or expired then the node returns a 401

- the node calculates the :term:`Token Secret` from its :term:`Master Secret` and the
  :term:`Auth Token`, and checks whether the signature in the Authorization header is
  valid [10]. If it's an invalid then the node returns a 401

- the node processes the request as defined by the :term:`Service` [11]

