# Defining Jobs

When defining a SMAWSConnector job, select a Windows or Unix / Linux job type and then select the **Web Services** job sub-type. 

The job sub-type requires the selection of an OpCon batch user that will be used to execute the connector. The Connector location consists of a global property that contains the root installation directory of the connector software.
 
The job sub-type has a template id value that can be used to uniquely identify the template used when creating the job definition. A job template can be loaded by selecting the Import Template button which will allow you to browse for a template. Once a template has been selected and imported, the information will be seen in the fields. The values of Variables and Environment Variables will be presented as ??????. Once a job definition is complete, it is possible to save this as a template by selecting the Export Template button. 

The job definition has three main tabs that are used to complete the definition. 

## Global Values Tab 
This tab supports the definition of **Variables**, **Environment Variables** and **Property Update on Completion** values. Variables and Environment Variables are used as arguments to update place holders in JSON or XML data and URLs, while Property update on Completion consists of property names that will be updated with the associated values that will be written into the defined OpCon property when all the steps have completed successfully. 

### Variables Tab
The tab supports the definition of global variables and properties. Global variables are used as arguments to update place holders in JSON data and URLs, while properties consist of property names and variable names. The contents of the variable will be written into the OpCon property when all steps have completed successfully.

Variables are used to define global variables that can be used during the associated **Steps**. These variables are prefixed with a **@**. The variables are then used as arguments that are substituted in the URL definition or the JSON data with the associated values and then submitted as part of POST or PUT requests. 

Examples are providing a URL value **@Url** value and user **@User**, password **@Password** and domain **@Domian** values during authentication. The @Url value is replaced in the URLs while the @User and @Password values are replaced in the JSON data message body.

Variables are defined using a **@** as the first character followed by a name in the Name field and the associated value in the Value field. 

When using OpCon properties to provide the associated value, it should be remembered that the OpCon property value will be resolved by OpCon during task start-up and substituted in the json task definition which is being submitted to the connector. If the OpCon property contains a Windows path definition, it will create problems with JSON parsing as the Windows path definition has not been escaped (i.e. \\server\directory) .In such a case, Environment Variables should be used as these values are not part of the JSON task definition. 

Examples:
 
URL replacement
```
@url = server:9010

https://@Url/api/tokens -> https://server:9030/api/tokens

```
JSON Message Body replacement
```
@User = test
@Password = [[testpwd]] (an encrypted property)

 {
  "id":null,
  "user":
  {
    "id":-1,
    "loginName":"@User",    -> "loginName":"test"
    "password":"@Password"  -> "password":"encrypted property definition"  (this will be decrypted by the target OpCon agent)
  },
  "tokenType":
  {
    "id":null,
    "type":"User"
  }
} 
```

In the above examples, the @Url variable value will be substituted in the URL and the @User and @Password variable values would be substituted into the defined JSON data.
The user code and password values could be placed in encrypted global tokens. However, it should be noted that the Windows Agent where the connector installed, must support this functionality.

### Environment Variables Tab
This is used to define variables that can be used during the steps. These variables are prefixed with a @. They can be used as arguments that are substituted in the URL definition or the JSON or XML data submitted as part of POST or PUT requests. 

Examples are providing URL value @Url value, user code (@User) and password (@Password) values during authentication and OpCon property values @MAIN_PATH ([[SI.MAIN_PATH]]). The @Url value is replaced in the URLs while the user and password values are inserted into the JSON data replacing the @User and @Password values with the real values. The @MAIN_PATH value is replaced with the contents of the OpCon property [[SI.MAIN_PATH]]. 

The Environment Variables option provides a way for OpCon properties that contain JSON parsing problems to be used by the connector as the Environment Variables are not part of the OpCon task definition but are passed as Environment Variables to the associated agent. The agent loads the environment variables for the task execution making them available for retrieval by the connector. 

Environment Variables are entered by defining them using a @ as the first character followed by a name in the Name field and the associated value in the Value field. 
Examples:
 
