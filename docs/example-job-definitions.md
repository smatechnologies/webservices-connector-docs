# Example Job Definitions
This sections provides sample definitions showing the various capabiities of teh Webservices connector.

## Using Token Authentication (multiple Steps)
Update the value of a global property in the OpCon system. 

The OpCon system uses token authentication and the first step is to obtain a valid token from the OpCon system that will be used for the remaining steps. Subsequent steps will retrieve the information about a global property and update the contents of the property. 

In the **Variables** section of **Global Values** tab, enter the @User and @Password variables that will provide the user and password information used during authentication request (the password can be an encrypted global property). Enter the name of the global property using the @GlobalProperty variable. To increase portability when sharing templates, enter the web server address using the @Url variable.

Variable Name   | Variable Value
--------------- | -----------
@Url            | SERVER1:9010
@User           | user
@Password       | [[user_pwd]]
@GlobalProperty | PROPTEST

In the **Step** tab, enter the following:

```
1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a POST request to obtain an authentication token.
    a.	Select the POST function.
    b.	Enter the full URL https://@Url/api/tokens (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Body tab.
        iii. In the message body, enter the JSON data to be submitted as part of the POST request using the @User and @Password global variables. While processing the request, the 
             connector will substitute the variable names in the JSON definition with the required values.

            {
                "id":null,
                "user":
                {
                    "id":-1,
                    "loginName":"@User",
                    "password":"@Password"
                },
                "tokenType":
                {
                    "id":null,
                    "type":"User"
                }
            } 
 
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Response Variable Management tab.
        iii. The request will return the authentication token and we need to extract the value and place it in a variable that will be used by the subsequent steps. In the Attribute
             section add @Token into the Name field and $.id into the value field which will extract the first id attribute returned in the JSON data and save it in the variable @Token. 
             The $.id is a JSONPath definition indicating the attribute value to extract from the JSON data. The ($) indicates start from the root of the JSON structure and the (.id) 
             indicates get the value of the first id attribute in the JSON data.
        iv.	 Select the Step Completion tab.
        v.	 Set the Step completion code to 200.
    e.	Press the Add Step button.
2.	Step 2 (Select the (+) definition).
    The second Step function is a GET request to obtain the global property defined in the @GlobalProperty variable.
    a.	Select the GET function.
    b.	Enter the full URL https://@Url/api/globalproperties?name=@GlobalProperty
    c.	Select the Request tab
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Header tab.
        iii. In the Attributes section indicate the authentication token. The authorization value is defined by the web server and the definition for an OpCon system is ‘Token <token 
             value>’. Enter an attribute consisting of Authorization into the Name field and Token @Token into the Value field. 
        iv.	 The @Token variable value extracted during the previous step (Step1) will be inserted into header attributes by the connector.
    d.	Select the Response tab
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Response Variable tab.
        iii. The request will return the property id and we need to extract the value and place it in a variable that will be used by the subsequent steps. In the Attribute section enter a 
             value of @Propertyid into the Name field and $[0].id into the Value field. In this case, the returned JSON data consists of array containing global property definitions. The
             ([0]) indicates that the first record of the array is required and the (.id) indicates that the value of the first id attribute is required. 
        iv.	 Select the Step Completion tab.
        v.	 Set the Step completion code to 200
        e.	 Press the Add Step button.
3.	Step 3 (Select the (+) definition).
    The third Step function is a PUT request to update the global property defined in the @GlobalProperty variable.
    a.	Select the PUT function.
    b.	Enter the full URL https://@Url/api/globalproperties/@Propertyid
        The @Propertyid will be replaced in the URL with the value of the @Propertyid variable value extracted during the previous step (Step2) the connector. This shows how information
        extracted from a previous step can be used when creating URLs.
    c.	Select the Request tab
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Header tab.
        iii. In the Attributes section indicate the authentication token. The authorization value is defined by the web server and the definition for an OpCon system is ‘Token <token 
             value>’. Enter an attribute consisting of Authorization into the Name field and Token @Token into the Value field. The @Token variable value extracted during the previous step
             (Step1) will be inserted into the attribute value by the connector. 
        iv.	Select the Body tab.
        v.	In the message body, enter the JSON data to be submitted as part of the PUT request to update the property value.

            {
                "id":"@Propertyid",
                "name":"@GlobalProperty",
                "value":"wsrest 1234 test value from UNIX 2"
            }

            Notice that the @Propertyid is also replaced in the JSON data by the Connector as the JSON data requires the id of the property that is being updated. The property name is 
            taken from the @GlobalProperty variable.
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Step Completion TAB.
        iii. Set the Step completion code to 200
    e.	Press the Add Step button.

```
## Using Poll capability to track execution (multiple Steps)
Perform a schedule build on the OpCon system tracking to see that the Schedule Build completes successfully. 

