
# Service Manager API

## Table of Contents

  - [Overview](#overview)
  - [Platform Management](#platform-management)
    - [Registering a Platform](#registering-a-platform)
    - [Retrieving a Platform](#retrieving-a-platform)
    - [Retrieving All Platforms](#retrieving-all-platforms)
    - [Deleting a Platform](#deleting-a-platform)
    - [Updating a Platform](#updating-a-platform)
  - [Service Broker Management](#service-broker-management)
    - [Registering a Service Broker](#registering-a-service-broker)
    - [Retrieving a Service Broker](#retrieving-a-service-broker)
    - [Retrieving All Service Brokers](#retrieving-all-service-brokers)
    - [Deleting a Service Broker](#deleting-a-service-broker)
    - [Updating a Service Broker](#updating-a-service-broker)
  - [Information](#information)
  - [Service Management](#service-management)
  - [Credentials Object](#credentials-object)
  - [Errors](#errors)
  - [Content Type](#content-type)

## Overview

The Service Manager API defines an HTTP interface that allows the management of platforms, brokers and services from a central place. In general, the Service Manager API can be split into two groups - a Service Controller API that allows the management of platforms and service brokers and an OSB compliant API. The latter implements the [Open Service Broker (OSB) API](https://github.com/openservicebrokerapi/servicebroker/) and allows the Service Manager to act as a broker.

One of the access channels to the Service Manager is via a CLI. The API should play nice in this context.
A CLI-friendly string is all lowercase, with no spaces. Keep it short -- imagine a user having to type it as an argument for a longer command.

## Platform Management

## Registering a Platform

In order for a platform to be usable with the Service Manager, the Service Manager needs to know about the platforms existence. Essentially, registering a platform means that a new service broker proxy for this particular platform has been registered with the Service Manager.

### Request

#### Route

`POST /v1/platforms`

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

#### Body

```json
{
    "id": "038001bc-80bd-4d67-bf3a-956e4d545e3c",
    "name": "cf-eu-10",
    "type": "cloudfoundry",
    "description": "Cloud Foundry on AWS in Frankfurt"
}
```

| Request field | Type | Description |
| --- | --- | --- |
| id  | string | ID of the platform. If provided, MUST be unique across all platforms registered with the Service Manager. |
| name* | string | A CLI-friendly name of the platform. MUST only contain alphanumeric characters and hyphens (no spaces). MUST be unique across all platforms registered with the Service Manager. MUST be a non-empty string. |
| type* | string | The type of the platform. MUST be a non-empty string. SHOULD be one of the values defined for `platform` field in OSB [context](https://github.com/openservicebrokerapi/servicebroker/blob/master/profile.md#context-object). |
| description | string | A description of the platform. |

\* Fields with an asterisk are REQUIRED.

### Response

| Status Code | Description |
| --- | --- |
| 201 Created | MUST be returned if the platform was registered as a result of this request. The expected response body is below. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. The `description` field MAY be used to return a user-facing error message, providing details about which part of the request is malformed or what data is missing as described in [Errors](#errors).|
| 409 Conflict | MUST be returned if a platform with the same `id` or `name` already exists. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors). |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

```json
{
    "id": "038001bc-80bd-4d67-bf3a-956e4d545e3c",
    "name": "cf-eu-10",
    "type": "cloudfoundry",
    "description": "Cloud Foundry on AWS in Frankfurt",
    "created_at": "2016-06-08T16:41:22Z",
    "updated_at": "2016-06-08T16:41:26Z",
    "credentials" : {
        "basic": {
            "username": "admin",
            "password": "secret"
        }
    }
}
```

| Response field | Type | Description |
| --- | --- | --- |
| id* | string | ID of the platform. If not provided in the request, new ID MUST be generated. |
| name* | string | Platform name. |
| type* | string | Type of the platform. |
| description | string | Platform description. |
| credentials* | [credentials](#credentials-object) | A JSON object that contains credentials which the service broker proxy (or the platform) MUST use to authenticate against the Service Manager. Service Manager SHOULD be able to identify the calling platform from these credentials. |
| created_at | string | The time of the creation in ISO-8601 format |
| updated_at | string | The time of the last update in ISO-8601 format |

\* Fields with an asterisk are REQUIRED.

## Retrieving a Platform

### Request

#### Route

`GET /v1/platforms/:platform_id`

`:platform_id` MUST be the ID of a previously registered platform.

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned if the request execution has been successful. The expected response body is below. |
| 404 Not Found | MUST be returned if the requested resource is missing. The `description` field MAY be used to return a user-facing error message, providing details about which part of the request is malformed or what data is missing as described in [Errors](#errors).|

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

##### Platform Object

```json
{
    "id": "038001bc-80bd-4d67-bf3a-956e4d545e3c",
    "name": "cf-eu-10",
    "type": "cloudfoundry",
    "description": "Cloud Foundry on AWS in Frankfurt",
    "created_at": "2016-06-08T16:41:22Z",
    "updated_at": "2016-06-08T16:41:26Z"
}
```

| Response field | Type | Description |
| --- | --- | --- |
| id* | string | ID of the platform. |
| name* | string | Platform name. |
| type* | string | Type of the platform. |
| description | string | Platform description. |
| created_at | string | The time of the creation in ISO-8601 format |
| updated_at | string | The time of the last update in ISO-8601 format |

\* Fields with an asterisk are REQUIRED.

## Retrieving All Platforms

### Request

#### Route

`GET /v1/platforms`

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned if the request execution has been successful. The expected response body is below. |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

```json
{
  "platforms": [
    {
      "id": "038001bc-80bd-4d67-bf3a-956e4d545e3c",
      "name": "cf-eu-10",
      "type": "cloudfoundry",
      "description": "Cloud Foundry on AWS in Frankfurt",
      "created_at": "2016-06-08T16:41:22Z",
      "updated_at": "2016-06-08T16:41:26Z"
    },
    {
      "id": "e031d646-62a5-4a50-9d8e-23165172e9e1",
      "name": "k8s-us-05",
      "type": "kubernetes",
      "description": "Kubernetes on GCP in us-west1",
      "created_at": "2016-06-08T17:41:22Z",
      "updated_at": "2016-06-08T17:41:26Z"
    }
  ]
}
```

| Response field | Type | Description |
| --- | --- | --- |
| platforms* | array of [platforms](#platform-object) | List of registered platforms. |

\* Fields with an asterisk are REQUIRED.

## Deleting a Platform

### Request

`DELETE /v1/platforms/:platform_id`

`:platform_id` MUST be the ID of a previously registered platform.

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned if the platform was deleted as a result of this request. The expected response body is `{}`. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors).|
| 404 Not Found | MUST be returned if the requested resource is missing. The `description` field MAY be used to return a user-facing error message, providing details about which part of the request is malformed or what data is missing as described in [Errors](#errors).|

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

For a success response, the expected response body is `{}`.

## Updating a Platform

### Request

`PATCH /v1/platforms/:platform_id`

`:platform_id` The ID of a previously registered platform.

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

#### Body

```json
{
    "name": "cf-eu-10",
    "type": "cloudfoundry",
    "description": "Cloud Foundry on AWS in Frankfurt"
}
```

| Request field | Type | Description |
| --- | --- | --- |
| name | string | A CLI-friendly name of the platform. MUST only contain alphanumeric characters and hyphens (no spaces). MUST be unique across all platforms registered with the Service Manager. MUST be a non-empty string. |
| type | string | The type of the platform. MUST be a non-empty string. SHOULD be one of the values defined for `platform` field in OSB [context](https://github.com/openservicebrokerapi/servicebroker/blob/master/profile.md#context-object). |
| description | string | A description of the platform. |

All fields are OPTIONAL. Fields that are not provided, MUST NOT be changed.

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned if the platform was updated as a result of this request. The expected response body is below. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. The `description` field MAY be used to return a user-facing error message, providing details about which part of the request is malformed or what data is missing as described in [Errors](#errors).|
| 404 Not Found | MUST be returned if the requested resource is missing. The `description` field MAY be used to return a user-facing error message, providing details about which part of the request is malformed or what data is missing as described in [Errors](#errors).|
| 409 Conflict | MUST be returned if a platform with a different `id` but the same `name` is already registered with the Service Manager. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors). |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

```json
{
    "id": "038001bc-80bd-4d67-bf3a-956e4d545e3c",
    "name": "cf-eu-10",
    "type": "cloudfoundry",
    "description": "Cloud Foundry on AWS in Frankfurt",
    "created_at": "2016-06-08T16:41:22Z",
    "updated_at": "2016-06-08T16:41:26Z"
}
```

| Response field | Type | Description |
| --- | --- | --- |
| id* | string | ID of the platform. |
| name* | string | Platform name. |
| type* | string | Type of the platform. |
| description | string | Platform description. |
| created_at | string | The time of the creation in ISO-8601 format |
| updated_at | string | The time of the last update in ISO-8601 format |

\* Fields with an asterisk are REQUIRED.

## Service Broker Management

## Registering a Service Broker

Registering a broker in the Service Manager makes the services exposed by this service broker available to all Platforms registered in the Service Manager.
Upon registration, Service Manager fetches and validate the catalog from the service broker.

### Request

#### Route

`POST /v1/service_brokers`

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

#### Body

```json
{
    "name": "service-broker-name",
    "description": "Service broker providing some valuable services",
    "broker_url": "http://service-broker-url.com",
    "credentials": {
        "basic": {
            "username": "admin",
            "password": "secret"
        }
    },
    "metadata": {

    }
}
```

| Name | Type | Description |
| ---- | ---- | ----------- |
| name* | string | A CLI-friendly name of the service broker. MUST only contain alphanumeric characters and hyphens (no spaces). MUST be unique across all service brokers registered with the Service Manager. MUST be a non-empty string. |
| description | string | A description of the service broker. |
| broker_url* | string | MUST be a valid base URL for an application that implements the OSB API |
| credentials* | [credentials](#credentials-object) | MUST be a valid credentials object which will be used to authenticate against the service broker. |
| metadata | object | Additional data associated with the service broker. This JSON object MAY have arbitrary content. |

\* Fields with an asterisk are REQUIRED.

### Response

| Status | Description |
| ------ | ----------- |
| 201 Created | MUST be returned if the service broker was registered as a result of this request. The expected response body is below. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors). |
| 409 Conflict | MUST be returned if a service broker with the same `name` is already registered. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors). |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

```json
{
    "id": "36931aaf-62a7-4019-a708-0e9abf7e7a8f",
    "name": "service-broker-name",
    "description": "Service broker providing some valuable services",
    "created_at": "2016-06-08T16:41:26Z",
    "updated_at": "2016-06-08T16:41:26Z",
    "broker_url": "https://service-broker-url",
    "metadata": {

    }
}
```

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| id*            | string | ID of the service broker. MUST be unique across all service brokers registered with the Service Manager. If the same service broker is registered multiple times, each registration will get a different ID. |
| name*          | string | Name of the service broker. |
| description    | string | Description of the service broker. |
| broker_url*    | string | URL of the service broker. |
| created_at     | string | the time of creation in ISO-8601 format |
| updated_at     | string | the time of the last update in ISO-8601 format |
| metadata | object | Additional data associated with the service broker. This JSON object MAY have arbitrary content. |

\* Fields with an asterisk are REQUIRED.

## Retrieving a Service Broker

### Request

#### Route

`GET /v1/service_brokers/:broker_id`

`:broker_id` MUST be the ID of a previously registered service broker.

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

### Response

| Status Code | Description |
| ----------- | ----------- |
| 200 OK      | MUST be returned upon successful retrieval of the service broker. The expected response body is below. |
| 404 Not Found | MUST be returned if a service broker with the specified ID does not exist. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors). |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

#### Service Broker Object

```json
{
    "id": "36931aaf-62a7-4019-a708-0e9abf7e7a8f",
    "name": "service-broker-name",
    "description": "Service broker providing some valuable services",
    "created_at": "2016-06-08T16:41:26Z",
    "updated_at": "2016-06-08T16:41:26Z",
    "broker_url": "https://service-broker-url",
    "metadata": {

    }
}
```

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| id*            | string | ID of the service broker. |
| name*          | string | Name of the service broker. |
| description    | string | Description of the service broker. |
| broker_url*    | string | URL of the service broker. |
| created_at     | string | the time of creation in ISO-8601 format |
| updated_at     | string | the time of the last update in ISO-8601 format |
| metadata | object | Additional data associated with the service broker. This JSON object MAY have arbitrary content. |

\* Fields with an asterisk are REQUIRED.

## Retrieving All Service Brokers

### Request

#### Route

`GET /v1/service_brokers`

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

### Response

| Status Code | Description |
| ----------- | ----------- |
| 200 OK      | MUST be returned upon successful retrieval of the service brokers. The expected response body is below. |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

```json
{
  "brokers": [
    {
      "id": "36931aaf-62a7-4019-a708-0e9abf7e7a8f",
      "name": "service-broker-name",
      "description": "Service broker providing some valuable services",
      "created_at": "2016-06-08T16:41:26Z",
      "updated_at": "2016-06-08T16:41:26Z",
      "broker_url": "https://service-broker-url",
      "metadata": {

      }
    },
    {
      "id": "a62b83e8-1604-427d-b079-200ae9247b60",
      "name": "another-broker",
      "description": "More services",
      "created_at": "2016-06-08T17:41:26Z",
      "updated_at": "2016-06-08T17:41:26Z",
      "broker_url": "https://another-broker-url"
    }
  ]
}
```

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| brokers* | array of [service brokers](#service-broker-object) | List of registered service brokers. |

\* Fields with an asterisk are REQUIRED.

## Deleting a Service Broker

When the Service Manager receives a delete request, it MUST delete any resources it created during registration of this service broker.

Deletion of a service broker for which there are Service Instances created MUST fail. This behavior can be overridden by specifying the `force` query parameter which will remove the service broker regardless of whether there are Service Instances created by it.


### Request

#### Route

`DELETE /v1/service_brokers/:broker_id`

`:broker_id` MUST be the ID of a previously registered service broker.

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

#### Parameters

| Name | Type | Description |
| ---- | ---- | ----------- |
| force | boolean | Whether to force the deletion of the service broker, ignoring existing Service Instances associated with it. |

### Response

| Status Code | Description |
| ----------- | ----------- |
| 200 OK      | MUST be returned upon successful deletion of the service broker. The expected response body is `{}`. |
| 400 Bad Request | Returned if the request is malformed or there are service instances associated with the service broker and `force` parameters is not `true`. |
| 404 Not Found | MUST be returned if a service broker with the specified ID does not exist. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors). |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

For a success response, the expected response body is `{}`.

## Updating a Service Broker

Updating a service broker allows to change its properties.

Updating a service broker MUST trigger an update of the catalog of this service broker.

### Request

#### Route

`PATCH /v1/service_brokers/:broker_id`

`:broker_id` MUST be the ID of a previously registered service broker.

#### Headers

The following HTTP Headers are defined for this operation:

| Header | Type | Description |
| --- | --- | --- |
| Authorization* | string | Provides a means for authentication and authorization |

\* Headers with an asterisk are REQUIRED.

#### Body

```json
{
    "name": "service-broker-name",
    "description": "Service broker providing some valuable services",
    "broker_url": "http://service-broker-url.com",
    "credentials": {
        "basic": {
            "username": "admin",
            "password": "secret"
        }
    },
    "metadata": {

    }
}
```

| Name | Type | Description |
| ---- | ---- | ----------- |
| name | string | A CLI-friendly name of the service broker. MUST only contain alphanumeric characters and hyphens (no spaces). MUST be unique across all service brokers registered with the Service Manager. MUST be a non-empty string. |
| description | string | A description of the service broker. |
| broker_url | string | MUST be a valid base URL for an application that implements the OSB API |
| credentials | [credentials](#credentials-object) | If provided, MUST be a valid credentials object which will be used to authenticate against the service broker. |
| metadata | object | Additional data associated with the service broker. This JSON object MAY have arbitrary content. |

All fields are OPTIONAL. Fields that are not provided MUST NOT be changed.

### Response

| Status Code | Description |
| ----------- | ----------- |
| 200 OK | MUST be returned if the requested changes have been applied. The expected response body is `{}` |
| 400 Bad Request | MUST be returned if the request is malformed or a forbidden modification attempt is made. |
| 409 Conflict | MUST be returned if a service broker with a different `id` but the same `name` is already registered with the Service Manager. The `description` field MAY be used to return a user-facing error message, as described in [Errors](#errors). |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

The response body MUST be a valid JSON Object (`{}`).

```json
{
    "id": "36931aaf-62a7-4019-a708-0e9abf7e7a8f",
    "name": "service-broker-name",
    "description": "Service broker providing some valuable services",
    "created_at": "2016-06-08T16:41:26Z",
    "updated_at": "2016-06-08T16:41:26Z",
    "broker_url": "https://service-broker-url",
    "metadata": {

    }
}
```

| Response Field | Type | Description |
| -------------- | ---- | ----------- |
| id*            | string | ID of the service broker. |
| name*          | string | Name of the service broker. |
| description    | string | Description of the service broker. |
| broker_url*    | string | URL of the service broker. |
| created_at     | string | the time of creation in ISO-8601 format |
| updated_at     | string | the time of the last update in ISO-8601 format |
| metadata | object | Additional data associated with the service broker. This JSON object MAY have arbitrary content. |

\* Fields with an asterisk are REQUIRED.

## Information

The Service Manager exposes publicly available information that can be used when accessing its APIs.

### Request

#### Route

`GET /v1/info`

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned upon successful processing of this request. The expected response body is below. |

Responses with any other status code will be interpreted as a failure. The response can include a user-facing message in the `description` field. For details see [Errors](#errors).

#### Body

```json
{
    "token_issuer_url": "https://example.com"
}
```

| Name | Type | Description |
| ---- | ---- | ----------- |
| token_issuer_url* | string | URL of the token issuer. The token issuer MUST have a public endpoint `/.well-known/openid-configuration` as specified by the [OpenID Provider Configuration](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig) |

\* Fields with an asterisk are REQUIRED.


## Service Management

The Service Management API is an implementation of v2.13 of the [OSB API specification](https://github.com/openservicebrokerapi/servicebroker). It enables the Service Manager to act as a central service broker and be registered as one in the  platforms that are associated with it (meaning the platforms that are registered in the Service Manager). The Service Manager also takes care of delegating the OSB calls to the registered brokers (meaning brokers that are registered in the Service Manager) that should process the request. As such, the Service Manager acts as a platform for the actual (registered) brokers.

The Service Management API prefixes the routes specified in the OSB spec with `/v1/osb/:broker_id`.

`:broker_id` is the id of the broker that the OSB call is targeting. The Service Manager MUST forward the call to this broker. The `broker_id` MUST be a globally unique non-empty string.

When a request is send to the Service Management API, after forwarding the call to the actual broker but before returning the response, the Service Manager MAY alter the body of the response. For example, in the case of `/v1/osb/:broker_id/v2/catalog` request, the Service Manager MAY, amongst other things, add additional plans (reference plan) to the catalog.

In its role of a platform for the registered brokers, the Service Manager MAY define its own format for `Context Object` and `Originating Identity Header` similar but not limited to those specified in the [OSB spec profiles page](https://github.com/openservicebrokerapi/servicebroker/blob/master/profile.md).

## Credentials Object

This specification does not limit how the Credentials Object should look like as different authentication mechanisms can be used. Depending on the used authentication mechanism, additional fields holding the actual credentials MAY be included.

| Field | Type | Description |
| --- | --- | --- |
| basic | [basic credentials](#basic-credentials-object) | Credentials for basic authentication |
| token | string | Bearer token |

_Exactly_ one of the properties `basic` or `token` MUST be provided.

### Basic Credentials Object

| Field | Type | Description |
| --- | --- | --- |
| username* | string | username |
| password* | string | password |

\* Fields with an asterisk are REQUIRED.

## Errors

When a request to the Service Manager fails, it MUST return an
appropriate HTTP response code. Where the specification defines the expected
response code, that response code MUST be used.

The response body MUST be a valid JSON Object (`{}`).
For error responses, the following fields are defined. The Service Manager MAY
include additional fields within the response.

| Response Field | Type | Description |
| --- | --- | --- |
| error | string | A single word that uniquely identifies the error condition. If present, MUST be a non-empty string with no whitespace. It MAY be used to identify the error programmatically on the client side. |
| description | string | A user-facing error message explaining why the request failed. If present, MUST be a non-empty string. |

Example:
```json
{
  "error": "InvalidCredentials",
  "description": "The supplied credentials could not be authorized"
}
```

## Content Type

All requests and responses defined in this specification with accompanying bodies SHOULD contain a `Content-Type` header set to `application/json`. If the `Content-Type` is not set, Service Brokers and Platforms MAY still attempt to process the body. If a Service Broker rejects a request due to a mismatched Content-Type or the body is unprocessable it SHOULD respond with `400 Bad Request`.
