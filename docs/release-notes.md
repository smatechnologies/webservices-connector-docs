# Release Notes 

## General

This version of the connector requires OpCon version STS 20.7 or LTS 21.0 or greater.

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
					Corrected octet streaming file upload for POST & PUT functions.
					Add header logging.
					Corrected POST function using application/json-patch+json media type.
					adjusted multi-part file upload.
					updated software libraries. 

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

