---
sidebar_label: 'Operation'
title: Operating the Webservices Connector
description: "Reference for how the Webservices Connector parses responses, uses variables, handles authentication, and supports proxy servers and client certificates."
tags:
  - Reference
  - Automation Engineer
  - Connectors
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Operation

## What is it?

This page describes how the Webservices Connector processes requests and responses. It covers response parsing using JSONPath, XPath, and header parsing; the use of variables and Environment Variables; authentication mechanisms; proxy server support; and client certificate handling.

## How requests and responses work

The connector is content-type aware and works generically with all JSON and XML data — it does not need to know about specific objects or schemas. The supported Content-Types are:

| Content-Type | Direction | Notes |
|--------------|-----------|-------|
| `application/json` | Request and response | Supports JSONPath response parsing. |
| `application/xml` | Request and response | Supports XPath response parsing. |
| `text/xml` | Request and response | Required for SOAP Webservices. |
| `application/x-www-form-urlencoded` | Request | Used for OAuth2 endpoints. |
| `multipart/form-data` | Request | Used for file uploads. |
| `text/plain` | Request and response | No structured parsing. |

:::tip Use Debug mode while building jobs
Set `DEBUG=True` in `Connector.config` while you are defining steps. Debug mode dumps the data received from the web server to the job output so you can determine the JSONPath, XPath, or header parsing syntax to use.
:::

## Response parsing

The connector can extract values from a response using three syntaxes:

| Source | Content-Type | Syntax | Example |
|--------|--------------|--------|---------|
| JSON body | `application/json` | JSONPath | `$.node.node1.[0].node3` |
| XML body | `application/xml` | XPath | `//issue/id/text()` |
| Response headers | Any | `#`-prefixed header name | `#Content-Type` |
| Plain text body (VisualCron) | `text/plain` | `TEXTSTRING` | — |

### JSONPath

A JSON document is a tree of nodes; each node is either a single element or an array of elements. JSONPath defines expressions that traverse this tree to extract a value.

The examples below use these sample documents:

<Tabs groupId="jsonpath-example" queryString>
  <TabItem value="single" label="Single attribute" default>

Sample JSON:

```json
{
  "node":{
    "node1":{
     [
      {"node3":"value3"},
      {"node4":"value4"}
   ],
  }
}
}
```

To extract the value of `node3`, use:

```text
$.node.node1.[0].node3
```

| Token | Meaning |
|-------|---------|
| `$` | Root node operator. |
| `node` | A JSON node. |
| `node1` | A JSON node within `node`. |
| `[0]` | The first record of the JSONARRAY following `node1`. |
| `node3` | The attribute whose value is extracted. |

  </TabItem>
  <TabItem value="scan" label="Scan an array">

Sample JSON:

```json
{
  "node":{
    "node1":{
     [
      {"nodeType":"Type3"},
      {"nodeType":"Type4"}
     ],
    }
  }
}
```

To scan an array for the value of `nodeType`, use:

```text
$.node.node1.[*].nodeType
```

| Token | Meaning |
|-------|---------|
| `$` | Root node operator. |
| `node` | A JSON node. |
| `node1` | A JSON node within `node`. |
| `[*]` | Scan all records of the JSONARRAY following `node1`. |

  </TabItem>
  <TabItem value="index" label="Array index">

To extract the value of a specific array item:

```text
$.node.node1.[1]
```

| Token | Meaning |
|-------|---------|
| `$` | Root node operator. |
| `node` | A JSON node. |
| `node1` | A JSON node within `node`. |
| `[1]` | The required record of the JSONARRAY following `node1`. |

  </TabItem>
</Tabs>

### XPath

XPath (XML Path Language) uses a path-like syntax to identify and navigate nodes in an XML document.

Sample XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<issue>
  <id>1495</id>
  <project id="1" name="Default Project"/><tracker id="2" name="Deliverable"/>
  <status id="2" name="New"/>
  <css_classes>issue tracker-2 status-2 priority-12 priority-default created-by-me multieditable-container</css_classes>
  <priority id="12" name="Normal"/>
  <author id="1" name="John Smith"/>
  <subject>API custom fields</subject>
  <description></description>
  <start_date>2020-07-09</start_date>
  <due_date/>
  <done_ratio>0</done_ratio>
  <is_private>false</is_private>
  <is_favorited>false</is_favorited>
  <estimated_hours/>
  <total_estimated_hours/>
  <spent_hours>0.0</spent_hours>
  <total_spent_hours>0.0</total_spent_hours>
  <created_on>2020-07-09T09:38:33Z</created_on>
  <updated_on>2020-07-09T09:38:33Z</updated_on>
  <closed_on/>
