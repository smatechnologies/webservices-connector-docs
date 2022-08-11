# Installation

## Requirements
- The SMAWSConnector can be installed on any Linux or Windows Server where there is an OpCon Agent installed.
- The SMAWSConnector uses the OpCon-API capabilities of OpCon to update properties and extract the unique job id of a job when the @JCorrellationid feature is used. 
- For installation on Linux environments the following LSAM configuration parameters should be set
  -	LSAM_0_255	should be set to 1 
  -	path_to_su	should be set to yes

## Restrictions
- No Java restrictions as the connector uses an embedded AdoptOpenJRE 11 which is installed during the connector installation.
- Requires OpCon version 21.0.x.
- Parsing of returned data for variables, is only supported for JSON and XML formats.
- The maximum size of the JSON data for Linux systems is limited to 2000 characters due to limitation of Linux parameters field size.
- The maximum size of the JSON data for Windows systems is limited to 4000 characters due to limitation of Windows command line field size.
- When using properties within the JSON definition, it should be noted that the property definition is inserted to the JSON definition at execution time. This means that if the property contains a backslash (\) it will not be escaped resulting in a JSON parsing error when the connector starts up. If possible, escape the backslash (\\) in the property. If this is not possible use an Environment Variable.
- When using Environment Variables, the Windows or UNIX Agent supporting the connector must support this feature.

## Installing the Connector Windows
Copy the supplied zip file SMAWebServicesConnector-win.zip to a directory on the server.
Unzip the file and extract the contents to a directory on the server. 

After the extraction is complete, the installation root directory contains the connector executable, a config directory containing the Connector.config file, a java directory containing the required Java environment, an emplugins directory containing the Job Sub type and a templates directory containing a basic template.

## Installing the Connector Linux
Copy the supplied zip file SMAWebServicesConnector-linux.zip to a directory on the server.
Unzip the file and extract the contents to a directory on the server. 

Create a ‘config’ directory off the root installation directory and copy the Connector.config file into this directory.

Set execute options on SMAWSConnector.sh and /java/bin/java using chmod.

## Installing the Connector Docker Image
The image can be downloaded from Docker Hub in the smatechnologies repository. Add the following information in the Docker composer or Kubernetes yaml files.
containers:
      - image: smatechnologies/ connector-webservices:latest

The image is a Linux image that contains a Linux SMA Agent and the connector Linux implementation. The image is installed using Kubernetes (see section Kubernetes yml File).
The following definition can be used to include the connector in an OpCon Kubernetes deployment yaml file.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ws-config
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: ws-connector
spec:
  replicas: 1
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - wsconnectorservices
  template:
    metadata:
      labels:
        app: wsconnectorservices
    spec:
      containers:
      - image: smaengineering/connector-webservices:latest
        name: ws-connector
        ports:
        - containerPort: 3100
          protocol: TCP
        - containerPort: 3110
          protocol: TCP
        volumeMounts:
        - name: ws-config
          mountPath: /app/config
      hostname: ws-connector
      volumes:
      - name: ws-config
        persistentVolumeClaim:
          claimName: ws-config
---
apiVersion: v1
kind: Service
metadata:
  name: ws-connector
spec:
  type: LoadBalancer
  ports:
  - name: wswork
    port: 3100
    targetPort: 3100
  - name: wsjors
    port: 3110
    targetPort: 3110
  selector:
    app: wsconnectorservices
```
After installation, configure the Webservices Linux agent (use get svc to display the Address of the load balancer associated with the uatws-connector POD in Kubernetes definition).

```
   NAME          TYPE            CLUSTER-IP      EXTERNAL-IP       PORT(S)                            AGE
   kubernetes    ClusterIP       10.0.0.1        <none>            443/TCP                            22d
   opcon         LoadBalancer    10.0.199.201    52.171.228.74     9010:331702/TCP                    66s
   ws-connector  LoadBalancer    10.0.34.38      52.171.228.113    3100:30641/TCP,3110:31161/TCP      66s

