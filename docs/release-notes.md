# Release Notes Webservices 21.1.2

## General

This version of the connector requiers OpCon version STS 20.7 or LTS 21.0 or greater.

### New Features

**CONNUTIL-560**    
					Added PowerShell version 7, Azure CLI and SMA Azure Storage Connector to container.   

## Migration Considerations

All three items are automatically installed when container is created.
To use Azure CLI, authentication must first be sorted out.
To use Azure Storage Connector see the Azure Storage Connector documentation.

### Fixes

**CONNUTIL-577**    
					Variable values not updated correctly if the message body is read from an input file.   
				




