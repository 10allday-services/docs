.. _server_aitc_api_20:

==========================
Apps-In-The-Cloud API v1.0
==========================

The AITC API defines a HTTP web service used to store and retrieve stuctured
JSON records that specify a user's **Apps** and **Devices**.


Status: DRAFT
=============

This document is currently in **draft** status.  While no substantial
modifications are expected, there may be small changes and clarifications
as implementation progresses.


.. _aitc_records:

Record Formats
==============

All records are JSON documents, and have both "full" and "abbreviated" forms.
Each document has a maximum size of 8 kilobytes.

.. _aitc_app_records:

App Records
-----------

A full **App Record** document contains the following fields:

+------------------+-----------+-----------------------+----------------------------------------------------+
| Field            | Default   | Type                  |  Description                                       |
+==================+===========+=======================+====================================================+
| origin           | required  | string                | The origin URL from which the app was installed.   |
+------------------+-----------+-----------------------+----------------------------------------------------+
| manifestPath     | required  | string                | The location relative to *origin* where the app's  |
|                  |           |                       | manifest was found.                                |
+------------------+-----------+-----------------------+----------------------------------------------------+
| installOrigin    | required  | string                | The URL of the marketplace from which the app      |
|                  |           |                       | was installed.                                     |
+------------------+-----------+-----------------------+----------------------------------------------------+
| installedTime    | required  | integer,              | The time at which the application was initially    |
|                  |           | millisecond timestamp | installed; set by the server on first write.       |
+------------------+-----------+-----------------------+----------------------------------------------------+
| modificationTime | required  | integer,              | The time at which the application record was last  |
|                  |           | millisecond timestamp | modified; set by the server on each write.         |
+------------------+-----------+-----------------------+----------------------------------------------------+
| receipts         | required  | array of strings      | List of the user's purchase recepits for this      |
|                  |           |                       | application.  Receipts are opaque strings          |
+------------------+-----------+-----------------------+----------------------------------------------------+


Example::

    {
       origin: "https://example.com",
       manifestPath: "/manifest.webapp",
       installOrigin: "https://marketplace.mozilla.org",
       installedTime: 1330535996745,
       modificationTime: 1330535996945,
       receipts: ["...", "..."]
    }

An *abbreviated* App Record contains only the "origin" and "modificationTime"
fields.  Example::

    {
      origin: "https://example.com",
      modificationTime: 1330535996945
    }


App records are uniquely identified by their *appid*, which is the SHA1 hash
of the origin URL, base64url-encoded with no padding.

.. _aitc_device_records:

Device Records
--------------

A *full* **Device Record** document contains the following fields:

+-------------+-----------+---------------------------+----------------------------------------------------+
| Field       | Default   | Type                      |  Description                                       |
+=============+===========+==========================++====================================================+
| uuid        | required  | string,                   | A unique identifier for the device.                |
|             |           | uppercase hexadecimal     |                                                    |
|             |           | in 8-4-4-4-12 UUID format |                                                    |
+-------------+-----------+---------------------------+----------------------------------------------------+
| name        | required  | string,                   | A human-readable description of the device.        |
|             |           | non-empty                 |                                                    |
+-------------+-----------+---------------------------+----------------------------------------------------+
| type        | required  | string,                   | XXX ??? WHAT IS THIS FOR ??? XXX                   |
|             |           | non-empty                 |                                                    |
+-------------+-----------+---------------------------+----------------------------------------------------+
| layout      | required  | string,                   | An identifier determining the specific format of   |
|             |           | non-empty                 | object stored in the "apps" field.                 |
+-------------+-----------+---------------------------+----------------------------------------------------+
| addedAt     | required  | integer,                  | The time at which this device was added to the     |
|             |           | millisecond timestamp     | user's account.                                    |
+-------------+-----------+---------------------------+----------------------------------------------------+
| modifiedAt  | required  | integer,                  | The time at which this device's details were last  |
|             |           | millisecond timestamp     | modified; set by the server on each write.         |
+-------------+-----------+---------------------------+----------------------------------------------------+
| apps        | required  | JSON object               | An arbitrary JSON object describing the apps on    |
|             |           |                           | device and how they are arranged.  The precise     |
|             |           |                           | details of this object will depend on the value of |
|             |           |                           | the "layout" field.                                |
+-------------+-----------+---------------------------+----------------------------------------------------+


Example::

    {
       uuid: "75B538D8-67AF-44E8-86A0-B1A07BE137C8",
       name: "Anant's Mac Pro",
       type: "mobile",
       layout: "android/phone",
       addedAt: "1330535996745",
       modifiedAt: "1330535996945",
       apps: {}
    }


