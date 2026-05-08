---
sidebar_label: 'Defining jobs'
title: Defining Webservices Connector jobs
description: "How to define a Webservices Connector job, configure global values, define steps, and set failure criteria."
tags:
  - Procedural
  - Reference
  - Automation Engineer
  - Connectors
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Defining jobs

## What is it?

This page describes how to define a Webservices Connector job in Enterprise Manager. It covers the **Global Values**, **Steps**, and **Failure Criteria** tabs of the **Web Services** job sub-type, including variables, Environment Variables, property updates on completion, request and response definitions, authentication options, and connector return codes.

## Job definition overview

A Webservices Connector job is configured across three tabs. Use this overview to orient yourself before reading the detail sections.

| Tab | What you configure | Section |
|-----|-------------------|---------|
| **Global Values** | Variables, Environment Variables, and properties to update when the job completes successfully. | [Global Values tab](#global-values-tab) |
| **Steps** | The sequence of HTTP requests the connector performs (URL, function, headers, body, response handling). | [Steps tab](#steps-tab) |
| **Failure Criteria** | The expected completion code of the final step. | [Failure Criteria tab](#failure-criteria-tab) |

## Selecting the job sub-type

To create a Webservices Connector job, complete the following steps:

1. Create a Windows or UNIX/Linux job.
2. Select the **Web Services** job sub-type.
3. Select an OpCon batch user. The connector runs as this user.
4. Set the connector location to the global property that points to the connector installation directory (`WS_PATH` on Windows, `WS_UNIX_PATH` on Linux or Docker — see [Installation](./installation.md#step-3-create-the-global-property)).

:::tip Use templates to bootstrap a job
The job sub-type has a **Template ID** field that uniquely identifies the template used to create the job.

- **Import Template** loads a template file into the fields. Variable and Environment Variable values are presented as `??????`.
- **Export Template** saves the current job as a template. Sensitive values are written as `??????` to prevent inadvertent export.
:::

## Global Values tab

The **Global Values** tab has three sub-tabs:

| Sub-tab | Purpose |
|---------|---------|
| **Variables** | Define `@`-prefixed variables substituted into URLs, headers, and bodies at run time. |
| **Environment Variables** | Define variables passed to the agent through the OS environment, bypassing JSON parsing. |
| **Property Update on Completion** | Map response variables to OpCon properties to be written when the job completes successfully. |

### Variables sub-tab

Variables are global to the job and available to every step. Reference them by name (prefixed with `@`) in URLs, request headers, and message bodies.

To define a variable:

1. Enter `@` followed by a name in the **Name** field.
2. Enter the value in the **Value** field.

The connector substitutes the variable's value wherever the variable name appears in the step definition.

:::caution Backslashes break JSON parsing
When a variable's value comes from an OpCon property, OpCon resolves the property at task start-up and inserts the raw value into the JSON job definition. Backslashes are not escaped — a value such as `\\server\directory` breaks JSON parsing.

For values that contain backslashes or other JSON-unsafe characters, use an [Environment Variable](#environment-variables-sub-tab) instead.
:::

#### URL replacement example

```text
@Url = server:9010

https://@Url/api/tokens   →   https://server:9030/api/tokens
```

#### JSON body replacement example

```text
@User = test
@Password = [[testpwd]]    (an encrypted property)
```

The variable references are substituted in place:

```text
{
  "id":null,
  "user":
  {
    "id":-1,
    "loginName":"@User",      →  "loginName":"test"
    "password":"@Password"    →  "password":"<encrypted property definition>"
  },
  "tokenType":
  {
    "id":null,
    "type":"User"
  }
}
```

The encrypted password value is decrypted by the target OpCon agent at run time.

:::note Encrypted properties require agent support
Storing user code and passwords in encrypted global properties is supported, but the Windows agent where the connector is installed must support this functionality.
:::

### Environment Variables sub-tab

Environment Variables are an alternative variable mechanism that bypasses JSON parsing. They are passed to the agent through the OS environment instead of through the OpCon job definition. The agent loads them into the run environment, and the connector reads them on start-up.

Use Environment Variables when an OpCon property value contains characters (such as backslashes in Windows paths) that would break JSON parsing.

To define an Environment Variable:

1. Enter `@` followed by a name in the **Name** field.
2. Enter the value in the **Value** field.

#### Example

```text
@MAIN_PATH = [[SI.MAIN_PATH]]
```

Reference the Environment Variable in a step:

```text
@MAIN_PATH\jfiles\login.json
```

The connector substitutes `[[SI.MAIN_PATH]]` into the file path at run time.

:::note Agent must support Environment Variables
The agent where the connector is installed must support the Environment Variables feature.
:::

### Property Update on Completion sub-tab

Use **Property Update on Completion** to save values extracted from response data into OpCon properties when all steps complete successfully. Three property scopes are supported:

| Scope | Used for |
|-------|----------|
| Global | OpCon-wide values. |
| Schedule instance (`SI.`) | Values associated with the schedule. |
| Job instance | Values associated with a specific job in the daily. |

#### Example

```text
SI.Version.[[$SCHEDULE DATE-YYYY-MM-DD]].API_EXP_TEST_CJOB001[SUB_API_EXP_TEST] = @Version
```

This writes the contents of the `@Version` response variable to the `Version` schedule instance property of the `API_EXP_TEST_CJOB001[SUB_API_EXP_TEST]` sub-schedule, for the date supplied by `[[$SCHEDULE DATE-YYYY-MM-DD]]`.

:::caution Avoid dots in schedule and job names
The dot (`.`) character is used as a field separator in property paths. A schedule or job name that contains a dot causes the property resolution to fail.
:::

:::caution Use the `yyyy-mm-dd` date format
The OpCon REST API expects dates in `yyyy-mm-dd` format. When targeting an instance property, create a `$SCHEDULE DATE` property in this format and use it (for example, `[[$SCHEDULE DATE-YYYY-MM-DD]]`).
:::

## Steps tab

The **Steps** tab defines the sequence of HTTP requests the connector performs. Steps run in order from first to last. Each step is shown on its own tab labeled `Step(n)`.

### Adding and removing steps

| Action | How |
|--------|-----|
| Create the first step | Enter the step data, then select **Add Step**. The step is created as `Step(1)`. |
| Add another step | Select the **+** tab, enter the step data, then select **Add Step**. The new step is `Step(n+1)`. |
| Remove a step | Select the step, then select **Remove Step**. Subsequent step numbers are renumbered. |

### Step fields

Each step has the following top-level fields:

| Field | Description |
|-------|-------------|
| **Function** | The HTTP function: `GET`, `POST`, `PUT`, or `DELETE`. |
| **URL** | The full URL of the web server request, including `http` or `https`, address, and port. The connector uses TLS automatically when the URL contains `https`. |
| **Proxy Server** | Optional. Full proxy URL to use for this step. Overrides the `Connector.config` proxy if both are set. |
| **TLS** | TLS version: `TLS` (default), `TLSv1.0`, `TLSv1.1`, `TLSv1.2`, or `TLSv1.3`. |

### Request Information

The **Request** sub-tab defines the Content-Type, header attributes, and (for POST and PUT) the payload.

#### Header tab

Enter header attributes by setting **Attribute Name** and **Attribute Value**:

| Attribute Name | Attribute Value |
|----------------|-----------------|
| *name* | *value* |
| *name* | `@variable` |

You can use variable values previously defined or created by a previous step.

##### Authentication modes

The connector recognizes the authentication mode from the `Authorization` header value (or `Message-Type` for SOAP). Set the **Header** values as shown below for each mode:

| Mode | Attribute Name | Attribute Value | Required variables | What the connector does |
|------|----------------|-----------------|--------------------|-------------------------|
| Basic | `Authorization` | `Basic` | `@User`, `@Password` | Performs Base64 encoding of `@User:@Password` and adds it to the header. |
| NTLM (Windows / IIS) | `Authorization` | `NTLM` | `@User`, `@Password`, `@Domain` | Builds NTLM credentials from the variables. |
| Token | `Authorization` | `Token @Token` | `@Token` (typically populated from a prior step) | Passes the token through to the target server. |
| Certificate | `Authorization` | `CERT` | `@CertStore`, `@CertStorePwd`, `@CertStoreType` | Loads the named keystore and authenticates with the client certificate. |
| SOAP | `Message-Type` | `SOAP` | — | Treats the request as a SOAP message. Set the Content-Type to `text/xml`. |

#### Body tab

Provide the request payload using one of two options:

<Tabs groupId="body-source" queryString>
  <TabItem value="message" label="Message Body" default>

Enter the payload directly in the **Message Body** field.

  </TabItem>
  <TabItem value="filename" label="File Name">

Enter the full path to a file containing the payload in the **File Name** field.

:::note File location
The file must be readable from the system where the connector is installed.
:::

  </TabItem>
</Tabs>

### Response Information

The **Response** sub-tab defines the expected Content-Type, response variables to extract, and step completion criteria.

#### Response Variable Management

Response Variable Management extracts values from the response payload and saves them into variables that subsequent steps can use. Attribute extraction is supported when the Content-Type is `application/json` or `application/xml`.

To define a response variable:

1. Enter `@` followed by a unique name in the **Name** field.
2. Enter a JSONPath (for JSON) or XPath (for XML) expression in the **Value** field.

| Attribute Name | Attribute Value |
|----------------|-----------------|
| `@Token` | `$.id` |
| `@Value` | `//issue/id/text()` |

For JSONPath and XPath syntax reference, see [Operation > Response parsing](./operation.md#response-parsing).

:::caution Missing attributes terminate the step
If the attribute is not found in the response, the connector terminates with an error code of `1`.
:::

#### Step Completion

Step Completion defines what makes a step succeed or fail. The connector evaluates three layered checks:

##### 1. HTTP Completion Code

| Field | Description |
|-------|-------------|
| **Completion Code** | The HTTP return code expected from the step. The workflow stops if the actual return code does not match. |
| **Ignore Result** | Bypass the **Completion Code** check. |

##### 2. Returned-data check

The HTTP return code only confirms whether the request reached the server. To validate the *content* of the response, use a returned-data check:

| Field | Description |
|-------|-------------|
| **Attribute to Check** | JSONPath or XPath expression identifying the value to compare. |
| **Good Finish** | Match values that indicate success. Multiple values separated by `/`. |
| **Bad Finish** | Match values that indicate failure. Multiple values separated by `/`. |

:::tip Scan a JSONARRAY for a match
Use a star (`*`) instead of a numeric index in the JSONPath to scan all records of an array until a match is found. JSONARRAY scanning is not supported in poll mode.
:::

##### 3. Polling

Polling re-issues the same request on an interval until a Good Finish or Bad Finish is matched, or until **Max Time** expires.

| Field | Unit | Description |
|-------|------|-------------|
| **Poll** | — | Enables polling. |
| **Delay** | Seconds | Wait before the first poll request. |
| **Interval** | Seconds | Wait between subsequent poll requests. |
| **Max Time** | Minutes | Maximum total time to wait for a valid response. |

| Outcome | Connector return code |
|---------|----------------------|
| Bad Finish matched | `408` |
| Max Time expired before any match | `460` |

#### Output File

Optionally, write the response data to a file by entering a filename in the **Output File** field. The location must be writable from the system where the connector is installed.

## Failure Criteria tab

The **Failure Criteria** tab defines the result of the workflow. The result is the completion code of the final step.

### Connector return codes

| Code | Meaning |
|------|---------|
| `1` | Initialization error or initial connection error. |
| `400`–`599` | Standard HTTP errors. |
| `408` | **Bad Finish** matched during a poll request sequence. |
| `460` | **Max Time** expired during a poll request sequence before a valid result. |

## Logging and job output

The connector writes its activity to a rolling log file:

| Property | Value |
|----------|-------|
| File name | `webservices.log` |
| Location | *installation root*`\log` directory |
| Rotation | Maximum cycle of five log files; new entries are appended |
| Contents | Connector activity, job activity, error messages, and return codes |

All information produced by the OpCon job is also captured in the job output and can be retrieved with the OpCon JORS capability.

## FAQs

**How do I import or export a job template?**
Select **Import Template** to load a template file into the job sub-type. Select **Export Template** to save the current definition as a template. Variable values are replaced with `??????` in the exported template to prevent inadvertent disclosure of sensitive data.

**What's the difference between Variables and Environment Variables?**
Variables are substituted into the JSON job definition at run time. Environment Variables are passed to the agent through the OS environment and are not part of the JSON definition. Use Environment Variables for values that would break JSON parsing (for example, Windows paths containing unescaped backslashes).

**Which authentication modes does the connector support?**
Basic (`Authorization: Basic`), Windows NTLM (`Authorization: NTLM`), token (`Authorization: Token <token>`), and client certificate (`Authorization: CERT`).

**How do I extract a value from a response and use it in a later step?**
On the **Response** tab of the step, define a response variable with a unique `@`-prefixed name and a JSONPath or XPath expression. The connector populates the variable with the extracted value, and subsequent steps can reference it by name.

**How do I poll a web server until a job completes?**
Select the **Poll** field on the **Step Completion** tab and set **Delay**, **Interval**, and **Max Time**. Define **Good Finish** and **Bad Finish** values to be matched against the **Attribute to Check**. The connector returns `408` on a Bad Finish and `460` if **Max Time** expires before a valid result is detected.

**What does the connector return when a step fails?**
A connector-internal error returns `1`. Standard HTTP errors return their HTTP status (`400`–`599`). A Bad Finish during polling returns `408`, and a poll timeout returns `460`.

## Glossary

| Term | Definition |
|------|------------|
| Step | A single request to a web server within a Webservices Connector job. Steps run in sequence. |
| Template | A JSON-formatted SMAWSConnector job definition that can be imported into the **Web Services** job sub-type. |
| Global variable | A variable defined on the **Global Values** tab and available to all steps. |
| Response variable | A variable populated from a step's response and available to subsequent steps. |
| Property Update on Completion | Configuration that writes variable values into OpCon properties when all steps complete successfully. |
| Completion Code | The HTTP return code expected from a step. The workflow stops if the actual return code does not match. |
| Good Finish / Bad Finish | Values matched against the response **Attribute to Check** to determine step success or failure. |
| Poll | A step option that re-issues requests on a defined interval until a Good Finish or Bad Finish is matched, or until **Max Time** expires. |
| JORS | OpCon Job Output Retrieval System, used to retrieve job output produced by the connector. |
