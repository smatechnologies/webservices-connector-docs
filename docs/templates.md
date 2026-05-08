---
sidebar_label: 'Templates'
title: Webservices Connector job templates
description: "Reusable JSON job templates for the Webservices Connector covering common authentication and request patterns, plus a Kubernetes deployment example."
tags:
  - Reference
  - Automation Engineer
  - Connectors
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Templates

## What is it?

This page contains JSON-formatted job templates that you can import into the **Web Services** job sub-type using the **Import Template** function. Templates accelerate job creation by providing pre-built definitions for common scenarios. Sensitive values such as user names and passwords are written as `??????` so templates can be shared without exposing credentials.

## How to use a template

To import and use a template, complete the following steps:

1. Save the JSON block from this page to a file on your workstation.
2. In Enterprise Manager, open a **Web Services** job sub-type.
3. Select **Import Template** and browse to the file.
4. On the **Global Values** tab, replace each `??????` placeholder with the value for your environment.
5. Adjust the URL host, port, and any other endpoint-specific values inside the imported steps.

To save a job you have built as a template for sharing, select **Export Template**. The connector writes sensitive variable values back as `??????` to prevent inadvertent disclosure.

For walk-through examples that explain each step's behavior, see [Example job definitions](./example-job-definitions.md).

## Choose a template