An *abbreviated* Device Record contains all fields except "apps".  Example::

    {
       uuid: "75B538D8-67AF-44E8-86A0-B1A07BE137C8",
       name: "Anant's Mac Pro",
       type: "mobile",
       layout: "android/phone",
       addedAt: "2012-02-28 12:23:35Z",
       modifiedAt: "2012-03-05 13:23:34Z"
    }


Device records are uniquely identified by their *uuid* field.


API Access and Discovery
========================


The AITC data for a given user may be accessed via authenticated
HTTP requests to their AITC API endpoint.  All requests will be
to URLs of the form::

    https://<endpoint-url>/<api-instruction>

The user's AITC endpoint URL can be obtained via the Sagrada Discovery
and Authentication workflow [1]_.  All requests must be signed using MAC
Access Authentication credentials [2]_.

Request and response bodies are all JSON-encoded.

The AITC API has a set of :ref:`respcodes` to cover errors in the
request or on the server side. The format of a successful response is
defined in the appropriate request method section.


.. [1] https://wiki.mozilla.org/Services/Sagrada/ServiceClientFlow
.. [2] https://wiki.mozilla.org/Services/Sagrada/ServiceClientFlow#Access


API Instructions
================


Apps
----

APIs in this section provide access to the app records stored for the currently
authenticated user.

**GET https://<endpoint-url>/apps/**

    Returns an object giving an array of apps records::

        {
          apps: [apps records for the user]
        }

    By default abbreviated records are returned.  Full records can be
    requested using the **full** parameter as described below.

    This request has additional optional parameters:

    - **newer**: a timestamp in milliseconds. Only records that were last
      modified after this time will be returned.
    - **full**: any value.  If provided then the response will contain a list
      of full records rather than abbreviated records.

    Possible HTTP status codes:

    - **304 Not Modified:**  no app records have been modified or deleted
      since the timestamp in the *X-If-Modified-Since* header.


**GET** **https://<endpoint-url>/apps/<appid>**

    Returns the full app record with the given appid.

    Possible HTTP error responses:

    - **304 Not Modified:**  the record has not been modified since the
      timestamp in the *X-If-Modified-Since* header.
    - **404 Not Found:**  the user has no app record with the given id.


**PUT** **https://<endpoint-url>/apps/<appid>**

    Create or update an app record with the given id.  The id must be
    the SHA1 hash of the app record's origin field, base64url-encoded
    with no padding.

    This request may include the *X-If-Unmodified-Since* header to avoid
    overwriting the data if it has been changed since the client fetched it.
    Successful requests will receive a **204 No Content** response, with the
    *X-Timestamp* header giving the new modification time of the object.

    Note that records are limited to 8KB in size.

    Possible HTTP error responses:

    - **400 Bad Request:**  the record is malformed or otherwise invalid.
    - **412 Precondition Failed:**  the record has been modified since the
      timestamp in the *X-If-Unmodified-Since* header.
    - **413 Request Entity Too Large:**  the record is larger than the
      server is willing to store.


**DELETE** **https://<endpoint-url>/apps/<appid>**

    Delete the app record with the given id.

    This request may include the *X-If-Unmodified-Since* header to avoid
    deleting the data if it has been changed since the client fetched it.
    Successful requests will receive a **204 No Content** response.

    Possible HTTP error responses:

    - **404 Not Found:**  the user has no app record with the given id.
    - **412 Precondition Failed:**  the record has been modified since the
      timestamp in the *X-If-Unmodified-Since* header.


Apps
----

APIs in this section provide access to the device records stored for the
currently authenticated user.

**GET https://<endpoint-url>/devices/**

    Returns an object giving an array of device records::

        {
          devices: [device records for the user]
        }

    By default abbreviated records are returned.  Full records can be
    requested using the **full** parameter as described below.

    This request has additional optional parameters:

    - **newer**: a timestamp in milliseconds. Only records that were last
      modified after this time will be returned.
    - **full**: any value.  If provided then the response will contain a list
      of full records rather than abbreviated records.

    Possible HTTP status codes:

    - **304 Not Modified:**  no device records have been modified or deleted
      since the timestamp in the *X-If-Modified-Since* header.


**GET** **https://<endpoint-url>/devices/<uuid>**

    Returns the full device record with the given uuid.

    Possible HTTP error responses:

    - **304 Not Modified:**  the record has not been modified since the
      timestamp in the *X-If-Modified-Since* header.
    - **404 Not Found:**  the user has no device record with the given id.