The OpCon system uses token authentication and the first step is to obtain a valid token that will be used for the remaining steps.

In the **Variables** section of **Global Values** tab, enter the @User and @Password variables that will provide the user and password information used during authentication (the password can be an encrypted global property). Enter the variable @ScheduleName that contains the name of the schedule to build and the variable @ScheduleDate containing the date to build the schedule on (format should be yyyy-mm-dd). To increase portability when sharing templates, enter the web server address using the @Url variable.
 
Variable Name   | Variable Value
--------------- | -----------
@Url            | SERVER1:9010
@User           | user
@Password       | [[user_pwd]]
@ScheduleName   | SCHED001
@ScheduleDate   | [[$SCHEDULE DATE-YYYY-MM-DD]]

In the **Step** tab, enter the following:

```
1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a POST request to obtain an authentication token.
    a.	Select the POST function.
    b.	Enter the full URL https://@Url/api/tokens (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Body tab.
        iii. In the message body, enter the JSON data to be submitted as part of the POST request using the @User and @Password global variables. While processing the request, the
             connector will substitute the variable names in the JSON definition with the required values.

            {
                "id":null,
                "user":
                {
                    "id":-1,
                    "loginName":"@User",
                    "password":"@Password"
                },
                "tokenType":
                {
                    "id":null,
                    "type":"User"
                }
            } 

    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Response Variable Management tab. 
        iii. The request will return the authentication token and we need to extract the value and place it in a variable that will be used by the subsequent steps. In the Attribute 
             section add @Token into the Name field and $.id into the value field which will extract the first id attribute returned in the JSON data and save it in the variable @Token. 
             The $.id is a JSONPath definition indicating the attribute value to extract from the JSON data. The ($) indicates start from the root of the JSON structure and the (.id) 
             indicates get the value of the first id attribute in the JSON data.
        iv.	 Select the Step Completion tab.
        v.	 Set the Step completion code to 200.
    e.	Press the Add Step button.
2.	Step 2 (Select the (+) definition).
    The second Step second Step function is a POST request to perform a schedule build.
    a.	Select the POST function.
    b.	Enter the full URL https://@Url/api/schedulebuilds
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Header tab.
        iii. In the Attributes section indicate the authentication token. The authorization value is defined by the web server and the definition for an OpCon system is ‘Token <token 
             value>’. Enter an attribute consisting of Authorization into the Name field and Token @Token into the Value field. The @Token variable value extracted during the previous step 
             (Step1) will be inserted into the attribute value by the connector. 
        iv.	Select the Body tab.
        v.	In the message body, enter the JSON data to be submitted as part of the POST request using the @ScheduleName and @ScheduleDate variables. While processing the request, the 
            connector will substitute the variable names in the JSON definition with the required values. 

            {
                "schedules":[
                    {"id":null,"name":"@ScheduleName"}
                ],
                "id":null,
                "expiryTime":null,
                "dates":[
                    "@ScheduleDate"
                ],
                "logFile":null,
                "overwrite":true,
                "hold":true
            }
 
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Response Variable Management tab.
        iii. The request will return information about the schedule build and we need the schedule build id to determine if the build completed successfully in subsequent steps. In the 
             Attribute section add @Buildid into the Name field and $.id into the value field which will extract the first id attribute returned in the JSON data and save it in the 
             variable @Buildid. The $.id is a JSONPath definition indicating the attribute value to extract from the JSON data. The ($) indicates start from the root of the JSON structure 
             and the (.id) indicates get the value of the first id attribute in the JSON data.
        iv.	 Select the Step Completion tab.
        v.	 Set the Step completion code to 201.
    e.	Press the Add Step button.
3.	Step 3 (Select the (+) definition).
    The third Step function is a GET request to determine the status of the build request. A poll is performed until a completion result is determined by checking the contents of the
    returned JSON data for a specific attribute value
    a.	Select the GET function.
    b.	Enter the full URL https://@Url/api/schedulebuilds/@Buildid
        The @Buildid will be replaced in the URL with the value of the @Buildid variable by the connector. This shows how information extracted from a previous step can be used when 
        creating URLs.
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Header tab.
        iii. In the Attributes section indicate the authentication token. The authorization value is defined by the web server and the definition for an OpCon system is ‘Token <token 
             value>’. Enter an attribute consisting of Authorization into the Name field and Token @Token into the Value field. The @Token variable value extracted during the previous step 
             (Step1) will be inserted into the attribute value by the connector. 
        iv.	 Select the Body tab.
    d.	Select the Response tab.
        The GET request will poll the server several times to get the completion status of the build request. The Check Returned Data and Poll values must be completed.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Step Completion tab.
        iii. The Good Finish field define the values associated with a successful build. Enter success/Completed meaning either a value of success or Completed will indicate a successful 
             build.
        iv.	 The Bad Finish field define the values associated with an unsuccessful build. Enter Failed meaning a value of Failed will indicate an unsuccessful build.
        v.	 Set the Attribute To Check field to the attribute that contains the value to check using JSONPath format. $[0].message indicates that the attribute message of the first record 
             in the array should be checked.
        vi.	 Select the Poll indicator and enter the poll delay and interval values in seconds. The delay value is how long to wait before the first check and the interval value is how 
             long to wait for subsequent checks.
        vii. By setting the Max Time to 5 minutes, the connector will check for a positive response for 5 minutes before returning a 408 (timeout error).
        viii. A return code of 460 will be sent if the Bad Finish value is received.
e.	Press the Add Step button.

```
## OAuth2 Authentication using x-www-form-urlencoded
Perform an OAuth2 authentication using x-www-form-urlencoded Content Type. When defining the data in the request body the values are entered as name=value pairs separated by an amphisan. 

