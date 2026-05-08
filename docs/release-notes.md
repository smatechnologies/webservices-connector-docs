---
sidebar_label: 'Release notes'
title: Webservices Connector release notes
description: "Version history and change details for the Webservices Connector, including new features, improvements, and bug fixes."
tags:
  - Reference
  - System Administrator
  - Connectors
---

# Webservices Connector release notes

## General

This version of the connector requires OpCon version STS 20.7 or LTS 21.0 or greater.

## 21

### 21.7

#### Fixes

:eight_spoked_asterisk: **CON-1031**: Fixed the following vulnerabilities:
- `webservices.connector.jar` contains xstream 1.3.1 (CVE-2021-39144)
- `webservices.connector.jar` contains commons-beanutils 1.8.3 (CVE-2014-0114)
- `webservices.connector.jar` contains commons-io 2.2 (CVE-2021-29425)

:eight_spoked_asterisk: **CON-1187**: Updated Webservices Docker Image removing vulnerabilities:
- Used new base image with latest Linux Agent and Java 25.
- Added Webservices 21.7.
- Removed the following additional software as images should be limited to a single function:
  - AzureStorage (now available in ACS AzureWebservices)
  - PowerShell
  - AzureCLI

### 21.6

#### Fixes

:eight_spoked_asterisk: **CON-9** and **CON-384**: Add TLS definition to each step. Requires updating both sub-type and connector. TLS value set to TLS by default to ensure backwards compatibility.

:eight_spoked_asterisk: **CON-622**: Updated multipart/form-body to correctly pass variables defined in the form-data during file upload.

:eight_spoked_asterisk: **CON-802**: Corrected a problem during GET poll loop when the step completes correctly, but then returns a 0 resulting in the task terminating in an error condition.

### 21.5

#### Fixes

:eight_spoked_asterisk: **CONNUTIL-657**: Updated JSON parsing routine to include a predicate to support more complex filters.

### 21.4

#### Upgrade considerations

During the upgrade process, both the Connector and the EM Plugin must be upgraded.

To implement the application/octet-stream media type selection correction and application/json-patch+json media selection, the Enterprise Manager WebServices sub-type must be updated.

#### What's new

:eight_spoked_asterisk: **CONNUTIL-644**: Add PATCH function to connector.

:eight_spoked_asterisk: **CONNUTIL-648**: Support json patch media type for POST and PUT functions.

#### Fixes

:eight_spoked_asterisk: **CONNUTIL-626**: Removed extra CR+LF at end of received file.

:eight_spoked_asterisk: **CONNUTIL-647**: Corrected typo in application/octet-stream in subtype drop-down list.

:eight_spoked_asterisk: **CONNUTIL-651**: Corrected octet streaming file upload for POST and PUT functions. Added header logging. Corrected POST function using application/json-patch+json media type. Adjusted multi-part file upload. Updated software libraries.

### 21.3

#### Upgrade considerations

During the upgrade process, both the Connector and the EM Plugin must be upgraded.

The **Communication Settings** value **Requires XML Escape Sequences** of the Windows Agent that the Webservices Connector is associated with must be set to **True**.

#### What's new

:eight_spoked_asterisk: **CONNUTIL-623**: Add new configuration check to allow property updates on failure. Includes new configuration option `UPDATE_PROPERTIES_ON_FAILURE`:

```
[GENERAL]
DATA_DIRECTORY=
USES_PROXY=False
UPDATE_PROPERTIES_ON_FAILURE=False
DEBUG=False
```

:eight_spoked_asterisk: **CONNUTIL-624**: Add new LF or CRLF selection when uploading message body information from file during POST and PUT requests as Windows expects CRLF and Unix/Linux expects LF for record termination. The default value is set as CRLF.

:eight_spoked_asterisk: **CONNUTIL-625**: Add the capability to extract the contents of an array item returned in a JSON structure. When using this, the JsonPath value should end in the required array value (i.e., `$.stats.[0]`).

#### Fixes

:eight_spoked_asterisk: **CONNUTIL-626**: Corrected a problem when single quote characters are used in the JSON message body. The single quote character is escaped by the Enterprise Manager plugin and then reverted by the Webservices connector prior to processing.

### 21.2

#### Upgrade considerations

To implement the application/octet-stream media type selection, the Enterprise Manager WebServices sub-type must be updated.

Provides information on running VisualCron Jobs to support RPA and on running WebServices Jobs as Windows Embedded Script jobs. Two script definitions are included in the templates section.

#### What's new

:eight_spoked_asterisk: **CONNUTIL-617**: Added new parsing type `TEXTSTRING` to support the parsing of values returned from requests for VisualCron job variable values.

:eight_spoked_asterisk: **CONNUTIL-620**: Added media type `application/octet-stream` to connector for POST and PUT requests.

#### Fixes

:eight_spoked_asterisk: **CONNUTIL-616**: Fixed a problem where password information was displayed in the log files when DEBUG was enabled.

:eight_spoked_asterisk: **CONNUTIL-618**: Fixed a problem when special characters are included in variable definitions causing an exception during variable replacement routines.

:eight_spoked_asterisk: **CONNUTIL-620**: Fixed a problem when doing a POST or PUT using a filename, the information was written into the message body without CR+LF.
