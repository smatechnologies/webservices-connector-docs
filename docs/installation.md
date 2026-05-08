---
sidebar_label: 'Installation'
title: Installing the Webservices Connector
description: "Requirements, restrictions, and installation procedures for the Webservices Connector on Windows, Linux, and Docker."
tags:
  - Procedural
  - System Administrator
  - Connectors
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Installation

## What is it?

This page describes how to install and configure the Webservices Connector on Windows, Linux, or as a Docker image. It also covers installing the **Web Services** job sub-type plug-in for Enterprise Manager and creating the global properties the connector requires.

## Installation flow

A complete installation has four steps. Complete them in order:

1. **Install the connector binary** on the host server (Windows, Linux, or Docker).
2. **Install the Job Sub-type plug-in** in Enterprise Manager so you can define **Web Services** jobs.
3. **Create the global property** that points OpCon to the connector installation.
4. **Configure the connector** by editing `Connector.config` and running the `--setup` switch to register an application token with the OpCon REST API.

## Before you begin

### Prerequisites

- An OpCon Agent is already installed on the target Linux or Windows server.
- OpCon version `21.0.x` or greater.
- An OpCon user with the `ocadm` role (used during the `--setup` step).

:::note Linux Agent settings
If you are installing on Linux, set the following Linux Agent configuration parameters before installing the connector:

| Parameter | Value |
|-----------|-------|
| `LSAM_0_255` | `1` |
| `path_to_su` | `yes` |
:::

### Limits and caveats

:::caution JSON payload size limits
The maximum JSON payload size depends on the operating system:

| OS | Limit | Reason |
|----|-------|--------|
| Linux | 2000 characters | Linux parameter field size |
| Windows | 4000 characters | Windows command-line field size |
:::

:::caution Backslashes in OpCon properties
When an OpCon property is referenced inside a JSON definition, OpCon resolves the property at run time without escaping backslashes. An unescaped backslash (for example, in `\\server\directory`) breaks JSON parsing.

**Workarounds:**
- Escape the backslash in the property value (`\\`).
- Use an Environment Variable instead (Environment Variables are not subject to JSON parsing).
:::

:::note Other restrictions
- Java is bundled — the connector ships with an embedded AdoptOpenJRE 11. No separate Java install is required.
- Response data parsing is supported only for `application/json` and `application/xml`.
- To use Environment Variables, the supporting Windows or UNIX Agent must support that feature.
:::

## Step 1: Install the connector binary

<Tabs groupId="install-target" queryString>
  <TabItem value="windows" label="Windows" default>

To install the connector on Windows, complete the following steps:

1. Copy `SMAWebServicesConnector-win.zip` to a directory on the server.
2. Unzip the file and extract the contents to a directory on the server.

After the extraction is complete, the installation root directory contains:

| Path | Contents |
|------|----------|
| `SMAWSConnector.exe` | The connector executable. |
| `config\` | Holds `Connector.config`. |
| `java\` | The embedded Java runtime. |
| `emplugins\` | The Enterprise Manager plug-in for the **Web Services** job sub-type. |
| `templates\` | A basic job template. |

  </TabItem>
  <TabItem value="linux" label="Linux">

To install the connector on Linux, complete the following steps:

1. Copy `SMAWebServicesConnector-linux.zip` to a directory on the server.
2. Unzip the file and extract the contents to a directory on the server.
3. Create a `config` directory off the root installation directory and copy `Connector.config` into it.
4. Set execute options on `SMAWSConnector.sh` and `/java/bin/java`:

   ```bash
   chmod +x SMAWSConnector.sh
   chmod +x java/bin/java
   ```

  </TabItem>
  <TabItem value="docker" label="Docker">

The image is published to Docker Hub in the `smatechnologies` repository and contains a Linux SMA Agent and the connector Linux implementation.

To pull the image, reference it from your Docker Compose or Kubernetes yaml:

```yaml
containers:
  - image: smatechnologies/connector-webservices:latest
```

To deploy the connector in an OpCon Kubernetes cluster, use the following yaml:

```yaml
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