In the **Variables** section of **Global Values** tab, enter the @Tenantid, @Clientid, @Key and @Subscription variables that will provide the information used during authentication (these values should all be placed in encrypted properties). To increase portability when sharing templates, enter the web server address using the @Url variable.

Variable Name   | Variable Value
--------------- | -----------
@Url            | login.microsoftonline.com
@Tenantid       | [[AZURE_TENANT_ID]]
@Clientid       | [[AZURE_CLIENT_ID]]
@Key            | [[AZURE_KEY]]
@Subscription   | [[AZURE_SUBSCRIPTION]]

In the **Step** tab, enter the following:

```
1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a POST request to obtain an authentication token.
    a.	Select the POST function.
    b.	Enter the full URL https://@Url/@Tenantid/oauth2/token (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/x-www-form-urlencoded from the Content Type drop-down list.
        ii.	 Select the Body tab.
        iii. In the message body, enter the data to be submitted as part of the POST request using the @Clientid and @Key global variables. The values are entered in name=value pairs using 
             the amphisan (&) character as the delimiter. While processing the request, the connector will substitute the variable names in the data with the required values.

            grant_type=client_credentials&client_id=@Clientid&client_secret=@Key&resource=https://management.azure.com/
 
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Response Variable Management tab.
        iii. The request will return the authentication token and we need to extract the value and place it in a variable that will be used by the subsequent steps. In the Attribute 
             section add @Token into the Name field and $access_token into the value field which will extract the first id attribute returned in the JSON data and save it in the variable 
             @Token. The $.iaccess_token is a JSONPath definition indicating the attribute value to extract from the JSON data. The ($) indicates start from the root of the JSON structure 
             and the (.access_token) indicates get the value of the first access-token attribute in the JSON data. 
        iv.	 Select the Step Completion TAB.
        v.   Set the Step completion code to 200.
    e.	Press the Add Step button.
 
```
## Basic Mode Authentication
Execute a request on a Web Server using Basic authentication.

In the **Variables** section of the **Global variables** tab,  enter the @User and @Password variables that will provide the information used during authentication (the password value should be placed in an encrypted property). To increase portability when sharing templates, enter the web server address using the @Url variable.
 
Variable Name   | Variable Value
--------------- | -----------
@Url            | easyredmine.com
@User           | USER001
@Password       | [[USER001_PWD]]

In the **Step** tab, enter the following:

```
1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a GET request to retrieve information from the Web Server.
    a.	Select the GET function.
    b.	Enter the full URL https://@Url/issues/182.xml (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/xml from the Content Type drop-down list.
        ii.	 Select the Header TAB.
        iii. In the Attributes section indicate basic authentication will be used. Enter an attribute consisting of Authorization into the Name field and Basic into the Value field. The 
        Connector will detect that Basic authentication is required and perform the required Base64 encoding of the ‘@User:@Password’ string and append this to the header attribute (Basic 
        @User:@Password). 
    d.	Select the Response tab
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Step Completion tab.
        iii. Set the Step completion code to 200.
    e.	Press the Add Step button.
 
```
## Using Environment Variables 
Execute a request on a Web Server using Environment Variables. 