```
@MAIN_PATH = [[SI.MAIN_PATH]]

File Name replacement
 
@MAIN_PATH\jfiles\login.json 
```

The @MAIN_PATH value will be replaced with the [[SI.MAIN_PATH]] value. However, it should be noted that the agent where the connector installed, must support the Environment Variables feature.

### Property Update on Completion
OpCon Properties can be used to save values extracted from the returned jJSON or xml data for subsequent task execution when the current task completed successfully. The values can be saved in OpCon global, schedule instance or job instance properties. It should be noted that the use of a dot (.) in the schedule or job name will cause problems with the property resolution. 

Example:
```
SI.Version.[[$SCHEDULE DATE-YYYY-MM-DD]].API_EXP_TEST_CJOB001[SUB_API_EXP_TEST] = @Version
```

In the above example, the contents of the @Version variable will be inserted into the OpCon schedule instance property Version of the sub-schedule API_EXP_TEST_CJOB001[SUB_API_EXP_TEST] in the daily table for the date associated with [[$SCHEDULE DATE-YYYY-MM-DD]] value. When defining instance properties as the destination, a special date property should be created and used as the OpCon-API expects date values in the yyyy-mm-dd format. 

## Steps Tab
The Steps tab is used to define the actions that the connector will perform. These steps are performed in sequence from first to last. Each time a step in created, a new Tab is created with the header Step(n) where n is the Step number. Each step definition consists of a URL and associated Request and Response definitions.

To create a Step, enter the required data and then select the **Add Step** button. This will create the step naming it Step(n). To create subsequent steps, select the **+** tab, enter the required data and once again select the **Add Step** button. To remove a step, select the step and then select the **Remove Step** button. The step will be removed and the Step(n) values will be adjusted.
 
Select the function (**GET, POST,PUT, DELETE**) from the drop-down list and enter the full URL of the web server request including http or https as well as address and port. The connector will use a TLS connection if it detects that the URL contains https.

If this step requires a proxy server, enter the full proxy server URL in the **Proxy Server** field. If the Proxy server is activated in the Connector.config, the value in the **Proxy Server** field will override it.

### Request Information Tab
The Request information consists of selecting the Content Type that will be submitted from the drop-down list, defining header attributes that will be inserted into the header of the request and the associated payload should the request be a POST or a PUT. 

#### Header Tab 
Header attributes are provided by using the **Header** tab and entering the header attribute name in the Attribute Name field and the associated value in the Attribute Value field. Different authentication methods can be provided for each step including the appropriate header definitions. It is possible to use variable values previously defined or created by a previous step. 

Attribute Name | Attribute Value
-------------- | -----------
***name***     | ***value***
***name***     | @variable

##### Basic Authentication
When using Basic Authentication is required, a header Attribute Name of **Authorization** and a Attribute Value of **Basic** must be defined. The connector detects this and uses the @User and @Password variable values to generate the required credentials by performing a Base64 encoding of the ‘@User:@Password’ string and insert this it to the value of the header attribute.
Requires that the @User and @Password variables are defined.

Attribute Name | Attribute Value
-------------- | -----------
Authorization  | Basic

##### Windows Authentication for IIS
When using Windows Authentication to IIS, a header Attribute Name of **Authorization** and a Attribute Value of **NTLM** must be defined. The connector detects this and uses the @User, @Password and @Domain variable values to generate the required credentials for the connection. Requires that the @User, @Password and @Domain variables are defined.

Attribute Name | Attribute Value
-------------- | -----------
Authorization  | NTLM

##### Token Authentication
When using Token Authentication, a header Attribute Name of **Authorization** and a Attribute Value which is dependent on the receipient application must be used. The token is usually retrieved in a previous step and saved in a dynamic variable (@Token) which is then used in then used in the subsequent requests. For authentication to the OpCon REST-API, the Attribute value is Token \<***token***\>. 

Attribute Name | Attribute Value
-------------- | -----------
Authorization  | Token @Token