```
Login to the OpCon instance and create the UNIX machine definition for the connector container using the External-IP of the uatws-connector POD (remember to include the jors port).

## Job Subtype Installation
Copy the Enterprise Manager plug-in from the installation /emplugins directory to the dropins directory of the Enterprise Manager installation. 
If the dropins directory does not exist, create the dropins directory off the root directory. 

Restart Enterprise Manager and a new Windows or UNIX job subtype called Webservices will be visible.

If not restart Enterprise Manager using 'Run as Administrator'. After this Enterprise Manager can be used normally.

## Create Global Properties for Web Services job sub-type
Within OpCon create a global property that points to the root installation directory of the connector. 
- Windows installation 
  - name is WS_PATH
  - value is ***root installation directory***
- Linux installation
  - name is WS_UNIX_PATH
  - value is ***root installation directory***
- Docker installation
  - name is WS_UNIX_PATH
  - value is /app

## Configuration
The configuration of the SMAWSConnector requires setting a default data directory and defining the connection to the OpCon-API of the host OpCon system.

When defining the value for the Containerized Webservices implementation, the software is installed in location /app.

### Connector.config File
The Connector.config file is in the following locations
- Windows
  - \\***root installation directory***\\config directory
- Linux
  - /***root installation directory***/config directory
- Docker
  - /app/config directory

The Connector.config contains the following values

Property Name | Value
--------- | -----------
**[GENERAL]**                | header
**DATA_DIRECTORY**           | A default data directory that can be used by the connector for storing created templates. When entering a value remember to include double \ characters.
**USES_PROXY**               | Indicates if the Connector uses a proxy server. Values are True or False, default is False.
**DEBUG**                    | Turns debug value on or off. Should be set when defining steps as this will dump all the data which can be analyzed to determine the location of attribute values to extract.Values are True or False, default is False.
**[PROXY]**                  | header - required if **USES_PROXY** is set to True.
**URL**                      | The full proxy server URL (i.e. http://***proxy server***:***port***).
**[OPCON_API]**              | header - required and defines the connection to the OpCon-API
**SERVER**                   | The address of the host OpCon server is inserted into the Connector.config file by using the -–setup switch when executing the connector. 
**USETLS**                   | Indicates if the OpCon Rest API server is supporting TLS and inserted into the Connector.config file by using the -–setup switch when executing the connector. 
**TOKEN**                    | The application level token value which is set by using the -–setup switch when executing the connector. A CONNECTORS application token is created, and the value of the token is inserted into the Connector.config file.

Example configuration file. 

```
[GENERAL]
DATA_DIRECTORY=c:\\connectors\\wsrest\\data
USES_PROXY=False
DEBUG=False

[PROXY]
URL=

[OPCON_API]
SERVER=BVHTEST02:9010
USESTLS=True
TOKEN=e4185480-7137-4bca-8220-0dccc555a946

```

### OpCon-API Configuration Setup 
The connector needs to connect to the OpCon host through the OpCon Rest API to retrieve JCorrelationid values, set the Incident value in the daily jobs and insert / update properties. The connector uses an application level token when communicating with the OpCon host. 

The OPCON_API section of the Connector.config file contains the information about the connection to the OpCon host including the server address, if TLS is being used and the application token value. 

These values can set by calling the SMAWSConnector.exe (Windows) or SMAWSConnector.sh (Linux) with the --setup switch which creates an application token (named **CONNECTORS**) and inserts the OpCon host address, tls and token information into the OPCON_API section of the Connector.config file. 

```
SMAWSConnector.exe/sh --setup -username -password -address <server>:<port> -tls
	Where   --setup     is the switch indicating that an application token should be should be created and the Connector.config file should be updated.
            -username   is the name of an OpCon user that has the ocadm role. Enclose the username in quotes in case special characters used.
            -password   the password of the user. Enclose the username in quotes in case special characters used.
            -address    the address and port used by the OpCon Rest API web server.
            -tls        if using tls set this argument. Please note that from OpCon version 20 only a tls connection is supported.
```

Example Windows:
```
SMAWSConnector.exe --setup -username "ocadm" -password "abc" -address opcon:9010 -tls
```
Example Linux:
```
SMAWSConnector.sh --setup -username "ocadm" -password "abc" -address opcon:9010 -tls
```
Example Container:
Start Image of job used to set the application token into the Connector.config file, using the External-IP value of the OpCon container.
```
[[WS_UNIX_PATH]]/CSMAWSConnector.sh --setup -username ocadm -password abc -address 52.249.57.233:9010 -tls
```