</issue>
```

<Tabs groupId="xpath-example" queryString>
  <TabItem value="element" label="Element value" default>

To extract the value of element `id`:

```text
//issue/id/text()
```

| Token | Meaning |
|-------|---------|
| `//` | Root element operator. |
| `issue` | The name of an element. |
| `id` | The element whose value is required. |
| `text()` | The value of the `id` element is required. |

  </TabItem>
  <TabItem value="attribute" label="Attribute value">

To extract the value of attribute `name` on element `author`:

```text
//issue/author/@name
```

| Token | Meaning |
|-------|---------|
| `//` | Root element operator. |
| `issue` | The name of an element. |
| `author` | The element whose attribute is required. |
| `@` | Indicates an attribute value is required. |
| `name` | The name of the attribute. |

  </TabItem>
</Tabs>

### Header parsing

Parse a returned header by prefixing the attribute name with a hash (`#`).

The syntax to retrieve the returned data type is:

```text
#Content-Type
```

| Token | Meaning |
|-------|---------|
| `#` | Indicates a header parsing definition. |
| `Content-Type` | The header attribute to extract the value from. |

### Text strings

Some endpoints (notably VisualCron job variables) return a single text string instead of a structured payload. Use the value `TEXTSTRING` in the response variable to insert the entire returned result into the variable without parsing.

## Variables

### Defining variables

All variables are defined as `@<name>`. The `@` prefix tells the connector that the token is a variable and should be substituted at run time. Variables can be used in URLs, request headers, and message bodies.

| Variable scope | Available to |
|----------------|--------------|
| Global variable | All steps in the job. |
| Response variable | All steps after the step that defined it. |

### Authentication modes

The connector recognizes the authentication mode from the `Authorization` header value and uses the variables you define to build credentials. The mode is set per step.

| Mode | `Authorization` header | Required variables | What the connector does |
|------|------------------------|--------------------|-------------------------|
| Basic | `Basic` | `@User`, `@Password` | Performs Base64 encoding of `@User:@Password` and adds it to the header. |
| NTLM (Windows / IIS) | `NTLM` | `@User`, `@Password`, `@Domain` | Builds NTLM credentials from the variables. |
| Token | `Token <token>` | `@Token` (typically populated from a prior step) | Passes the token value through to the target server. |
| Certificate | `CERT` | `@CertStore`, `@CertStorePwd`, `@CertStoreType` | Loads the named keystore and authenticates with the client certificate. |

:::note SOAP Webservices
For SOAP requests, set `Message-Type=SOAP` in the request header and select Content-Type `text/xml`.
:::

### Special variables

The following variable names are reserved by the connector:

| Variable | Description |
|----------|-------------|
| `@User` | The name of the user associated with the requests. |
| `@Password` | The password of the user. Encrypted global properties can be used. |
| `@Domain` | The domain associated with the user when using Windows Authentication to IIS. |
| `@CertStore` | The location of the keystore when client certificates are used. |
| `@CertStorePwd` | The password of the keystore. Encrypted global properties can be used. |
| `@CertStoreType` | The format of the client key in the keystore. Currently `PKCS12` is the only supported format. |
| `@JCorrelationid` | Creates the unique ID of a job in the daily tables that can be used during call back procedures. |
| `@I_<name>` | Indicates that the variable contains an integer value. |

#### @JCorrelationid

Use `@JCorrelationid` when you start a process on a web server and want the web server to signal job completion back to OpCon.

The value `@JCorrelationid` corresponds to the next job in the processing sequence. The value should follow the format:

```text
[[SCHEDULE DATE-YYYY-MM-DD]].[[$SCHEDULE NAME]].JOB001
```

The schedule date is in `YYYY-MM-DD` format. Create a `$SCHEDULE DATE` property to provide this value.

At run time, the connector calls the OpCon REST API to retrieve the unique job ID for that job and passes it in the JSON payload to the web server.

When the work on the web server completes, it sends an `/api/jobactions` request back to OpCon using the unique job ID and sets the job status to either `markFinishedOk` or `markFailed`.

:::caution Use a Threshold dependency, not "On Hold"
The next job must be held on a Threshold dependency that is never triggered — not on an `On Hold` condition. "Late to Start" monitoring does not evaluate jobs that are in the `On Hold` state.
:::

#### @I_\<name\> integer variables

The `@I_` prefix tells the connector to substitute the variable as an integer rather than a string.

Without `@I_`, the placeholder in a JSON payload is treated as a string, so substitution preserves the surrounding quotes:

```json
"fixno" : "[[version]]"   →   "fixno" : "122"
```

With `@I_`, the connector strips the quotes during substitution:

```json
"fixno" : "@I_version"    →   "fixno" : 122
```

