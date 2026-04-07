# Release Notes 

## General

This version of the connector requires OpCon version STS 20.7 or LTS 21.0 or greater.

## WebServices 21.7

**CON-1031**    
					Fixed the following vulnerabilities
                    - webservices.connector.jar contains xstream 1.3.1 (CVE-2021-39144) 
                    - webservices.connector.jar contains commons-beanutils 1.8.3 (CVE-2014-0114) 
                    - webservices.connector.jar contains commons-io 2.2 (CVE-2021-29425) 
**CON-1187**
					Updated Webservices Docker Image removing vulnerabilities
					- Used new base image with latest Linux Agent and Java 25.
					- Added Webservices 21.7.
					- Removed the following additional spftware as images should be limited to a single function.
						- AzureStorage (now available in ACS AzureWebservices)
						- PowerShell
						- AzureCLI
## WebServices 21.6

**CON-9 and CON-384**    
					Add TLS definition to each step. Requires updating both sub-type and connector. TLS value set to TLS by default to ensure backwards compatibility. 
**CON-622**    
					Updated multipart/form-body to correctly pass variables defined in the from-data during file upload. 
**CON-802**    
					Corrected a problem during GET poll loop when the step completes correctly, but then returns a 0 resulting in the task terminating in an error condition. 
					
## WebServices 21.5

**CONNUTIL-657**    
					Updated JSON parsing routine to include a predicate to support more complex filters.   

## WebServices 21.4

During the upgrade process, both the Connector and the EM Plugin must be upgraded.

**CONNUTIL-626**    
					Removed extra CR+LF at end of received file.   
**CONNUTIL-644**    
					Add PATCH function to connector.   
**CONNUTIL-647**    
					Corrected typo in application/octet-stream in subtype drop-down list.   
**CONNUTIL-648**    
					Support json patch media type for POST and PUT functions.   
**CONNUTIL-651**    
					Corrected octet streaming file upload for POST & PUT functions.  \
					Add header logging.  \
					Corrected POST function using application/json-patch+json media type.  \
					Adjusted multi-part file upload.  \
					Updated software libraries.

### Upgrade Considerations

To implement the application/octet-stream media type selection correction & application/json-patch+json media selection, the Enterprise Manager WebServices sub-type must be updated.

## WebServices 21.3

During the upgrade process, both the Connector and the EM Plugin must be upgraded.

The **Communication Settings** value **Requires XML Escape Sequences** of the Windows Agent that the Webservices Connector is associated with
must be set to **True**.

**CONNUTIL-623**    
					Add new configuration check to allow property updates on failure.   
					Includes new configuration option UPDATE_PROPERTIES_ON_FAILURE

					[GENERAL]
					DATA_DIRECTORY=
					USES_PROXY=False
					UPDATE_PROPERTIES_ON_FAILURE=False
					DEBUG=False

**CONNUTIL-624**    
					Add new LF or CRLF selection when uploading message body information from file during POST & PUT requests as Windows expects CRLF and
					Unix/Linux expects LF for record termination.
					The default value is set as CRLF.
**CONNUTIL-625**    
					Add the capability to extract the contents of an array item returned in a JSON structure. When using this, the JsonPath value should
					end in the required array value (i.e $.stats.[0]).					
**CONNUTIL-626**    
					Corrected a problem when single quote characters are used in the JSON message body. The single quote character is escaped by the 
					Enterprise Manager plugin and then reverted by the Webservices connector prior to execution. 

## WebServices 21.2

Provides information on executing VisualCron Jobs to support RPA.
Provides information on running WebServices Jobs as Windows Embedded Script jobs.

### New Features

**CONNUTIL-617**    
					Added new parsing type TEXTSTRING to support the parsing of values returned from requests for VisualCron job variable values.   
**CONNUTIL-620**    
					Added media type application/octet-stream to connector for POST and PUT requests.   

### Fixes

**CONNUTIL-616**    
					Fixed a problem where password information was displayed in the log files when DEBUG was enabled.   
				
**CONNUTIL-618**    
					Fixed a problem when special characters are included in variable definitions causing an exception during variable replacement routines.
**CONNUTIL-620**    
					Fixed a problem when doing a POST or PUT using a filename, the information was written into the msg body without CR+LF.

### Upgrade Considerations

To implement the application/octet-stream media type selection, the Enterprise Manager WebServices sub-type must be updated.

Included information on using WebServices jobs as an Embedded Script for supporting VisualCron RPA jobs. Provided two script definitions in the templates section.

