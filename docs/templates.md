# Templates
This section contains various job templates that can be used when working with the Webservices Connector.

## Get without authentication 

```
{
  "templateid" : "OPCON-API_version",
  "steps" : [ {
    "function" : "GET",
    "url" : "https://<server name>:9010/api/version",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## OpCon property update using Token authentication

```
{
  "templateid" : "OpConAPI-PUPD",
  "steps" : [ {
    "function" : "POST",
    "url" : "https://<server name>:9010/api/tokens",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ ],
      "body" : "{\"id\":null,\"user\":{\"id\":-1,\"loginName\":\"@User\",\"password\":\"@Password\"},\"tokenType\":{\"id\":null,\"type\":\"User\"}}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.id"
      } ]
    }
  }, {
    "function" : "GET",
    "url" : "https://<server name>:9010/api/globalproperties?name=@GlobalPropertyName",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Propertyid",
        "value" : "$[0].id"
      } ]
    }
  }, {
    "function" : "PUT",
    "url" : "https://<server name>:9010/api/globalproperties/@Propertyid",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : "{\"id\":\"@Propertyid\",\"name\":\"@GlobalPropertyName\",\"value\":\"wsrest 1234 testvalue\"}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
}, {
    "name" : "@GlobalPropertyName",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## OpCon schedule build including poll to check build result using Token authentication

```
{
  "templateid" : "OpConAPI-SBUILD",
  "steps" : [ {
    "function" : "POST",
    "url" : "https://<server name>:9010/api/tokens",
    "request" : {
      "headers" : [ ],
      "body" : "{\"id\":null,\"user\":{\"id\":-1,\"loginName\":\"@User\",\"password\":
      \"@Password\"},\"tokenType\":{\"id\":null,\"type\":\"User\"}}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.id"
      } ]
    }
  }, {
    "function" : "POST",
    "url" : "https://<server name>:9010/api/schedulebuilds",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : "{\"schedules\":[{\"id\":null,\"name\":\"@ScheduleName\"}],\"id\":null,\"expiryTime\":null,\"dates\":[\"@ScheduleDate\"],\"logFile\":null,\"overwrite\":true,\"hold\":true}",
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 201,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ {
        "name" : "@Buildid",
        "value" : "$.id"
      } ]
    }
  }, {
    "function" : "GET",
    "url" : "https://<server name>:9010/api/schedulebuilds/@Buildid",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Token @Token"
      } ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "success/Completed",
        "badFin" : "Failed",
        "attributeToCheck" : "$[0].message",
        "poll" : true,
        "pollDelay" : 3,
        "pollInterval" : 2,
        "pollMaxTime" : 2
      },
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
  }, {
    "name" : "@ScheduleName",
    "value" : "??????"
  }, {
    "name" : "@ScheduleDate",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## Get using Basic authentication

```
{
  "templateid" : "EASYVISTA",
  "steps" : [ {
    "function" : "GET",
    "url" : "https://uap.easyvista.com:/api/v1/@Company/requests",
    "request" : {
      "contentType" : "application/json",
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "Basic"
      } ],
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null,
      "variables" : [ ]
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
  }, {
    "name" : "@Company",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## Get using Windows Authentication to IIS

```
{
  "templateid" : null,
  "steps" : [ {
    "function" : "GET",
    "url" : "https://10.1.0.4/version.json",
    "proxyServer" : null,
    "request" : {
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "NTLM"
      } ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  } ],
  "variables" : [ {
    "name" : "@User",
    "value" : "??????"
  }, {
    "name" : "@Password",
    "value" : "??????"
  }, {
    "name" : "@Domain",
    "value" : "??????"
  }, {
    "name" : "@User1",
    "value" : "??????"
  }, {
    "name" : "@Password1",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## Get using Certificate

```
{
  "templateid" : "certifcate",
  "steps" : [ {
    "function" : "GET",
    "url" : "https://client.badssl.com",
    "proxyServer" : null,
    "request" : {
      "headers" : [ {
        "attribute" : "Authorization",
        "value" : "CERT"
      } ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  } ],
  "variables" : [ {
    "name" : "@CertStore",
    "value" : "??????"
  }, {
    "name" : "@CertStorePwd",
    "value" : "??????"
  }, {
    "name" : "@CertStoreType",
    "value" : "??????"
  } ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
``` 
## WebServices VisualCron Embedded Script Definitions
The following definition contains the Embedded Script WebServices-vcon-vars 
```
{
  "templateid" : null,
  "steps" : [ {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/logon?username=@User&password=@Password&expire=3600",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.Token"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/GetByName?token=@Token&name=@Jobname",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Jobid",
        "value" : "$.Id"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Run?token=@Token&id=@Jobid&variables=@Variables",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "1",
        "badFin" : "2",
        "attributeToCheck" : "$.Stats.Status",
        "poll" : true,
        "pollDelay" : 3,
        "pollInterval" : 2,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "0",
        "badFin" : null,
        "attributeToCheck" : "$.Stats.ExitCode",
        "poll" : false,
        "pollDelay" : null,
        "pollInterval" : null,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  } ],
  "variables" : [ ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
The following definition contains the Embedded Script WebServices-vcon-no-vars 
```
{
  "templateid" : null,
  "steps" : [ {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/logon?username=@User&password=@Password&expire=3600",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Token",
        "value" : "$.Token"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/GetByName?token=@Token&name=@Jobname",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ {
        "name" : "@Jobid",
        "value" : "$.Id"
      } ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Run?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : null,
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "1",
        "badFin" : "2",
        "attributeToCheck" : "$.Stats.Status",
        "poll" : true,
        "pollDelay" : 3,
        "pollInterval" : 2,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  }, {
    "function" : "GET",
    "url" : "http://@Url/VisualCron/json/Job/Get?token=@Token&id=@Jobid",
    "proxyServer" : null,
    "request" : {
      "headers" : [ ],
      "contentType" : "application/json",
      "body" : null,
      "fileName" : null
    },
    "response" : {
      "contentType" : "application/json",
      "variables" : [ ],
      "ignoreResult" : false,
      "stepCompletionCode" : 200,
      "responseDataCheck" : {
        "goodFin" : "0",
        "badFin" : null,
        "attributeToCheck" : "$.Stats.ExitCode",
        "poll" : false,
        "pollDelay" : null,
        "pollInterval" : null,
        "pollMaxTime" : null
      },
      "fileName" : null
    }
  } ],
  "variables" : [ ],
  "environmentVariables" : [ ],
  "properties" : [ ]
}
```
## Kubernetes yml File
The following is a Kubernetes yaml file that can be used to deploy an OpCon environment within a Kubernetes cluster. The deployment consists of 3 containers (OpCon, Deploy (Impex2), Webservices). The environment uses Azure-SQL for the OpCon database.  
```
# 
# full OpCon deployment for Kubernetes
# includes opcon, deploy and webservices PODs
#
# uses Azure SQL Database
# database should be created before starting 
# LICENSE is OpCon 21.0 0.lic generated value
#
apiVersion: v1
kind: Secret
metadata:
  name: uat-dbpasswords
stringData:
  saPassword: "password"
  dbPassword: "password"
  sqlAdminPassword: "password"
  dbPasswordEncrypted: "encrypted password using OpCon encryption"

---
# OpCon environment values
apiVersion: v1
kind: ConfigMap
metadata:
  name: opconenv
data:
  DB_SERVER_NAME: "sscazuretest.database.windows.net"
  DATABASE_NAME: "opconxps"
  DB_USER_NAME: "admin"
  SQL_ADMIN_USER: "admin"
  API_USES_TLS: "true" 
  CREATE_API_CERTIFICATE: "true"
  DB_SETUP: "true"
  TZ: "America/Chicago"
  LANG: "en_US.utf-8"
  LICENSE: "0:414c484b040615081644696d2e28273a317e2a2a682d2d7f2a64642d636307681b1b56195d186c246d3e3e6e3c733466276a6a2968266827737331747426733d3d743a3a5e3142420f400441357d34670d0a256d24777727753a7d2f6e232360216f216e3a3a783d3d6f3a74743d737317780b0b46094d087c347d2e2e7e2c632476377a0d0a405245534654445048592366347b377e3d3d3d3d2c3e2a3a2b3f2e3a2b39283d2a3f293a2f382c382b392f392c3a2e3a293f3c293e2a3e0d0a414d5e4e5e52421e781d7c087b6e626f7e22313d30217d6f636e7f23373b36277b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b6b"
---
# Deploy (Impex) environment values
apiVersion: v1
kind: ConfigMap
metadata:
  name: uat-impexenv
data:
  opcon.server.name: "sscazuretest.database.windows.net"
  opcon.db.name: "opconuat"
  opcon.db.user: "sscadmin"
  web.port: "9011"
  web.ssl: "true" 
  system.debug: "false"
  TZ: "America/Chicago"
  LANG: "en_US.utf-8"
---
# OpCon persistent storage for config information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-opconconfig
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# OpCon persistent storage for log information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-opconlog
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# Deploy (Impex) persistent storage for log information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-impexlog
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# Webservices persistent storage for config information
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uat-wsconfig
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
# OpCon pod 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: uat-opcon
spec:
  replicas: 1
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - uat-opconservices
  template:
    metadata:
      labels:
        app: uat-opconservices
    spec:
      containers:
      - env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: uat-dbpasswords
              key: dbPassword
        - name: SQL_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: uat-dbpasswords
              key: sqlAdminPassword
        envFrom:
          - configMapRef:
              name: uat-opconenv
        image: smatechnologies/opcon-server:latest
        name: uat-opcon
        ports:
        - containerPort: 443
          protocol: TCP
        volumeMounts:
        - name: uat-opconconfig
          mountPath: /app/config
        - name: uat-opconlog
          mountPath: /app/log
      hostname: uat-opcon
      volumes:
      - name: uat-opconconfig
        persistentVolumeClaim:
          claimName: uat-opconconfig
      - name: uat-opconlog
        persistentVolumeClaim:
          claimName: uat-opconlog
---
# Deploy (Impex) pod
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: uat-impex
spec:
  replicas: 1
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - uat-opconservices
  template:
    metadata:
      labels:
        app: uat-opconservices
    spec:
      containers:
      - env:
        - name: opcon.db.password
          valueFrom:
            secretKeyRef:
              name: uat-dbpasswords
              key: dbPasswordEncrypted
        envFrom:
          - configMapRef:
              name: uat-impexenv
        image: smatechnologies/deploy-impex2:latest
        name: uatimpex
        volumeMounts:
        - name: uat-impexlog
          mountPath: /app/log
      hostname: uat-impex
      volumes:
      - name: uat-impexlog
        persistentVolumeClaim:
          claimName: uat-impexlog
---
# Webservices pod
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: uat-wsconnector
spec:
  replicas: 1
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - uat-wsservices
  template:
    metadata:
      labels:
        app: uat-wsservices
    spec:
      containers:
      - image: smatechnologies/connector-webservices:latest
        name: uat-wsconnector
        ports:
        - containerPort: 3100
          protocol: TCP
        - containerPort: 3110
          protocol: TCP
        volumeMounts:
        - name: uat-wsconfig
          mountPath: /app/config
      hostname: uat-wsconnector
      volumes:
      - name: uat-wsconfig
        persistentVolumeClaim:
          claimName: uat-wsconfig
---
# OpCon svc
apiVersion: v1
kind: Service
metadata:
  name: uat-lbopcon
spec:
  type: LoadBalancer
  ports:
  - name: apiport
    port: 9010
    targetPort: 443
  - name: impexport
    port: 9011
    targetPort: 9011
  selector:
    app: uat-opconservices
---
# Webservices svc
apiVersion: v1
kind: Service
metadata:
  name: uat-lbws
spec:
  type: LoadBalancer
  ports:
  - name: uatwswork
    port: 3100
    targetPort: 3100
  - name: uatwsjors
    port: 3110
    targetPort: 3110
  selector:
    app: uat-wsservices

```