**PUT** **https://<endpoint-url>/devices/<uuid>**

    Create or update a device record with the given id.  The uuid must be
    be uppercase hexadecimal in 8-4-4-4-12 UUID format, and must match the
    uuid contained in the uploaded record.

    This request may include the *X-If-Unmodified-Since* header to avoid
    overwriting the data if it has been changed since the client fetched it.
    Successful requests will receive a **204 No Content** response, with the
    *X-Timestamp* header giving the new modification time of the object.

    Note that records are limited to 8KB in size.

    Possible HTTP error responses:

    - **400 Bad Request:**  the record is malformed or otherwise invalid.
    - **412 Precondition Failed:**  the record has been modified since the
      timestamp in the *X-If-Unmodified-Since* header.
    - **413 Request Entity Too Large:**  the record is larger than the
      server is willing to store.


**DELETE** **https://<endpoint-url>/devices/<uuid>**

    Delete the device record with the given id.

    This request may include the *X-If-Unmodified-Since* header to avoid
    deleting the data if it has been changed since the client fetched it.
    Successful requests will receive a **204 No Content** response.

    Possible HTTP error responses:

    - **404 Not Found:**  the user has no app record with the given id.
    - **412 Precondition Failed:**  the record has been modified since the
      timestamp in the *X-If-Unmodified-Since* header.


Request Headers
===============

**X-If-Modified-Since**

    This header may be added to any GET request to avoid transmission of the
    resource body if it has not been modified since the client last fetched
    it.  It is similar to the standard If-Modified-Since header except the
    value is expressed in milliseconds.

    If the value of this header is not a valid integer, a **400 Bad Request**
    response will be returned.


**X-If-Unmodified-Since**

    This header may be added to any PUT or DELETE request, set to a timestamp.
    If the record to be acted on has been modified since the timestamp given,
    the request will fail.  It is similar to the the standard
    If-Unmodified-Since header except the value is expressed in milliseconds.

    If the value of this header is not a valid integer, a **400 Bad Request**
    response will be returned.


Response Headers
================

**Retry-After**

    When sent together with an HTTP 503 status code, this header signifies that
    the server is undergoing maintenance. The client should not attempt any
    further requests to the server for the number of seconds specified in
    the header value.

**X-Backoff**

    This header may be sent to indicate that the server is under heavy load
    but is still capable of servicing requests.  Unlike the **Retry-After**
    header, **X-Backoff** may be included with any type of response, including
    a **200 OK**.

    Clients should perform the minimum number of additional requests required
    to maintain consistency of their stored data, then not attempt any futher
    requests for the number of seconds specified in the header value.

**X-Timestamp**

    This header will be sent back with all responses, indicating the current
    timestamp on the server. If the request was a PUT or POST, this will
    also be the modification date of any BSOs modified by the request.


HTTP status codes
=================

Since the aitc protocol is implemented on top of HTTP, clients should be
prepared to deal gracefully with any valid HTTP response.  This section serves
to highlight the response codes that explicitly form part of the aitc
protocol.


**200 OK**

    The request was processed successfully, and the server is returning
    useful information in the response body.

**204 Not Content**

    The request was processed successfully, and the server has no useful
    data to return in the response body.


**304 Not Modified**

    For requests the included the *X-If-Modified-Since* header, this response
    code indicates that the resource has not been modified.  The client should
    continue to use its local copy of the data.


**400 Bad Request**

    The request itself or the data supplied along with the request is invalid.
    The response contains a numeric code indicating the reason for why the
    request was rejected. See :ref:`respcodes` for a list of valid response
    codes.


**401 Unauthorized**

    The authentication credentials are invalid on this node. This may be caused
    by a node reassignment or by an expired/invalid auth token. The client
    should check with the auth server whether the user's node has changed. If
    it has changed, the current sync is to be aborted and should be retried
    against the new node.


**404 Not Found**

    The requested resource could not be found. This may be returned for **GET**
    and **DELETE** requests that reference non-existent records.


**405 Method Not Allowed**

    The request URL does not support the specific request method.  For example,
    attempting a PUT request to /info/quota would produce a 405 response.


**412 Precondition Failed**

    For requests that include the *X-If-Unmodified-Since* header, this response
    code indicates that the resource was in fact modified.  The requested write
    operation will not have been performed.


**413 Request Entity Too Large**

    The body submitted with a write request (PUT, POST) was larger than the
    server is willing to accept.  For multi-record POST requests, the client
    should retry by sending the records in smaller batches.


**503 Service Unavailable**

    Indicates that the server is undergoing maintenance.  Such a response will
    include a  *Retry-After* header, and the client should not attempt
    another sync for the number of seconds specified in the header value.
    The response body may contain a JSON string describing the server's status
    or error.

