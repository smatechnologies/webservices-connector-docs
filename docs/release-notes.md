# Release Notes Webservices 21.1.2

## General

This version of the connector requires OpCon version STS 20.7 or LTS 21.0 or greater.

### New Features

### Fixes

**CONNUTIL-577**    
					Fixed a problem Token replacement not working correctly when the information for a POST or PUT request is containing in a file. Corrected the token replacement when checking input files.   
				
**CONNUTIL-613**    
					Fixed an issue where the web services connector communicating with the OpCon Rest API returned an error, if any fields in the response from the API were changed from previous versions, as happened in OpCon version 22.7.0.


