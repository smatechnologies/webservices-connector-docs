# Release Notes WebServices 21.2

## General

This version of the connector requires OpCon version STS 20.7 or LTS 21.0 or greater.

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

