# Operation

The connector supports various content-types such as application/json, application/xml, application/x-www-form-urlencoded and text/plain. 
The connector is unaware of specific objects and works generically with all JSON and XML data. 

## Response Parsing
Response parsing in the returned data is supported for application/json using **JSONPath** and application/xml using **XPath**. Header parsing is also supported using the header attribute name. When defining jobs, the best approach is to run the connector in Debug mode as this dumps the data received from the web server. The job output can then be examined for the returned data so the JSONPath, XPath or header parsing syntax can be determined.

### JSONPath 
Every JSON object is composed of an inherit hierarchy and structure. All JSON ends up creating a tree of nodes where each node is a JSON Element. These can be a single element or an array of elements. JSONPath provides a standard syntax to define different parts of a JSON document. It defines expressions to traverse through a JSON document.

In the following example:
```
{
  “node”:{
    “node1”:{
     [
      {“node3”:”value3”},
      {“node4”:”value4”}
   ],
  }
}
}
```
The JSONPath syntax to extract the value of attribute node3 is: **$.node.node1.[0].node3** 
- $ indicates root node operator
- node is the name of a JSON node
- node1 is the name of a JSON node within the first JSON node
- [0] indicates the first record of the JSONARRAY following the JSON node node1
- node3 is the required attribute to extract the value from

In the following example:
```
{
  “node”:{
    “node1”:{
     [
      {“nodeType”:”Type3”},
      {“nodeType”:”Type4”}
     ],
    }
  }
}
```
The JSONPath syntax to scan for the value of an attribute nodeType within a JSONARRY is: **$.node.node1.[*].nodeType** 
- $ indicates root node operator
- node is the name of a JSON node
- node1 is the name of a JSON node within the first JSON node
- [*] indicates the scan all records of the JSONARRAY following the JSON node node1

### XPath 
XPath stands for XML Path Language. XPath uses ‘path like’ syntax to identify and navigate nodes in an XML document.

In the following example:
```
{<?xml version="1.0" encoding="UTF-8"?>
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
</issue>)
```
The XPath syntax to extract the value of element id **//issue/id/text()** 
- // indicates root element operator
- issue is the name of an element
- issue id the name of an element that is required.
- text() indicates that we want the value of the id element.

The XPath syntax to extract the value of attribute name of element author **//issue/author/@name** 
- // indicates root element operator
- issue is the name of an element
- author is the name of an element that is required.
- @ indicates we require an attribute value.
- name is the name of the attribute of the element.

### Text Strings
When requesting the value of variables associated with VisualCron jobs, a single text string is returned instead of a JSON structure. Using the **TEXTSTRING** value will insert the complete returned result into the associated token.  