| # | Template | Template ID | Use case |
|---|----------|-------------|----------|
| 1 | [GET with no authentication](#1-get-with-no-authentication) | `OPCON-API_version` | Read the OpCon REST API version. |
| 2 | [Update an OpCon property (token auth)](#2-update-an-opcon-property-token-auth) | `OpConAPI-PUPD` | Acquire a token, look up a property ID, and update the property. |
| 3 | [Build an OpCon schedule with polling (token auth)](#3-build-an-opcon-schedule-with-polling-token-auth) | `OpConAPI-SBUILD` | Submit a schedule build and poll the build status. |
| 4 | [GET with Basic authentication](#4-get-with-basic-authentication) | `EASYVISTA` | Send an authenticated GET request using HTTP Basic auth. |
| 5 | [GET with Windows Authentication (NTLM)](#5-get-with-windows-authentication-ntlm) | *(none)* | Send an authenticated GET request to an IIS endpoint. |
| 6 | [GET with client certificate](#6-get-with-client-certificate) | `certifcate` | Send a GET request authenticated with a PKCS12 client certificate. |
| 7 | [VisualCron Embedded Script (vars / no-vars)](#7-visualcron-embedded-script-definitions) | *(none)* | Launch and monitor a VisualCron job, with or without passing job variables. |
| 8 | [Kubernetes deployment (OpCon + Deploy + Webservices)](#8-kubernetes-deployment-opcon--deploy--webservices) | — | Deploy OpCon, Deploy (Impex2), and the Webservices Connector to a Kubernetes cluster. |

:::tip Free-form template ID
The `templateid` field is a free-form identifier you can use to recognize the source of an imported template — it does not affect connector behavior. A value of `null` simply means no identifier was assigned.
:::

## 1. GET with no authentication

A single-step `GET` request that retrieves the OpCon REST API version. Useful as a smoke test for connectivity.

**Template ID:** `OPCON-API_version`

**Variables to fill in:** none.

**Endpoint values to update:**

| Field | Replace with |
|-------|--------------|
| URL `<server name>` | The hostname of your OpCon server. |

```json
{
  "templateid" : "OPCON-API_version",
  "steps" : [ {
    "function" : "GET",
    "url" : "https://<server name>:9010/api/version",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## 2. Update an OpCon property (token auth)

Three-step template that acquires an OpCon REST API token, looks up the global property ID, then updates the property value.

**Template ID:** `OpConAPI-PUPD`

**Variables to fill in:**

| Variable | Description |
|----------|-------------|
| `@User` | OpCon user name. |
| `@Password` | OpCon password. Encrypted global properties are supported (for example, `[[user_pwd]]`). |
| `@GlobalPropertyName` | Name of the global property to update. |

**Endpoint values to update:**

| Field | Replace with |
|-------|--------------|
| URL `<server name>` | The hostname of your OpCon server. |
| Step 3 body `value` | The new value to write into the global property. |

```json
{
  "templateid" : "OpConAPI-PUPD",
  "steps" : [ {
    "function" : "POST",
    "url" : "https://<server name>:9010/api/tokens",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ ],
      "body" : "{\"id\":null,\"user\":{\"id\":-1,\"loginName\":\"@User\",\"password\":\"@Password\"},\"tokenType\":{\"id\":null,\"type\":\"User\"}}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.id"
      } ]
    }
  }, {
    "function" : "GET",
    "url" : "https://<server name>:9010/api/globalproperties?name=@GlobalPropertyName",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Propertyid",
        "value" : "$[0].id"
      } ]
    }
  }, {
    "function" : "PUT",
    "url" : "https://<server name>:9010/api/globalproperties/@Propertyid",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : "{\"id\":\"@Propertyid\",\"name\":\"@GlobalPropertyName\",\"value\":\"wsrest 1234 testvalue\"}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
}, {
    "name" : "@GlobalPropertyName",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## 3. Build an OpCon schedule with polling (token auth)

Three-step template that acquires a token, submits a schedule build, then polls the build until it completes successfully or fails.

**Template ID:** `OpConAPI-SBUILD`

**Variables to fill in:**

| Variable | Description |
|----------|-------------|
| `@User` | OpCon user name. |
| `@Password` | OpCon password. Encrypted global properties are supported. |
| `@ScheduleName` | Name of the schedule to build. |
| `@ScheduleDate` | Schedule date in `yyyy-mm-dd` format. Use `[[$SCHEDULE DATE-YYYY-MM-DD]]`. |

**Endpoint values to update:**

| Field | Replace with |
|-------|--------------|
| URL `<server name>` | The hostname of your OpCon server. |

**Polling behavior (Step 3):**

| Field | Value |
|-------|-------|
| **Attribute to Check** | `$[0].message` |
| **Good Finish** | `success/Completed` |
| **Bad Finish** | `Failed` |
| **Delay** | `3` seconds |
| **Interval** | `2` seconds |
| **Max Time** | `2` minutes |

```json
{
  "templateid" : "OpConAPI-SBUILD",
  "steps" : [ {
    "function" : "POST",
    "url" : "https://<server name>:9010/api/tokens",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "body" : "{\"id\":null,\"user\":{\"id\":-1,\"loginName\":\"@User\",\"password\":
      \"@Password\"},\"tokenType\":{\"id\":null,\"type\":\"User\"}}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.id"
      } ]
    }
  }, {
    "function" : "POST",
    "url" : "https://<server name>:9010/api/schedulebuilds",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : "{\"schedules\":[{\"id\":null,\"name\":\"@ScheduleName\"}],\"id\":null,\"expiryTime\":null,\"dates\":[\"@ScheduleDate\"],\"logFile\":null,\"overwrite\":true,\"hold\":true}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 201,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Buildid",
        "value" : "$.id"
      } ]
    }
  }, {
    "function" : "GET",
    "url" : "https://<server name>:9010/api/schedulebuilds/@Buildid",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "success/Completed",
        "badFin" : "Failed",
        "attributeToCheck" : "$[0].message",
        "poll" : true,
        "pollDelay" : 3,
        "pollInterval" : 2,
        "pollMaxTime" : 2
      },
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
  }, {
    "name" : "@ScheduleName",
    "value" : "??????"
  }, {
    "name" : "@ScheduleDate",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## 4. GET with Basic authentication

Single-step `GET` request that authenticates against an EasyVista endpoint using HTTP Basic. The connector detects `Authorization: Basic` and Base64-encodes `@User:@Password` automatically.

**Template ID:** `EASYVISTA`

**Variables to fill in:**

| Variable | Description |
|----------|-------------|
| `@User` | EasyVista user name. |
| `@Password` | EasyVista password. Encrypted global properties are supported. |
| `@Company` | EasyVista company identifier used in the URL path. |

**Endpoint values to update:**

| Field | Replace with |
|-------|--------------|
| URL host | The host of the EasyVista API for your tenant if it is not `uap.easyvista.com`. |

```json
{
  "templateid" : "EASYVISTA",
  "steps" : [ {
    "function" : "GET",
    "url" : "https://uap.easyvista.com:/api/v1/@Company/requests",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Basic"
      } ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
  }, {
    "name" : "@Company",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## 5. GET with Windows Authentication (NTLM)

Single-step `GET` request that authenticates against an IIS endpoint using NTLM. The connector detects `Authorization: NTLM` and uses `@User`, `@Password`, and `@Domain` to build the credentials.

**Template ID:** *(none)*

**Variables to fill in:**

| Variable | Description |
|----------|-------------|
| `@User` | Windows user name. |
| `@Password` | Windows password. Encrypted global properties are supported. |
| `@Domain` | Windows domain. |
| `@User1` | Additional credential slot included in the template. Replace or remove if unused. |
| `@Password1` | Additional credential slot included in the template. Replace or remove if unused. |

**Endpoint values to update:**

| Field | Replace with |
|-------|--------------|
| URL `10.1.0.4` | The address of your IIS endpoint. |

```json
{
  "templateid" : null,
  "steps" : [ {
    "function" : "GET",
    "url" : "https://10.1.0.4/version.json",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "NTLM"
      } ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
  }, {
    "name" : "@Domain",
    "value" : "??????"
  }, {
    "name" : "@User1",
    "value" : "??????"
  }, {
    "name" : "@Password1",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## 6. GET with client certificate

Single-step `GET` request authenticated with a PKCS12 client certificate. The connector detects `Authorization: CERT` and loads the certificate from the keystore named in `@CertStore`.

For instructions on creating the keystore from a `.p12` client key, see [Operation > Create a keystore](./operation.md#create-a-keystore).

**Template ID:** `certifcate`

**Variables to fill in:**

| Variable | Description |
|----------|-------------|
| `@CertStore` | Full path to the keystore (for example, `c:\wsconnector\store\test.pkcs12`). |
| `@CertStorePwd` | Keystore password. Encrypted global properties are supported. |
| `@CertStoreType` | Keystore format. Currently only `PKCS12` is supported. |

**Endpoint values to update:**

| Field | Replace with |
|-------|--------------|
| URL | The endpoint that requires the client certificate (default `https://client.badssl.com` is a public test endpoint). |

```json
{
  "templateid" : "certifcate",
  "steps" : [ {
    "function" : "GET",
    "url" : "https://client.badssl.com",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
   "request" : {
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "CERT"
      } ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  } ],
  "variables" : [ {
    "name" : "@CertStore",
    "value" : "??????"
  }, {
    "name" : "@CertStorePwd",
    "value" : "??????"
  }, {
    "name" : "@CertStoreType",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
``` 
## 7. VisualCron Embedded Script definitions

These templates are used when defining Webservices Connector jobs as **OpCon Embedded Scripts** that launch and monitor VisualCron jobs through the VisualCron REST API. Both definitions follow the same five-step sequence: authenticate, look up the job ID, run the job, poll status, and read the exit code.

For background, see [Operation > Webservices jobs as Embedded Scripts](./operation.md#webservices-jobs-as-embedded-scripts) and [Example 11 — VisualCron RPA](./example-job-definitions.md#11-start-and-monitor-a-visualcron-job-rpa).

**Template ID:** *(none)*

**Required Environment Variables:**

| Variable | Description |
|----------|-------------|
| `@Url` | VisualCron REST API host (for example, `localhost:8001`). |
| `@User` | VisualCron user. |
| `@Password` | VisualCron password. |
| `@Jobname` | Name of the VisualCron job to launch. |
| `@Variables` | (vars template only) VisualCron job variable values, formatted as `varName1=value\|varName2=value`. |

Choose the variant that matches your need:

<Tabs groupId="vcron-template" queryString>
  <TabItem value="vars" label="WebServices-vcron-vars (with variables)" default>

Passes VisualCron job variable values as part of the launch request via the `variables=@Variables` query string.

```json
{
  "templateid" : null,
  "steps" : [ {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/logon?username=@User&password=@Password&expire=3600",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.Token"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/GetByName?token=@Token&name=@Jobname",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Jobid",
        "value" : "$.Id"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Run?token=@Token&id=@Jobid&variables=@Variables",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "1",
        "badFin" : "2",
        "attributeToCheck" : "$.Stats.Status",
        "poll" : true,
        "pollDelay" : 3,
        "pollInterval" : 2,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "0",
        "badFin" : null,
        "attributeToCheck" : "$.Stats.ExitCode",
        "poll" : false,
        "pollDelay" : null,
        "pollInterval" : null,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  } ],
  "variables" : [ ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
  </TabItem>
  <TabItem value="no-vars" label="WebServices-vcron-no-vars (no variables)">

Launches the VisualCron job without passing any job variables.

```json
{
  "templateid" : null,
  "steps" : [ {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/logon?username=@User&password=@Password&expire=3600",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.Token"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/GetByName?token=@Token&name=@Jobname",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Jobid",
        "value" : "$.Id"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Run?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "1",
        "badFin" : "2",
        "attributeToCheck" : "$.Stats.Status",
        "poll" : true,
        "pollDelay" : 3,
        "pollInterval" : 2,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "tlsVersion" : "TLS",
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "0",
        "badFin" : null,
        "attributeToCheck" : "$.Stats.ExitCode",
        "poll" : false,
        "pollDelay" : null,
        "pollInterval" : null,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  } ],
  "variables" : [ ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```

  </TabItem>
</Tabs>

## 8. Kubernetes deployment (OpCon + Deploy + Webservices)

A complete Kubernetes deployment yaml that deploys three pods — OpCon, Deploy (Impex2), and the Webservices Connector — backed by Azure SQL Database for the OpCon database.

### Components deployed

| Component | Image | Purpose |
|-----------|-------|---------|
| OpCon | `smatechnologies/opcon-server:latest` | OpCon server (REST API on port `9010`). |
| Deploy (Impex2) | `smatechnologies/deploy-impex2:latest` | OpCon Deploy / ImpEx2 service (web port `9011`). |
| Webservices Connector | `smatechnologies/connector-webservices:latest` | The connector itself (work port `3100`, JORS port `3110`). |

### Prerequisites

- An Azure SQL database created before deployment.
- An OpCon license value (replaces the `LICENSE` placeholder in the OpCon `ConfigMap`).
- A `kubectl` context that targets the destination cluster.

### Placeholders to replace before applying

| Placeholder | Where it appears | Replace with |
|-------------|------------------|--------------|
| `saPassword` | `uat-dbpasswords` Secret | The SA-equivalent password for the database. |
| `dbPassword` | `uat-dbpasswords` Secret | The database account password used by OpCon. |
| `sqlAdminPassword` | `uat-dbpasswords` Secret | The SQL admin password. |
| `dbPasswordEncrypted` | `uat-dbpasswords` Secret | The OpCon-encrypted form of the database password (used by Deploy). |
| `LICENSE` | `uat-opconenv` ConfigMap | Your generated OpCon `0.lic` value. |
| `DB_SERVER_NAME` | `uat-opconenv` ConfigMap and `uat-impexenv` ConfigMap (`opcon.server.name`) | Your Azure SQL server hostname. |
| `DATABASE_NAME` | `uat-opconenv` ConfigMap (and `opcon.db.name` in `uat-impexenv`) | Your OpCon database name. |
| `DB_USER_NAME` / `SQL_ADMIN_USER` | `uat-opconenv` ConfigMap | Database user names. |
| `opcon.db.user` | `uat-impexenv` ConfigMap | The Deploy database user. |
| `TZ`, `LANG` | Both ConfigMaps | Adjust to your timezone and locale. |

:::caution Replace placeholder passwords before applying
The `Secret` block contains literal `"password"` placeholder strings. Do not apply the manifest until you replace each value with a real credential and review whether to use Kubernetes secret management appropriate for your environment.
:::

### Manifest

```yaml
# 
# full OpCon deployment for Kubernetes
# includes opcon, deploy and webservices PODs
#
# uses Azure SQL Database
# database should be created before starting 
# LICENSE is OpCon 21.0 0.lic generated value
#
apiVersion: v1
kind: Secret
metadata:
  name: uat-dbpasswords
stringData:
  saPassword: "password"
  dbPassword: "password"
  sqlAdminPassword: "password"
  dbPasswordEncrypted: "encrypted password using OpCon encryption"

---
# OpCon environment values
apiVersion: v1
kind: ConfigMap
metadata:
  name: opconenv
data:
  DB_SERVER_NAME: "sscazuretest.database.windows.net"
  DATABASE_NAME: "opconxps"
  DB_USER_NAME: "admin"
  SQL_ADMIN_USER: "admin"
  API_USES_TLS: "true" 
  CREATE_API_CERTIFICATE: "true"
  DB_SETUP: "true"
  TZ: "America/Chicago"
  LANG: "en_US.utf-8"
  LICENSE: "0:414c484b040615081644696d2e28273a317e2a2a682d2d7f2a64642d636307681b1b56195d186c246d3e3e6e3c733466276a6a2968266827737331747426733d3d743a3a5e3142420f400441357d34670d0a256d24777727753a7d2f6e232360216f216e3a3a783d3d6f3a74743d737317780b0b46094d087c347d2e2e7e2c632476377a0d0a405245534654445048592366347b377e3d3d3d3d2c3e2a3a2b3f2e3a2b39283d2a3f293a2f382c382b392f392c3a2e3a293f3c293e2a3e0d0a414d5e4e5e52421e781d7c087b6e626f7e22313d30217d6f636e7f23373b36277b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b"
---
# Deploy (Impex) environment values
apiVersion: v1
kind: ConfigMap
metadata:
  name: uat-impexenv
data:
  opcon.server.name: "sscazuretest.database.windows.net"
  opcon.db.name: "opconuat"
  opcon.db.user: "sscadmin"
  web.port: "9011"
  web.ssl: "true" 
  system.debug: "false"
  TZ: "America/Chicago"
  LANG: "en_US.utf-8"
---
# OpCon persistent storage for config information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-opconconfig
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# OpCon persistent storage for log information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-opconlog
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# Deploy (Impex) persistent storage for log information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-impexlog
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# Webservices persistent storage for config information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-wsconfig
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# OpCon pod 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: uat-opcon
spec:
  replicas: 1
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - uat-opconservices
  template:
    metadata:
      labels:
        app: uat-opconservices
    spec:
      containers:
      - env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: uat-dbpasswords
              key: dbPassword
        - name: SQL_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: uat-dbpasswords
              key: sqlAdminPassword
        envFrom:
          - configMapRef:
              name: uat-opconenv
        image: smatechnologies/opcon-server:latest
        name: uat-opcon
        ports:
        - containerPort: 443
          protocol: TCP
        volumeMounts:
        - name: uat-opconconfig
          mountPath: /app/config
        - name: uat-opconlog
          mountPath: /app/log
      hostname: uat-opcon
      volumes:
      - name: uat-opconconfig
        persistentVolumeClaim:
          claimName: uat-opconconfig
      - name: uat-opconlog
        persistentVolumeClaim:
          claimName: uat-opconlog
---
# Deploy (Impex) pod
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: uat-impex
spec:
  replicas: 1
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - uat-opconservices
  template:
    metadata:
      labels:
        app: uat-opconservices
    spec:
      containers:
      - env:
        - name: opcon.db.password
          valueFrom:
            secretKeyRef:
              name: uat-dbpasswords
              key: dbPasswordEncrypted
        envFrom:
          - configMapRef:
              name: uat-impexenv
        image: smatechnologies/deploy-impex2:latest
        name: uatimpex
        volumeMounts:
        - name: uat-impexlog
          mountPath: /app/log
      hostname: uat-impex
      volumes:
      - name: uat-impexlog
        persistentVolumeClaim:
          claimName: uat-impexlog
---
# Webservices pod
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: uat-wsconnector
spec:
  replicas: 1
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - uat-wsservices
  template:
    metadata:
      labels:
        app: uat-wsservices
    spec:
      containers:
      - image: smatechnologies/connector-webservices:latest
        name: uat-wsconnector
        ports:
        - containerPort: 3100
          protocol: TCP
        - containerPort: 3110
          protocol: TCP
        volumeMounts:
        - name: uat-wsconfig
          mountPath: /app/config
      hostname: uat-wsconnector
      volumes:
      - name: uat-wsconfig
        persistentVolumeClaim:
          claimName: uat-wsconfig
---
# OpCon svc
apiVersion: v1
kind: Service
metadata:
  name: uat-lbopcon
spec:
  type: LoadBalancer
  ports:
  - name: apiport
    port: 9010
    targetPort: 443
  - name: impexport
    port: 9011
    targetPort: 9011
  selector:
    app: uat-opconservices
---
# Webservices svc
apiVersion: v1
kind: Service
metadata:
  name: uat-lbws
spec:
  type: LoadBalancer
  ports:
  - name: uatwswork
    port: 3100
    targetPort: 3100
  - name: uatwsjors
    port: 3110
    targetPort: 3110
  selector:
    app: uat-wsservices

```

## FAQs

**How do I import a template?**
Select **Import Template** in the **Web Services** job sub-type and browse to the template file. The template values populate the **Global Values** and **Steps** tabs of the job definition. Variable values stored as `??????` must be replaced with values for your environment.

**How do I export a template I created?**
Once a job definition is complete, select **Export Template**. The connector saves the definition as a JSON file. Variable and Environment Variable values are written as `??????` to prevent the inadvertent export of sensitive data.

**Where can I find additional templates?**
Templates for various integrations can be found in the SMA Technologies Innovation Lab on GitHub. Project names follow the pattern *name*`-webservicestemplate`.

**What does the `templateid` field do?**
`templateid` is a free-form identifier you can use to recognize the source of an imported template. It does not affect connector behavior.

## Glossary

| Term | Definition |
|------|------------|
| Template | A JSON-formatted SMAWSConnector job definition that can be imported into the **Web Services** job sub-type. |
| Import Template | A button in the **Web Services** job sub-type that loads a template into the job definition fields. |
| Export Template | A button in the **Web Services** job sub-type that saves the current job definition as a template, replacing sensitive values with `??????`. |
| `??????` | Placeholder used in exported templates for variable and Environment Variable values, preventing the inadvertent export of sensitive data. |
| Innovation Lab | The SMA Technologies GitHub organization where additional templates are published, with project names ending in `-webservicestemplate`. |