After deployment, run `kubectl get svc` to display the load balancer address for the `ws-connector` POD:

```text
NAME          TYPE            CLUSTER-IP      EXTERNAL-IP       PORT(S)                            AGE
kubernetes    ClusterIP       10.0.0.1        <none>            443/TCP                            22d
opcon         LoadBalancer    10.0.199.201    52.171.228.74     9010:331702/TCP                    66s
ws-connector  LoadBalancer    10.0.34.38      52.171.228.113    3100:30641/TCP,3110:31161/TCP      66s
```

Log in to the OpCon instance and create the UNIX machine definition for the connector container using the `EXTERNAL-IP` of the `ws-connector` POD. Remember to include the JORS port.

  </TabItem>
</Tabs>

## Step 2: Install the Job Sub-type plug-in

To install the **Web Services** job sub-type plug-in in Enterprise Manager, complete the following steps:

1. Copy the plug-in from the connector installation `/emplugins` directory into the `dropins` directory of the Enterprise Manager installation.

   :::note
   If the `dropins` directory does not exist, create it off the Enterprise Manager root directory.
   :::

2. Restart Enterprise Manager.

A new Windows or UNIX job sub-type called **Web Services** is now visible.

:::tip Plug-in not visible?
Restart Enterprise Manager using **Run as Administrator**. After this, Enterprise Manager can be used normally.
:::

## Step 3: Create the global property

Create a global property in OpCon that points to the root installation directory of the connector. The property name depends on the install type:

| Installation | Property name | Value |
|--------------|---------------|-------|
| Windows | `WS_PATH` | *root installation directory* |
| Linux | `WS_UNIX_PATH` | *root installation directory* |
| Docker | `WS_UNIX_PATH` | `/app` |

## Step 4: Configure the connector

Configuration consists of two tasks:

1. Review or edit the `Connector.config` file (see [Connector.config file](#connectorconfig-file)).
2. Register an application token with the OpCon REST API by running the connector with the `--setup` switch (see [OpCon REST API setup](#opcon-rest-api-setup)).

### Connector.config file

The `Connector.config` file lives in the following location for each install type:

| Installation | Location |
|--------------|----------|
| Windows | *root installation directory*`\config\Connector.config` |
| Linux | *root installation directory*`/config/Connector.config` |
| Docker | `/app/config/Connector.config` |

The file is divided into three sections — `[GENERAL]`, `[PROXY]`, and `[OPCON_API]`.

#### [GENERAL] section

| Property | Description | Default |
|----------|-------------|---------|
| `DATA_DIRECTORY` | A default data directory used by the connector for storing created templates. On Windows, escape backslashes (`\\`). | — |
| `USES_PROXY` | Indicates whether the connector uses a proxy server. | `False` |
| `UPDATE_PROPERTIES_ON_FAILURE` | Indicates whether the connector should update OpCon properties when a task fails. | `False` |
| `DEBUG` | Turns debug output on or off. Set to `True` while defining steps to dump returned data so you can locate attribute values to extract. | `False` |

#### [PROXY] section

Required only if `USES_PROXY` is `True`.

| Property | Description |
|----------|-------------|
| `URL` | Full proxy server URL (for example, `http://`*proxy server*`:`*port*). |

#### [OPCON_API] section

Defines the connection to the OpCon REST API. The values in this section are populated for you by the `--setup` switch — you do not edit them by hand.

| Property | Description |
|----------|-------------|
| `SERVER` | The address of the host OpCon server. |
| `USETLS` | Indicates whether the OpCon REST API server uses TLS. |
| `TOKEN` | The application-level token. The `--setup` switch creates a `CONNECTORS` application token and writes it here. |

#### Example Connector.config

```ini
[GENERAL]
DATA_DIRECTORY=c:\\connectors\\wsrest\\data
USES_PROXY=False
UPDATE_PROPERTIES_ON_FAILURE=False
DEBUG=False

[PROXY]
URL=

[OPCON_API]
SERVER=BVHTEST02:9010
USESTLS=True
TOKEN=e4185480-7137-4bca-8220-0dccc555a946
```

### OpCon REST API setup

The connector connects to the OpCon host through the OpCon REST API to retrieve `@JCorrelationid` values, set the incident value in daily jobs, and insert or update properties. It uses an application-level token for these calls.

The `--setup` switch automates the token creation and writes the connection details into the `[OPCON_API]` section of `Connector.config`.

#### Command syntax

```text
SMAWSConnector.exe --setup -username <user> -password <password> -address <server>:<port> [-tls]
```

| Argument | Description |
|----------|-------------|
| `--setup` | Creates a `CONNECTORS` application token and updates `Connector.config`. |
| `-username` | Name of an OpCon user that has the `ocadm` role. Enclose in quotes if it contains special characters. |
| `-password` | Password of the user. Enclose in quotes if it contains special characters. |
| `-address` | Address and port of the OpCon REST API web server. |
| `-tls` | Use TLS to connect. |

:::caution TLS requirement
From OpCon version 20 onward, only TLS connections are supported. Always include the `-tls` argument.
:::

#### Examples

<Tabs groupId="install-target" queryString>
  <TabItem value="windows" label="Windows" default>

```bash
SMAWSConnector.exe --setup -username "ocadm" -password "abc" -address opcon:9010 -tls
```

  </TabItem>
  <TabItem value="linux" label="Linux">

```bash
SMAWSConnector.sh --setup -username "ocadm" -password "abc" -address opcon:9010 -tls
```

  </TabItem>
  <TabItem value="docker" label="Docker">

Run from inside the container, using the `EXTERNAL-IP` of the OpCon container as the address:

```bash
[[WS_UNIX_PATH]]/SMAWSConnector.sh --setup -username ocadm -password abc -address 52.249.57.233:9010 -tls
```

  </TabItem>
</Tabs>

## FAQs

**Where can the connector be installed?**
On any Linux or Windows server where an OpCon Agent is installed, or as a Docker image from Docker Hub in the `smatechnologies` repository.

**Which OpCon version is required?**
OpCon version `21.0.x` or greater.

**What size limits apply to JSON data?**
2000 characters on Linux and 4000 characters on Windows, due to OS command-line and parameter field size limits.

**Why might a backslash in a property cause a JSON parse error?**
When using properties within a JSON definition, OpCon inserts the property value at run time and does not escape backslashes. If the value contains an unescaped backslash, JSON parsing fails when the connector starts. Either escape the backslash (`\\`) in the property value or use an Environment Variable instead.

**How do I create the application token used by the connector?**
Run `SMAWSConnector.exe` (Windows) or `SMAWSConnector.sh` (Linux) with the `--setup` switch, passing an OpCon user that holds the `ocadm` role. The connector creates a `CONNECTORS` application token and writes it to `Connector.config`.

**Which global property points to the connector installation directory?**
- Windows: `WS_PATH`
- Linux: `WS_UNIX_PATH`
- Docker: `WS_UNIX_PATH` set to `/app`

## Glossary

| Term | Definition |
|------|------------|
| SMAWSConnector | The Webservices Connector executable (`SMAWSConnector.exe` on Windows, `SMAWSConnector.sh` on Linux). |
| Connector.config | The configuration file that defines the data directory, proxy settings, and OpCon REST API connection used by the connector. |
| Web Services job sub-type | The Enterprise Manager job sub-type, installed by the connector plug-in, used to define Webservices Connector jobs. |
| `--setup` switch | Command-line switch that creates the `CONNECTORS` application token and writes the OpCon host address, TLS, and token values to `Connector.config`. |
| WS_PATH | Global property pointing to the connector root installation directory on Windows. |
| WS_UNIX_PATH | Global property pointing to the connector root installation directory on Linux or Docker. |
| dropins | Enterprise Manager directory used to load the **Web Services** job sub-type plug-in. |
| AdoptOpenJRE 11 | The embedded Java runtime supplied with the connector installation. |