To use this:

1. Create a global variable `@I_version` with a value of `[[version]]`.
2. In the JSON payload, use the `@I_version` variable name where the integer should be inserted.

### Environment Variables

Environment Variables are an alternative variable mechanism used for values that would break JSON parsing if substituted into the OpCon job definition (most often Windows paths with unescaped backslashes).

Environment Variables differ from regular variables in three ways:

| Aspect | Variable | Environment Variable |
|--------|----------|----------------------|
| Where it lives | Inside the JSON job definition | OS environment of the agent |
| Subject to JSON parsing | Yes | No |
| Standard OpCon property resolution | No (raw substitution) | Yes |

The connector reads all OS environment variables that begin with `@` during start-up and adds them to the list of variables available for substitution.

:::note Agent must support Environment Variables
The Windows or Linux Agent supporting the connector must support the Environment Variables feature, because the agent is responsible for setting the variables in the run environment for each job.
:::

## x-www-form-urlencoded content type

The `application/x-www-form-urlencoded` content type is used when an authentication endpoint expects OAuth2-style form data. Enter the body as `name=value` pairs separated by an ampersand.

```text
grant_type=client_credentials&client_id=@Clientid&client_secret=@Key&resource=https://management.azure.com/
```

## Templates

Templates are reusable JSON job definitions that can be imported into the **Web Services** job sub-type. Additional templates are published in the SMA Technologies Innovation Lab on GitHub under projects named *name*`-webservicestemplate`.

For ready-to-import examples, see the [Templates](./templates.md) page.

## Proxy servers

### Proxy modes

The connector supports two ways of specifying a proxy server. If both are configured, the step-level value wins.

| Scope | Where to configure | Use when |
|-------|-------------------|----------|
| All jobs | `[PROXY]` section of `Connector.config` | Every job through this connector requires the same proxy. |
| Single step | **Proxy Server** field of the **Step** definition | You need multiple proxies, or a mix of proxied and direct jobs. |

### All jobs through one proxy