### Header Parsing 
It is possible to parse the returned header for attribute values by specifying the attribute name.
The parsing syntax is prefixed by a hash (#). 
The header syntax to retrieve the returned data type is **#Content-Type**
- \# indicates this is a header parsing definition
- Content-Type is header attribute to extract the value from.

## Variables
When working with variables, all variables must be defined as @\<***name**\> as the @ indicates to the connector that this definition is a variable meaning these values are available to the defined steps. Global variables are available to all steps while response variables are available to all subsequent steps.

When working with Basic mode authentication, **@User** and **@Password** variables should be added and the **Authorization Basic** header attribute must be included in the request of each step. The connector will detect the ‘Basic’ mode and perform the required Base64 encoding of ‘@User:@Password’ string and add it to the header attribute.

When working with Windows Authentication (NTLM), **@User**, **@Password** and **@Domain** variables should be added and the **Authorization NTLM**  header attribute must be included in the request of each step. The connector will detect the ‘NTLM’ mode and create the required credentials using the @User, @Password and @Domain values.

When working with client certificates, **@CertStore**, **@CertStorePwd** and **@CertStoreType** variables should be added and the **Authorization CERT** header attribute must be included in the request of each step. The connector will detect the ‘CERT’ mode and create the required credentials extracting the certificate from the named keystore (@CertStore value).

When working with SOAP Webservices the **Message-Type=SOAP** value must be included in the attribute header for each step and the Content-Type **text/xml** should be selected.

## Environment variables 
The connector supports the retrieval of Environment Variables during start-up. During this process the connector extracts all Environment variables starting with a @ and adds them to the list of variables available for value substitution. As the Environment Variables are not passed to the agent as part of the JSON definition, they are not subject to JSON parsing. This means that normal OpCon property resolution is supported. The associated Windows or Linux agent must support this feature as the agent will register the Environment Variables associated with each job execution. 

## Special Variables
The following special variables are currently defined within the connector.

Variable | Description
-------------------- | -----------
**@User**            | Is used to define the name of the user associated with the requests. 
**@Password**        | Is used to define the password of the user. Encrypted global properties can be used.
**@Domain**          | Is used to define the domain associated with the user when using Windows Authentication to IIS.
**@CertStore**       | Is used to define the location of the keystore when client certificates are used.
**@CertStorePwd**    | Is used to define the password of the keystore. Encrypted global properties can be used.
**@CertStoreType**   | Is used to define the format of the client key in the keystore. Currently PKCS12 is the only supported format.
**@JCorrelationid**	 | Is used to create the unique id of a job in the daily tables that can be used during call back procedures.
**@I_\<name\>**	     | is used to indicate to the connector that the variable is an integer value.

### @JCorrelationid
If there is a requirement to start a process in a web server and then allow the web server to return a completion indication, the **@JCorrellationid** value can be used. The value corresponds to the next job in the processing sequence. The job itself should have a Threshold dependency that is never triggered. The reason that the job should be held on a dependency instead of an ‘On Hold’ condition, is that any monitoring to determine if the job is ‘Late to Start’ will not valid if the job is in an ‘On Hold’ condition. The web server should respond with a /api/jobactions request using the unique jobid and setting the job status to either ‘markFinishedOk’ or ‘markFailed’ depending on the result processing in the web server.

The value of the @JCorrellationid ‘[[SCHEDULE DATE-YYYY-MM-DD]].[[$SCHEDULE NAME]].JOB001’ should point to a valid job in the daily. The schedule date values used is a special format consisting of YYYY-MM-DD . (a $SCHEDULE DATE property should be created to provide this value). 

During the connector processing, the connector will connect to the host OpCon system using the OpCon-API and retrieve the unique job id. This value can then be passed as part of the JSON payload when starting the process on the web server.

## @I_\<name\> Integer Variables
The @I_ indicates that the defined variable contains an integer value. When using properties and variables within the JSON payload, the names are defined within the JSON payload as string values. When the value is substituted even if the value is an integer value it will be inserted into the JSON payload as a string value due to the place holder being a string value (i.e. “fixno” : “[[version]]” will result in “fixno” : “122”). When this is defined as an @I_ variable, the value will be inserted into the JSON payload as an integer value (i.e. “fixno” : “@I_version” will result in “fixno” : 122). 
To implement this:
- Create a global variable @I_verion with a value of [[version]].
- In the JSON payload use the @I_version variable value where the [[version]] property should be inserted.

## Using x-www-form-urlencoded content-type
The x-www-form-urlencoded content-type is used when authenticating requirements use OAuth2 endpoints. When defining the data in the request body the values are entered as name=value pairs separated by an ampersand. 

Example:
```
grant_type=client_credentials&client_id=@Clientid&client_secret=@Key&resource=https://management.azure.com/
```
## Templates
Templates can be used to import task definitions when creating the tasks. The templates can be used either for Windows or Linux tasks. Templates for job definitions can be found in the SMA Technologies Innovation Lab on GitHub. Projects have been assigned names associating themselves with templates (i.e. \<***name***\>-webservicestemplate).

## Proxy Servers
The connector supports the use of proxy servers. When requiring a proxy server, it is possible to configure this in two possible ways. If a proxy server definition exists in both the configuration and the Step definition, the value in the Step definition will be used.

### All jobs using Proxy Server
If all jobs associated with the connector require a proxy server, then the proxy server can be configured in the Connector.config. See the SMAWSConnector Configuration section for more information.

```
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

### Specific Steps within a Job using a Proxy Server
This approach allows a proxy server to be associated with a specific step within a job, by entering the proxy server information in the **Proxy Server** field of the **Step** definition. If there is a requirement to support multiple proxy servers or a mixture of proxy server jobs and non-proxy server jobs, use this option. 

## Client Certificates
When using client certificates, the certificate needs to be inserted into a keystore that is then associated with the connector. The suggestion is to create a store directory off the connector root installation directory and create a unique keystore for each certificate.

When creating the keystore, download the client key \<***keyname***\>.p12 and use the Java program keytools (location in the \java\bin directory) to create the keystore. When you receive a client key, you will also receive a password that will allow you to execute a request on the received key. To create the keystore for a received client key test_client.p12 enter the following:

```
keytool -importkeystore -srckeystore c:\wsconnector\store\test-client.p12 -destkeystore c:\wsconnector\store\test.pkcs12 -srcstoretype pkcs12
where:  -importkeystore     indicates create the keystore
        -srckeystore        the received certificate (format p12)
        -destkeystore       the name of the keystore to create inserting the received Certificate into the keystore
        -srcstoretype       the format of the received key (only pkcs12 supported)
```
During the process, you will be asked to create a password for keystore as well as enter the password you received with the certificate file. The -destkeystore value should be set in the **@CertStore** variable and the keystore password you entered when creating the keystore should be set in the **@CertStorePwd** variable.  

### WebServices Jobs as Embedded Scripts
It is possible to define WebServices jobs using Embedded Scripts. The templates section includes two definitions (WebServices-vcron-vars and WebServices-vcron-no-vars) that can be used to execute VisualCron jobs passing the unique job definitions as Environment Variables.
When defining WebServices jobs as Embedded Script jobs, the Working Dir value must point to the WebServices installation directory.

Script Name                | Description
-------------------------- | -------------------------------------------------------------------------------
WebServices-vcron-vars     | includes variable definitions that are passed to the VisualCron job as VisualCron Job variables.
Webservices-vcron-no-vars  | no variables are passed to the VisualCron job.

The following Environment Variables should be set for each execution request:

Environment Variable | Description
-------------------- | -------------------------------------------------------------------------------
@Url                 | the url used to connect to the VisualCron Rest-API (localhost:8001).
@User                | the VisualCron user that will be used to execute the VisualCron job.
@Password            | the password of the VisualCron user.
@Jobname             | the name of the VisualCron job.
@Variables           | optional. required for WebServices-vcron-vars script and consists of the VisualCron job variable name and the value. Multiple variables are separated using the pipe (\|) character (varName1=value\|varName2=value).

### Installing WebServices Connector as an Embedded Script
To install the WebServices connector as an embedded script requires the definition of a script type, a script runner type and the two supplied scripts WebServices-vcron-vars and WebServices-vcron-no-vars.

#### Script Type
Select **Scripts.Types** and select the **Add** icon.
for **Name** enter **WebServices**
for **File Extension** enter **.web**
for **Description** enter **Running WebServices as an embedded script**

#### Script Runner
Select **Scripts.Runners** and select the **Add** icon.
for **OS** select **Windows**
for **Type of Script** enter **WebServices**
for **Command Template** enter **install-dir\SMAWSCOnnector.exe $FILE $ARGUMENTS**

#### Install supplied scripts
Select **Scripts.Repository** and select the **Add** icon.
for **Name** enter **WebServices-vcron-vars**
for **Description** enter **WebServices script to call VisualCron passing variables**
for **Script** paste the WebServices-vcron-vars definition from the templates section
for **Type** select **WebServices**

Select **Scripts.Repository** and select the **Add** icon.
for **Name** enter **WebServices-vcron-no-vars**
for **Description** enter **WebServices script to call VisualCron without variables**
for **Script** paste the WebServices-vcron-no-vars definition from the templates section
for **Type** select **WebServices**