When passing information from OpCon to the connector, the task definition is stored in JSON format in the CommandLine field in a Windows definition and in the Parameters field in a UNIX definition. During task definition, the stored JSON definition is created by a JSON parser meaning all values are properly escaped. When OpCon properties are contained within the created JSON definition, these values are only resolved at task start time. This means that it is possible for unescaped values to be inserted into the JSON definition causing json parsing problems in the connector. 

If such a case exists, Environment Variables should be used to pass the OpCon property values as Environment Variables are not passed to the connector as part of the JSON definition, but is separate fields and are loaded into the task execution environment by the agent. The connector checks for Environment Variables during start-up and loads these values into variables that can be used within defined steps. 

In the **Variables** section of **Global Values** tab, enter the @User and @Password variables that will provide the user and password information used during authenticationTo increase portability when sharing templates, enter the web server address using the @Url variable.
 
Variable Name   | Variable Value
--------------- | -----------
@Url            | SERVER1:9010
@User           | user
@Password       | [[user_pwd]]

In the **Environment Variables** section of **Global Values**, enter the **@MAIN_PATH** value which points to a schedule instance property associated with the schedule called SI.MAIN_PATH. The schedule instance property contains the directory where the required files can be found (i.e. c:\production\main). 

Variable Name   | Variable Value
--------------- | -----------
@MAIN_PATH      | [[SI.MAIN_PATH]]

In the **Step** tab, enter the following:

```
1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a POST request to obtain an authentication token.
    a.	Select the POST function.
    b.	Enter the full URL https://@Url/api/tokens (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Body tab.
        iii. In the File Name field enter the name of the full filename (@MAIN_PATH\wsfiles\login.json) that contains the login information using the Environment Variable name to provide 
        the path information. While processing the request, the connector will substitute the variable names in the JSON definition with the required values.
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Response Variable Management tab.
        iii. The request will return the authentication token and we need to extract the value and place it in a variable that will be used by the subsequent steps. In the Attribute 
             section add @Token into the Name field and $.id into the value field which will extract the first id attribute returned in the JSON data and save it in the variable @Token. 
             The $.id is a JSONPath definition indicating the attribute value to extract from the JSON data. The ($) indicates start from the root of the JSON structure and the (.id) 
             indicates get the value of the first id attribute in the JSON data.
    e.	Press the Add Step button.

            File contents:
            {
                "id":null,
                "user":
                {
                    "id":-1,
                    "loginName":"@User",
                    "password":"@Password"
                },
                "tokenType":
                {
                    "id":null,
                    "type":"User"
                }
            } 

```
## Saving variable value in OpCon property 
Execute a request on a Web Server and save a returned value in an OpCon Schedule Instance property. 

When the tasks are completed, the variables values will be stored in OpCon properties according to the definitions in the **Property update on Completion** tab.

In the **Update Property on Completion** section of **Global Values** tab, enter the full path to the SI property. Remember that the dot (.) is used as a field separator, so it is not possible to use a schedule or job name that has a period (.) within the name. The OpCon-API requires that the date has the yyyy-mm-dd syntax, so a special $SCHEDULE DATE property is defined.

Variable Name                                      | Variable Value
-------------------------------------------------- | -----------
SI.Version.[[$SCHEDULE DATE-YYYY-MM-DD]].SCHED001  | @Version

In the **Step** tab, enter the following:
``` 

1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a GET request to obtain the OpCon-API version.
    a.	Select the GET function.
    b.	Enter the full URL https://@Url/api/version (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Response Variable Management tab. 
        iii. The request will return the OpCon-API version and we need to extract the value and place it in a variable that will be used by the subsequent steps. In the Attribute section 
             add @Version into the Name field and $.opConRestApiProductVersion into the value field which will extract the first id attribute returned in the JSON data and save it in the 
             variable @Version.
             The $.opConRestApiProductVersion is a JSONPath definition indicating the attribute value to extract from the JSON data. The ($) indicates start from the root of the JSON
             structure and the (.opConRestApiProductVersion) indicates get the value of the first id attribute in the JSON data.
        iv.	 Select the Step Completion TAB.
        v.	 Set the Step completion code to 200.
    e.	Press the Add Step button.
 
```
## Windows Authentication
Execute a request on an IIS Web Server using Windows Authentication.

In the **Variables** section of the **Global variables** tab,  enter the @User, @Password and @Domain variables that will provide the information used during authentication (the password value should be placed in an encrypted property). To increase portability when sharing templates, enter the web server address using the @Url variable.
 
Variable Name   | Variable Value
--------------- | -----------
@Url            | OPCON01
@User           | USER001
@Password       | [[USER001_PWD]]
@DOMAIN         | OPCON001

In the **Step** tab, enter the following:

``` 
1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a GET request to retrieve information from the Web Server.
    a.	Select the GET function.
    b.	Enter the full URL https://@Url/version (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Header tab.
        iii. In the Attributes section indicate Windows authentication will be used. Enter an attribute consisting of Authorization into the Name field and NTLM into the Value field. The 
        Connector will detect that Windows authentication is required and create the required credentials for such a connection.
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Step Completion TAB.
        iii. Set the Step completion code to 200.
    e.	Press the Add Step button.

```
## Using Client Certificates
Execute a request on Web Server using Client certificate for Authentication.

In the Variables section of the Global variables enter the @CertStore, @CertStorePwd and @CertStoreType variables that will provide the information used during authentication (the password value should be placed in an encrypted property). 
 
Variable Name   | Variable Value
--------------- | -----------
@CertStore      | c:\wsconnector\store\badssl.pkcs12
@CertStorePwd   | [[BADSSL_PWD]]
@CertStoreType  | PKCS12

In the **Step** tab, enter the following:
``` 
1.	Step 1 (Enter data in the (+) definition).
    The first Step function is a GET request to retrieve information from the Web Server.
    a.	Select the GET function.
    b.	Enter the full URL https://client.badssl.com (in this case the https indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Header tab.
        iii. In the Attributes section indicate certificate authentication will be used. Enter an attribute consisting of Authorization into the Name field and CERT into the Value field. 
             The Connector will detect that certification authentication is required and create the required credentials for such a connection.
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Step Completion tab.
        iii. Set the Step completion code to 200.
    e.	Press the Add Step button.

``` 
## File upload
Upload a file to a webserver.

When uploading a file, the file information is entered into the File Name field of the Request information and the multipart/form-data Content-Type is selected. The selection of the multipart/form-data indicates that this is a file upload request. The file must reside on the server where the connector is installed.
 
In the **Step** tab, enter the following:
``` 
1.	Step (Enter data in the (+) definition).
    The Step function is a POST request to submit information to the Web Server.
    a.	Select the POST function.
    b.	Enter the full URL http://httpbin.org/post (in this case the http indicates that the connector will use non-TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select multipart/form-data from the Content Type drop-down list.
        ii.	 Select the Body TAB.
        iii. In the File Name field enter the full name of the file that must be uploaded.
    d.	Select the Response tab.
        i.	 Select application/json from the Content Type drop-down list.
        ii.	 Select the Step Completion tab.
        iii. Set the Step completion code to 200.
    e.	Press the Add Step button.

```
## SOAP Webservices
Submit a POST request to a SOAP Webservice.

It is possible to submit a POST request from the WebServices connector to a SOAP Webserver. When using this capability, the Message-Type=SOAP attribute must be set in the header attributes, the submitted Content-Type must be text/xml, the response Content-Type should be application/xml and the message body must contain the complete soap definition (soap12:Envelope). 

The example below will submit a request to a SOAP webserver and then extract a value from the returned data into a variable @Fahrenheit.

In the **Step** tab, enter the following:
```  
1.	Step (Enter data in the (+) definition).
    The Step function is a POST request to submit information to the Web Server.
    a.	Select the POST function
    b.	Enter the full URL https://www.w3schools.com/xml/tempconvert.asmx (in this case the http indicates that the connector will use TLS when communicating with the web server).
    c.	Select the Request tab.
        i.	 Select text/xml from the Content Type drop-down list.
        ii.	 Add Message-Type=SOAP header attribute.
        iii. Select the Body tab.
        iv.	 In the Message Body field insert the SOAP envelope.

            <?xml version="1.0" encoding="utf-8"?>
            <soap12:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap12="http://www.w3.org/2003/05/soap-envelope">
            <soap12:Body>
                <CelsiusToFahrenheit xmlns="https://www.w3schools.com/xml/">
                <Celsius>50</Celsius>
                </CelsiusToFahrenheit>
            </soap12:Body>
            </soap12:Envelope>
    
    d.	Select the Response tab.
        i.	 Select application/xml from the Content Type drop-down list.
        ii.	Insert the attribute extract information into the Response Variable Management table. Variable name @Fahrenheit value //CelsiusToFahrenheitResponse/CelsiusToFahrenheitResult/
            text().
        iii. Select the Step Completion tab.
        iv.  Set the Step completion code to 200.
    e.	Press the Add Step button.

```      
When trying to define the SOAP envelope, the best approach is to retrieve the wsdl definition by submitted the https://www.w3schools.com/xml/tempconvert.asmx?wsdl in a browser. This will return the end-points supported by the tempconvert.asmx service.
