---
sidebar_label: 'Example job definitions'
title: Example Webservices Connector job definitions
description: "Sample job definitions showing token authentication, polling, OAuth2, Basic, NTLM, certificate authentication, file upload, SOAP, and VisualCron RPA scenarios."
tags:
  - Reference
  - Automation Engineer
  - Connectors
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Example job definitions

## What is it?

This page provides sample Webservices Connector job definitions showing common capabilities of the connector. Each example covers the variables to define on the **Global Values** tab and the steps to define on the **Steps** tab.

For a reference to all the fields shown in these examples, see [Defining jobs](./defining-jobs.md).

## Choose an example

| # | Example | What it demonstrates |
|---|---------|----------------------|
| 1 | [Token authentication (multi-step)](#1-token-authentication-multi-step) | Acquire a token, look up a property ID, update the property. |
| 2 | [Poll a long-running operation](#2-poll-a-long-running-operation) | Submit a schedule build and poll the build status. |
| 3 | [OAuth2 with x-www-form-urlencoded](#3-oauth2-with-x-www-form-urlencoded) | Acquire an OAuth2 token from Azure AD. |
| 4 | [Basic Mode authentication](#4-basic-mode-authentication) | Send a request with HTTP Basic auth. |
| 5 | [Use Environment Variables](#5-use-environment-variables) | Pass an OpCon property containing a Windows path through the agent environment. |
| 6 | [Save a returned value into an OpCon property](#6-save-a-returned-value-into-an-opcon-property) | Use Property Update on Completion to write a response value back to OpCon. |
| 7 | [Windows Authentication (NTLM)](#7-windows-authentication-ntlm) | Authenticate to IIS using NTLM. |
| 8 | [Client certificate authentication](#8-client-certificate-authentication) | Authenticate using a PKCS12 keystore. |
| 9 | [File upload](#9-file-upload) | Upload a file using `multipart/form-data`. |
| 10 | [SOAP Webservices](#10-soap-webservices) | Submit a SOAP envelope and extract a value from the response. |
| 11 | [Start and monitor a VisualCron job (RPA)](#11-start-and-monitor-a-visualcron-job-rpa) | Use the VisualCron REST API to launch and track a VisualCron job. |

:::tip Reading these examples
Each step in an example shows only the fields you need to set. Fields not shown — such as **Proxy Server** and **TLS** — keep their defaults. The connector uses TLS automatically when the URL begins with `https`.
:::

## 1. Token authentication (multi-step)

**Goal:** update the value of an OpCon global property using token authentication.

The OpCon REST API uses token authentication. The first step obtains a token, the second step looks up the property ID, and the third step updates the property value.

### Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@Url` | `SERVER1:9010` |
| `@User` | `user` |
| `@Password` | `[[user_pwd]]` |
| `@GlobalProperty` | `PROPTEST` |

The password is held in an encrypted global property (`[[user_pwd]]`).

### Step 1 — Acquire a token

| Field | Value |
|-------|-------|
| **Function** | `POST` |
| **URL** | `https://@Url/api/tokens` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

**Request body:**

```json
{
    "id":null,
    "user":
    {
        "id":-1,
        "loginName":"@User",
        "password":"@Password"
    },
    "tokenType":
    {
        "id":null,
        "type":"User"
    }
}
```

**Response variable:**

| Name | JSONPath |
|------|----------|
| `@Token` | `$.id` |

`$.id` extracts the value of the first `id` attribute at the JSON root and stores it in `@Token` for use by later steps.

### Step 2 — Look up the global property ID

| Field | Value |
|-------|-------|
| **Function** | `GET` |
| **URL** | `https://@Url/api/globalproperties?name=@GlobalProperty` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Authorization` | `Token @Token` |

The connector substitutes `@Token` with the value extracted in Step 1.

**Response variable:**

| Name | JSONPath |
|------|----------|
| `@Propertyid` | `$[0].id` |

The response is an array of property definitions. `$[0].id` selects the `id` of the first array element.

### Step 3 — Update the global property

| Field | Value |
|-------|-------|
| **Function** | `PUT` |
| **URL** | `https://@Url/api/globalproperties/@Propertyid` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

The `@Propertyid` extracted in Step 2 is substituted into the URL.

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Authorization` | `Token @Token` |

**Request body:**

```json
{
    "id":"@Propertyid",
    "name":"@GlobalProperty",
    "value":"wsrest 1234 test value from UNIX 2"
}
```

`@Propertyid` is also substituted in the body because the OpCon REST API expects the property ID in the payload when updating.

## 2. Poll a long-running operation

**Goal:** trigger an OpCon schedule build and poll the build status until it succeeds or fails.

### Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@Url` | `SERVER1:9010` |
| `@User` | `user` |
| `@Password` | `[[user_pwd]]` |
| `@ScheduleName` | `SCHED001` |
| `@ScheduleDate` | `[[$SCHEDULE DATE-YYYY-MM-DD]]` |

:::note Date format
`@ScheduleDate` must be in `yyyy-mm-dd` format. Use a `$SCHEDULE DATE` property to supply this.
:::

### Step 1 — Acquire a token

Same as [Step 1 of Example 1](#step-1--acquire-a-token).

### Step 2 — Submit the schedule build

| Field | Value |
|-------|-------|
| **Function** | `POST` |
| **URL** | `https://@Url/api/schedulebuilds` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `201` |

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Authorization` | `Token @Token` |

**Request body:**

```json
{
    "schedules":[
        {"id":null,"name":"@ScheduleName"}
    ],
    "id":null,
    "expiryTime":null,
    "dates":[
        "@ScheduleDate"
    ],
    "logFile":null,
    "overwrite":true,
    "hold":true
}
```

**Response variable:**

| Name | JSONPath |
|------|----------|
| `@Buildid` | `$.id` |

### Step 3 — Poll the build status

| Field | Value |
|-------|-------|
| **Function** | `GET` |
| **URL** | `https://@Url/api/schedulebuilds/@Buildid` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Authorization` | `Token @Token` |

**Step Completion (returned-data check + polling):**

| Field | Value |
|-------|-------|
| **Attribute to Check** | `$[0].message` |
| **Good Finish** | `success/Completed` |
| **Bad Finish** | `Failed` |
| **Poll** | enabled |
| **Delay** | seconds before the first check |
| **Interval** | seconds between subsequent checks |
| **Max Time** | `5` minutes |

| Outcome | Connector return code |
|---------|----------------------|
| Good Finish matched | `200` |
| Bad Finish matched | `460` |
| Max Time expired | `408` |

## 3. OAuth2 with x-www-form-urlencoded

**Goal:** acquire an OAuth2 access token from Azure Active Directory.

The body is sent as `name=value` pairs separated by an ampersand (`&`).

### Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@Url` | `login.microsoftonline.com` |
| `@Tenantid` | `[[AZURE_TENANT_ID]]` |
| `@Clientid` | `[[AZURE_CLIENT_ID]]` |
| `@Key` | `[[AZURE_KEY]]` |
| `@Subscription` | `[[AZURE_SUBSCRIPTION]]` |

:::caution
All OAuth2 credentials should be held in encrypted global properties.
:::

### Step 1 — Acquire an OAuth2 token

| Field | Value |
|-------|-------|
| **Function** | `POST` |
| **URL** | `https://@Url/@Tenantid/oauth2/token` |
| **Request Content-Type** | `application/x-www-form-urlencoded` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

**Request body:**

```text
grant_type=client_credentials&client_id=@Clientid&client_secret=@Key&resource=https://management.azure.com/
```

**Response variable:**

| Name | JSONPath |
|------|----------|
| `@Token` | `$.access_token` |

## 4. Basic Mode authentication

**Goal:** send a request authenticated with HTTP Basic.

The connector detects `Authorization: Basic` and Base64-encodes the `@User:@Password` string into the header automatically.

### Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@Url` | `easyredmine.com` |
| `@User` | `USER001` |
| `@Password` | `[[USER001_PWD]]` |

### Step 1 — Send the authenticated request

| Field | Value |
|-------|-------|
| **Function** | `GET` |
| **URL** | `https://@Url/issues/182.xml` |
| **Request Content-Type** | `application/xml` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Authorization` | `Basic` |

## 5. Use Environment Variables

**Goal:** use a body file whose path comes from an OpCon schedule instance property containing Windows path separators.

OpCon resolves properties into the JSON job definition at run time without escaping backslashes — a value like `c:\production\main` would break JSON parsing. Passing the value as an Environment Variable bypasses the JSON definition entirely.

For background, see [Operation > Environment Variables](./operation.md#environment-variables).

### Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@Url` | `SERVER1:9010` |
| `@User` | `user` |
| `@Password` | `[[user_pwd]]` |

### Environment Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@MAIN_PATH` | `[[SI.MAIN_PATH]]` |

`SI.MAIN_PATH` is a schedule instance property containing the directory where the body file lives (for example, `c:\production\main`).

### Step 1 — POST a body loaded from a file

| Field | Value |
|-------|-------|
| **Function** | `POST` |
| **URL** | `https://@Url/api/tokens` |
| **Request Content-Type** | `application/json` |
| **Body source** | **File Name** field |
| **File Name** | `@MAIN_PATH\wsfiles\login.json` |
| **Response Content-Type** | `application/json` |

**File contents (`login.json`):**

```json
{
    "id":null,
    "user":
    {
        "id":-1,
        "loginName":"@User",
        "password":"@Password"
    },
    "tokenType":
    {
        "id":null,
        "type":"User"
    }
}
```

**Response variable:**

| Name | JSONPath |
|------|----------|
| `@Token` | `$.id` |

## 6. Save a returned value into an OpCon property

**Goal:** capture a value from the response and persist it into an OpCon schedule instance property when the job completes successfully.

### Property Update on Completion (Global Values tab)

| Property path | Source variable |
|---------------|-----------------|
| `SI.Version.[[$SCHEDULE DATE-YYYY-MM-DD]].SCHED001` | `@Version` |

:::caution
- The dot (`.`) is used as a field separator in property paths. Schedule and job names must not contain a dot.
- Date values must be in `yyyy-mm-dd` format. Use a `$SCHEDULE DATE` property to supply this value.
:::

### Step 1 — Read the OpCon REST API version

| Field | Value |
|-------|-------|
| **Function** | `GET` |
| **URL** | `https://@Url/api/version` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

**Response variable:**

| Name | JSONPath |
|------|----------|
| `@Version` | `$.opConRestApiProductVersion` |

When the job completes successfully, the connector writes the value of `@Version` into the schedule instance property defined in **Property Update on Completion**.

## 7. Windows Authentication (NTLM)

**Goal:** authenticate to an IIS endpoint using Windows credentials.

### Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@Url` | `OPCON01` |
| `@User` | `USER001` |
| `@Password` | `[[USER001_PWD]]` |
| `@DOMAIN` | `OPCON001` |

### Step 1 — Send the authenticated request

| Field | Value |
|-------|-------|
| **Function** | `GET` |
| **URL** | `https://@Url/version` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Authorization` | `NTLM` |

The connector detects `NTLM` and uses `@User`, `@Password`, and `@Domain` to build the credentials.

## 8. Client certificate authentication

**Goal:** authenticate to a server that requires a client certificate.

### Variables (Global Values tab)

| Variable | Value |
|----------|-------|
| `@CertStore` | `c:\wsconnector\store\badssl.pkcs12` |
| `@CertStorePwd` | `[[BADSSL_PWD]]` |
| `@CertStoreType` | `PKCS12` |

For instructions on creating the keystore from a `.p12` client key, see [Operation > Create a keystore](./operation.md#create-a-keystore).

### Step 1 — Send the authenticated request

| Field | Value |
|-------|-------|
| **Function** | `GET` |
| **URL** | `https://client.badssl.com` |
| **Request Content-Type** | `application/json` |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Authorization` | `CERT` |

The connector detects `CERT` and loads the certificate from the keystore named in `@CertStore`.

## 9. File upload

**Goal:** upload a file to a web server using `multipart/form-data`.

:::note File location
The file must be present on the server where the connector is installed.
:::

### Step 1 — Upload the file

| Field | Value |
|-------|-------|
| **Function** | `POST` |
| **URL** | `http://httpbin.org/post` |
| **Request Content-Type** | `multipart/form-data` |
| **Body source** | **File Name** field |
| **File Name** | full path of the file to upload |
| **Record termination** | `CRLF` (Windows targets) or `LF` (Unix/Linux targets) |
| **Response Content-Type** | `application/json` |
| **Completion Code** | `200` |

To submit additional form fields alongside the file, enter them in the **Message Body** as `name=value&name1=value1`.

## 10. SOAP Webservices

**Goal:** submit a SOAP envelope to a SOAP web service and extract a value from the response.

A SOAP request requires:

| Setting | Value |
|---------|-------|
| **Function** | `POST` |
| **Header attribute** | `Message-Type=SOAP` |
| **Request Content-Type** | `text/xml` |
| **Response Content-Type** | `application/xml` |
| **Body** | The complete SOAP envelope (`soap12:Envelope`) |

:::tip Find the SOAP envelope shape
Retrieve the WSDL by opening the service URL with `?wsdl` appended in a browser (for example, `https://www.w3schools.com/xml/tempconvert.asmx?wsdl`). The WSDL lists the supported endpoints.
:::

### Step 1 — Convert Celsius to Fahrenheit

| Field | Value |
|-------|-------|
| **Function** | `POST` |
| **URL** | `https://www.w3schools.com/xml/tempconvert.asmx` |
| **Request Content-Type** | `text/xml` |
| **Response Content-Type** | `application/xml` |
| **Completion Code** | `200` |

**Request header:**

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `Message-Type` | `SOAP` |

**Request body (SOAP envelope):**

```xml
<?xml version="1.0" encoding="utf-8"?>
<soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
  <soap12:Body>
    <CelsiusToFahrenheit xmlns="https://www.w3schools.com/xml/">
      <Celsius>50</Celsius>
    </CelsiusToFahrenheit>
  </soap12:Body>
</soap12:Envelope>
```

**Response variable:**

| Name | XPath |
|------|-------|
| `@Fahrenheit` | `//CelsiusToFahrenheitResponse/CelsiusToFahrenheitResult/text()` |

## 11. Start and monitor a VisualCron job (RPA)

**Goal:** use the VisualCron REST API to authenticate, start a job, monitor it to completion, and optionally retrieve a job variable value.

A VisualCron job is similar to an OpCon sub-schedule and contains one or more tasks. The connector monitors the job — not individual tasks — for completion.

:::note VisualCron uses GET for everything
The VisualCron REST API uses `GET` for all requests. Job parameters are passed as query-string arguments.
:::

### Recommended variables

Use Webservices variables to define the URL, user, password, and job name so the job is portable.

| Variable | Purpose |
|----------|---------|
| `@Url` | VisualCron REST API host. |
| `@User`, `@Password` | VisualCron credentials used to authenticate. |
| `@Jobname` | Name of the VisualCron job to launch. |
| `@Variables` | Optional. VisualCron job variable values, formatted as `varName1=value\|varName2=value`. |

### Step sequence

| # | Purpose | URL | Response variable |
|---|---------|-----|-------------------|
| 1 | Authenticate | `http://@Url/VisualCron/json/logon?username=@User&password=@Password&expire=3600` | `@Token` from `$.Token` |
| 2 | Look up job ID | `http://@Url/VisualCron/json/Job/GetByName?token=@Token&name=@Jobname` | `@Jobid` from `$.Id` |
| 3 | Start the job | `http://@Url/VisualCron/json/Job/Run?token=@Token&id=@Jobid&variables=@Variables` (or omit the `variables=` parameter) | — |
| 4 | Poll the job status | `http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid` | Check `$.Stats.Status` (Good Finish `1`, Bad Finish `2`) |
| 5 | Read the exit code | `http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid` | Check `$.Stats.ExitCode` (Good Finish `0`) |
| 6 | (Optional) Read a job variable | `http://@Url/VisualCron/json/JobVariableValue/Get?token=@Token&jobId=@Jobid&id=jobvarname` | Use `TEXTSTRING` |

### VisualCron status codes

The status returned in `$.Stats.Status` uses the following codes:

| Status | Meaning |
|--------|---------|
| `0` | Job is running. |
| `1` | Job is waiting. In VisualCron, a job returns to the waiting state when it completes. |
| `2` | Paused. |

### VisualCron exit codes

After completion, read the exit code from `$.Stats.ExitCode`:

| Exit code | Connector return |
|-----------|------------------|
| `0` (Good Finish) | `200` |
| Any other value | `404` |

### Reading a VisualCron job variable

VisualCron job variable values are returned as a plain text string, not JSON. Capture the result with a response variable whose value is `TEXTSTRING`. The connector then inserts the entire returned payload into the variable without parsing it.

## FAQs

**Where do I define variables that are reused across steps?**
On the **Variables** section of the **Global Values** tab. Variables are referenced by name (prefixed with `@`) in URLs, headers, and message bodies. Values from a previous step's response can be assigned to a variable on the **Response Variable Management** tab and used in later steps.

**How is a token from an authentication step passed to later steps?**
Define a response variable (for example, `@Token`) with a JSONPath expression that extracts the token from the authentication response. Subsequent steps reference `@Token` in the `Authorization` header value (for example, `Token @Token`).

**When should I use Environment Variables instead of Variables?**
When an OpCon property value contains characters (such as unescaped backslashes in Windows paths) that would break JSON parsing. Environment Variables are passed to the agent through the OS environment rather than through the JSON job definition.

**How do I retrieve a VisualCron job variable value?**
Issue a `GET` request to `http://@Url/VisualCron/json/JobVariableValue/Get?token=@Token&jobId=@Jobid&id=jobvarname`. Because VisualCron returns a plain text string rather than JSON or XML, define a response variable with the value `TEXTSTRING` to capture the result.

**What completion codes does the connector return for poll requests?**
The connector returns `200` (or the configured **Completion Code**) on a Good Finish, `460` on a Bad Finish, and `408` if the **Max Time** value expires before a Good Finish or Bad Finish is detected.

**How do I upload a file?**
Select `multipart/form-data` as the Content-Type, enter the full file path in the **File Name** field of the **Body** tab, and select the record termination string (`CRLF` for Windows targets, `LF` for Unix/Linux targets). To submit additional form fields, add them in the **Message Body** as `name=value&name1=value1`.

## Glossary

| Term | Definition |
|------|------------|
| Token authentication | Authentication mode where credentials are exchanged for a short-lived token that is passed in the `Authorization` header on subsequent requests. |
| Basic authentication | Authentication mode that Base64-encodes `@User:@Password` and passes it in the `Authorization` header. |
| NTLM | Windows Authentication mode used for IIS endpoints. Requires `@User`, `@Password`, and `@Domain` variables. |
| OAuth2 | Token-based authentication mode that uses `application/x-www-form-urlencoded` and credentials such as `client_id`, `client_secret`, and `tenant_id`. |
| Polling | Connector capability that re-issues a request on a configured interval until a Good Finish or Bad Finish is matched, or **Max Time** expires. |
| TEXTSTRING | A response variable value that tells the connector to capture the entire returned payload as a plain text string rather than parse it as JSON or XML. |
| `multipart/form-data` | A Content-Type used to upload a file. Requires a record termination selection (`CRLF` or `LF`) on the **Body** tab. |
| SOAP envelope | The XML message body submitted to a SOAP web service. Requires `Message-Type=SOAP` in the request header and a Content-Type of `text/xml`. |