##### Certificate Authentication
When working with client certificates, a header Attribute Name of **Authorization** and a Attribute Value of **CERT** must be defined. The connector detects this and uses the @CertStore variable value to create the required credentials extracting the certificate from the named keystore. Requires that the @CertStore, @CertStorePwd and @CertStoreType variables are defined.

Attribute Name | Attribute Value
-------------- | -----------
Authorization  | CERT

##### Using SOAP 
When working with SOAP Webservices, a header Attribute Name of **Message-Type** and a Attribute Value of **SOAP** must be definedand the Content-Type text/xml should be selected.

#### Body Tab
The payload can be entered using the Body Tab. 

##### FileName
The payload can be provided by entering the full path of the filename containing the data in the **File Name** field. It should be noted that if a filename is used, the location of the file should be available on the system where the connector is installed.

##### Message Body
The payload can be entered directly into the **Message Body** field.
 
## Response Information Tab
The Response information consists of selecting the Content Type to be returned from the drop-down list, defining Response Variables that are used to extract attribute values from the returned data and setting step completion information.

### Response Variable Management
This provides a mechanism to extract information in the returned payload and save the extracted information into a variable so that subsequent steps can use the information. Currently attribute extraction is only supported when the Content Type is application/json or application/xml.

When defining a response variable give it a unique name starting with a (@) and then define the attribute name in the JSON or XML data to extract the value for. The syntax uses standard JSONPath formats which is a query language for JSON and standard XPath formats which is a query language for XML. If the attribute is not found, the connector will terminate with an appropriate message and an error code of 1.

Attribute Name | Attribute Value
-------------- | -----------
@Token         | $.id
@Value         | //issue/id/text()
 
#### Response Variable Management 
The Step completion information defines what to do when the step completes. 
It requires setting a HTTP return code value in the **Completion Code** which is then checked after the step completes. If the step retun code does not match the defined value, the workflow will stop, returning the failed code. It is possible to bypass the **Completion Code** check by selecting the **Ignore Result** field. 

It is also possible to check the contents of the returned JSON or XML data to determine if the processing was successful as web server return code indicates that the function you requested completed successfully or failed. The connector will check the contents of the attribute defined in the **Attribute to Check** field for a match for values contained in the **Good Finish** and **Bad Finish** fields. Multiple values can be entered in these fields by separating each value with a forward slash (/).

It is possible to scan through a JSONARRAY looking for a specific value in the records. When using this capability, use a star (*) instead of a numeric value. The star value indicates that all records in the array should be searched until a match is found (this capability is not supported for polling).
 
It is also possible to perform a ‘poll’ performing subsequent requests until a match is found. This function can be enabled by selected the **Poll** field and entering the polling requirements. The connector will use the values (defined in seconds) in the **Delay** and **Interval** fields to determine the polling frequency. The **Delay** field defines how long to wait before the first poll request and the **Interval** field defines the time to wait between subsequent request . The **Max Time** value (defined in minutes) represents the maximum time to check for a valid response. The connector returns a 408 code if a **Bad Finish** is detected or a 460 if the **Max Time** value expires before a valid result is detected (**Good Finish** or **Bad Finish**).

#### Output File
It is also possible to write the returned data to an output file by entering a filename in the **Output File** field. It should be noted that if a filename is used, the location of the file should be available to the system the connector is installed on.

## Failure Criteria Tab
The Failure Criteria tab is used to define result of the defined workflow. The result should be the completion code of the final step.
 
The connector returns the following codes:

Code     | Description
---------| -----------
1        | When an initialization error or an initial connection errors occurs.
400-599	 | Standard HTTP errors
408	     | When the maximum wait time expires during a poll request sequence
460	     | When the error condition is returned during a poll request sequence
 
## Logging and Job Output
The default logging implemented by the connector consists of a maximum cycle of five log files. The log files contain information about the connector and any jobs run by the connector. The log files (webservices.log) can be found in the \<***installation root***\>\\log directory. Information is appended into the log files and any error messages, return codes can be viewed in these log files.

All information produced by the OpCon job is available in the job output and can be retrieved using the OpCon JORS capability.