Set `USES_PROXY=True` and the proxy URL in `Connector.config`. See the [Connector.config file](./installation.md#connectorconfig-file) section in Installation for full details.

```ini
[GENERAL]
DATA_DIRECTORY=c:\\connectors\\wsrest\\data
USES_PROXY=True
DEBUG=False

[PROXY]
URL=http://bvhtest02:2134

[OPCON_API]
SERVER=BVHTEST02:9010
USESTLS=True
TOKEN=e4185480-7137-4bca-8220-0dccc555a946
```

### Per-step proxy

Enter the full proxy server URL in the **Proxy Server** field on the **Step** definition. A step-level value overrides the value in `Connector.config`.

## Client certificates

When using client certificates, you create a keystore that holds the certificate and reference it from the connector through variables.

### Create a keystore

To create a keystore for a received client key, complete the following steps:

1. Create a `store` directory off the connector root installation directory.
2. Place the received client key file (*keyname*`.p12`) in the `store` directory.
3. Run `keytool` (located in the `\java\bin` directory of the connector installation) to create the keystore:

   ```bash
   keytool -importkeystore \
     -srckeystore c:\wsconnector\store\test-client.p12 \
     -destkeystore c:\wsconnector\store\test.pkcs12 \
     -srcstoretype pkcs12
   ```

4. When prompted, enter a new password for the keystore and the password supplied with the client key file.

| Argument | Description |
|----------|-------------|
| `-importkeystore` | Creates the keystore. |
| `-srckeystore` | Path to the received certificate (`p12` format). |
| `-destkeystore` | Path of the keystore to create. The certificate is inserted into this keystore. |
| `-srcstoretype` | Format of the received key. |

:::caution Only PKCS12 is supported
Set `@CertStoreType` to `PKCS12`. The connector does not support other keystore formats.
:::

### Map the keystore to variables

After the keystore is created, set these variables in the job definition:

| Variable | Value |
|----------|-------|
| `@CertStore` | Full path to the keystore (the `-destkeystore` value, for example, `c:\wsconnector\store\test.pkcs12`). |
| `@CertStorePwd` | The password you set when creating the keystore. |
| `@CertStoreType` | `PKCS12` |

## Webservices jobs as Embedded Scripts

You can define Webservices jobs using OpCon Embedded Scripts. The connector ships with two Embedded Script definitions for running VisualCron jobs.

:::note Working Dir
When defining Webservices jobs as Embedded Script jobs, the **Working Dir** value must point to the Webservices installation directory.
:::

### Supplied scripts

| Script name | Description |
|-------------|-------------|
| `WebServices-vcron-vars` | Includes variable definitions that are passed to the VisualCron job as VisualCron job variables. |
| `WebServices-vcron-no-vars` | No variables are passed to the VisualCron job. |

### Required Environment Variables

Set the following Environment Variables for each run request:

| Environment Variable | Description |
|----------------------|-------------|
| `@Url` | The URL used to connect to the VisualCron REST API (`localhost:8001`). |
| `@User` | The VisualCron user that will be used to run the VisualCron job. |
| `@Password` | The password of the VisualCron user. |
| `@Jobname` | The name of the VisualCron job. |
| `@Variables` | Optional. Required for the `WebServices-vcron-vars` script. Consists of the VisualCron job variable name and the value. Multiple variables are separated by the pipe character (`varName1=value\|varName2=value`). |

### Install the connector as an Embedded Script

To install the Webservices Connector as an Embedded Script, complete the three procedures below in order.

#### 1. Define the script type

1. Go to **Scripts > Types** and select the **Add** button.
2. Set the following values:

   | Field | Value |
   |-------|-------|
   | **Name** | `WebServices` |
   | **File Extension** | `.web` |
   | **Description** | `Running WebServices as an embedded script` |

#### 2. Define the script runner

1. Go to **Scripts > Runners** and select the **Add** button.
2. Set the following values:

   | Field | Value |
   |-------|-------|
   | **OS** | `Windows` |
   | **Type of Script** | `WebServices` |
   | **Command Template** | `install-dir\SMAWSConnector.exe $FILE $ARGUMENTS` |

#### 3. Install the supplied scripts

To install both supplied scripts, complete the following steps once for each script:

1. Go to **Scripts > Repository** and select the **Add** button.
2. Set the values from the table below for the script you are installing.
3. Paste the corresponding script body from the templates section into the **Script** field.

| Field | First script | Second script |
|-------|--------------|---------------|
| **Name** | `WebServices-vcron-vars` | `WebServices-vcron-no-vars` |
| **Description** | `WebServices script to call VisualCron passing variables` | `WebServices script to call VisualCron without variables` |
| **Type** | `WebServices` | `WebServices` |

## FAQs

**Which content types can the connector parse for response data?**
Response parsing is supported for `application/json` using JSONPath and `application/xml` using XPath. Headers can be parsed using a hash-prefixed syntax (`#Content-Type`). VisualCron text-string responses can be captured with `TEXTSTRING`.

**How does the connector recognize a variable?**
Names that begin with `@` are treated as variables and substituted into URLs, headers, and message bodies at run time. Variables prefixed with `@I_` are substituted as integer values rather than strings.

**When should I use Environment Variables instead of Variables?**
Use Environment Variables when an OpCon property value contains characters (such as backslashes in Windows paths) that would break JSON parsing if substituted into the job definition. Environment Variables are passed through the agent rather than through the JSON definition.

**How does the connector handle authentication?**
Authentication mode is determined by the `Authorization` header value: `Basic` for Base64-encoded `@User:@Password`, `NTLM` for Windows Authentication using `@User`, `@Password`, and `@Domain`, `Token` for token-based authentication, and `CERT` for client certificate authentication.

**What is `@JCorrelationid` used for?**
`@JCorrelationid` retrieves the unique job ID of the next job in the daily so an external web server can return a completion status to OpCon using the REST API `/api/jobactions` endpoint.

**Where can I configure a proxy server?**
Either globally in the `[PROXY]` section of `Connector.config`, or per step in the **Proxy Server** field of the **Step** definition. A step-level value overrides the global value.

## Glossary

| Term | Definition |
|------|------------|
| JSONPath | A query syntax used to identify and extract values from JSON data. The Webservices Connector uses JSONPath for `application/json` responses. |
| XPath | A query syntax used to identify and navigate nodes in an XML document. The Webservices Connector uses XPath for `application/xml` responses. |
| Header parsing | A connector feature that extracts a value from a response header using `#`-prefixed syntax (for example, `#Content-Type`). |
| Variable | A named value, prefixed with `@`, that the connector substitutes into URLs, headers, or message bodies at run time. |
| Environment Variable | A variable, prefixed with `@`, passed to the agent through the OS environment rather than through the JSON job definition. |
| Special variable | A reserved variable name (`@User`, `@Password`, `@Domain`, `@JCorrelationid`, `@CertStore`, `@CertStorePwd`, `@CertStoreType`) recognized by the connector for credentials, correlation, or certificate handling. |
| Integer variable | A variable prefixed with `@I_` whose value is substituted into the JSON payload as an integer rather than a string. |
| Keystore | A file (PKCS12 format) that holds a client certificate and is referenced by the `@CertStore` variable. |
| Embedded Script | A method of defining Webservices Connector jobs as scripts run by an agent. |
