# Overview
A HTTP API Gateway (aka proxy) for interacting with the Appgate API. The name is a portmanteau for `Controller` and `kolla`, translated from a discovered rune stone in Gothenburg in Sweden meaning: "to look at the Controller".

While the Customer Reliability Team has been working with various integration projects which involved Appgate SDP API, we learned that a tool, which was used over and over, will be handy to rapid prototyping. Eventually "the tool" became more feature rich and we started to believe it is useful for anyone working with Appgate SDP integrations. 

From an educational standpoint, Conkolla allows you to:

* Discover the API against a real system.
* Model or craft API requests and work with real responses.
* [Report data on API responses in form of JSON or CSV](./example_reporting.md)
* Let's you retrieve tokens when you have not yet covered the auth process in your script.
* Can be used as a API gateway connecting to many Appgate Controllers at the same time.
* Gives you discovery with a Web based UI.
* Gives you all the scripting flexibility to use the http rest API (JSON).
* Administrate your Appgate.
* Copy from one collective to another.
* [Do multi entity manipulation](./console_multi.png).
* One time system admin tasks that would be too heavy doing it in the UI.
* Prometheus pull Gateway, one target per connection.
* Relaying all snmp-exported prometheus metrics for all appliances in the collective.
* [Auto connect: control connections](./autoconnect.md) solely by a connections file. Useful in autonomous deployments (k8s/ConfigMap)
* Upgrade monitor, watching your cluster upgrade progress in a tabular format.
* Backup and download backup file with the UI.


![baup](backup-download.gif)

![edite](editentity.gif)

and many more.... 

The Appgate API can be learned and discovered in different ways. One of the common ways is to use the developer tools in a web browser while browsing through the Appgate admin UI, or use existing API Gateway or API Mocker such as postman or prism. 
However, with Conkolla you get another option to choose from.

# Usage
Conkolla can be seen as a proxy, sitting between the user and the Appgate Controller:
![login form](/arch.png)



The user will basically do the following steps when using Conkolla:
1. Launch Conkolla.
2. Login to a Appgate Controller through Conkolla. 
3. Use the the new connection in Conkolla to run rest calls against the connected Controller.

# Quick start
## Get Conkolla
Conkolla (64bit) runs on macOS, Windows and linux. Download the binaries:
* [Conkolla latest release](https://github.com/appgate/conkolla/releases/latest)

### Docker
```shell
docker pull mar8x/conkolla:latest
docker run -p 4433:4433 mar8x/conkolla:latest --authName maraboux --authPassword maraboux
```

### k8s
See the [example deployment.yaml](https://github.com/appgate/conkolla/tree/master/k8s):
```shell
# deploy
kubectl apply -f deployment.yaml

# pfwd to app
kubectl --namespace ${NAMESPACE} port-forward \
$(kubectl get pod --namespace ${NAMESPACE} -l app=conkolla -o template \
--template "{{(index .items 0).metadata.name}}") 4433
```
## Supported Browser
Currently Chrome is the browser for which testing is conducted on. Other browsers work as well but we cannot guarantee full functionality.

## Run Conkolla
By default Conkolla serves on `https://localhost:4433` with Basic Auth. Different options to launch are available, check with `Conkolla --help` for more information.
Keep the window in the background, it will reveal a good part of the interactions between client - Conkolla - Controller, and can be useful to follow the requests & responses, and payloads.

### Windows
Open a cmd line or powershell, then type `Conkolla.exe --authName maraboux --authPassword maraboux`. 

### Linux & macOS
```shell
chmod +x conkolla
./Conkolla --authName maraboux --authPassword maraboux
```
*you will see a start message, something like this:*
```shell
$ conkolla -noAuth
Basic Auth deactivated.
{
    "copyright": "AppGate Inc. 2021 (marx)",
    "date": "2021-02-09",
    "directory": "/Users/marc/git-appgate/conkolla/bin",
    "certPath": "templates/cert.pem",
    "keyPath": "templates/key.pem",
    "url": "https://localhost:4433",
    "tls": true,
    "mode": "release",
    "version": "8.2.0 (release-8:cc02)",
    "defaultAPIVersion": 14,
    "operatingSystem": "darwin",
    "memoryUsageMBbase2": 71.8291015625,
    "numberGoRoutines": 3,
    "systemTime": "02-10-2021 17:04:36",
    "systemTimeZone": "Local",
    "basicAuthentication": false,
    "conkollaID": "",
    "defaultClientConnectionParameters": {
        "tcpTimeout": 10000000000,
        "tlsHandshakeTimeout": 7000000000,
        "requestTimeout": 15000000000,
        "keepAlive": 30000000000,
        "maxIdleConnections": 100,
        "idleConnTimeout": 90000000000,
        "expectContinueTimeout": 1000000000,
        "helpText": "If prometheus enabled: queries to fetch new metrics upstream do always reset idling tcp connections (not re-useing from pool)."
    },
    "CertificateSANs": "",
    "upstreamHTTPGetOnly": false,
    "whiteListUpstream": null
}
{
    "b5b2739dd17c": [
        "127.0.0.1/8",
        "172.17.0.2/16"
    ]
}

```

## Conkolla in a deployment
If you are intending to use ck in a deployment, we recomend:
- Follow the principle of least privilege:
  - API account: define an admin role which restricts to the operations and objects needed for the purpose (see next chapter).
- Does it require read-only? 
  - If so launch ck with the flags: `-getOnly` to limit upstream calls to GET only method.
- Are you using it for monitoring only? 
  - If so launch ck with the flags: `-getOnly` and `whiteListMonitoring`.
- Do you need access to ck from the public? 
  - Put a reverse proxy infront with a proper TLS setup. 
  - Use basic auth on the reverse proxy.
  - Optionally use basic auth on ck -- if you use prometheus to scrape it locally, you might not need ck running on basic auth.


## Connect to a Controller
At this stage you will need to have a user with admin permission and privileges to read or write Appgate objects. For more information read:
* [SDP Help > rest api](https://sdphelp.appgate.com/adminguide/rest-apis.html)

Make sure you know where Conkolla is serving:
* look for "url" in start message, or
* use the default `https://localhost:4433`

You will encounter an option to specify the *API version*. Appgate SDP has different versions of the API available (from min-max version). During login, conkolla determines the correct (max) version to use and sets it for the connection. If conkolla is not able to determine, it will fall back to 

1. the built in version of conkolla (set at build time)
2. the overwritten version when launching conkolla (`--apiversion` overwrites the built in one)
3. the version set in the login form/json data

2 overwrites 1, and 3 overwrites 2.

Whenever the claims token is generated for the user, conkolla determines min and max again, and sets the version for the connection to max. This also happens if you force token renewal (example from the UI). You might change the version number for experimenting, but latest when user claims are created, the API version is set to max again. 

### Connect to a Controller through web-based UI
* Open a browser and point to the url (look for "url" in start message) 
* Default is [https://localhost:4433/login/](https://localhost:4433/login/)
* Allow the exception for the self signed cert when (if you have not deployed separate admin UI)


![login form](/login.png)

Check the [examples page](/examples.md) for how to connect via command line.

### Using the UI
It is recommended to use chrome browser (tested only in chrome/ECMAScript2018 or later). The browser helps user to quickly work against the Controller API. The UI supports
* Autocomplete for Upstream API path (indexed from the onboard API only).
* If entities have been in the results containing IDs, you can pre-set from a dropdown list API path with `{id}`.
* The on-board editor is the [Ace editor](https://github.com/ajaxorg/ace/wiki/Default-Keyboard-Shortcuts) configured for JSON mode.

#### Editing many entities (advanced)
You can only update or create one entity (for example an appliance or entitlement) with a single request. Hence you will need to have a single entity in the editor window (for create or update at least). 
If you need to manipulate multiple entities use the developer mode in chrome and manipulate records with JavaScript. Check the advanced page for details:

* [advanced, developer mode](developermode.md)

## Crafting REST calls via Conkolla
### Rest calls to an Appgate Controller
Rest calls towards an Appgate Controller are in the form of:
* `https://{URL}:{PORT}/admin/{RESOURCE or PATH}`

where as:
* `URL` is the Controllers URL
* `PORT`: the port of the admin access, default is `444`
* `RESOURCE or PATH`: the actual rest resource

Example: 
* `https://cc1.packnot.com:444/admin/stats/appliances`

### Rest calls via Conkolla
Note the separation of the different calls:
* From user to Conkolla: `proxy call`
* From Conkolla to Controller: `upstream call`

The first one is what the user will use to make calls, where the second is always initiated by Conkolla.

In Conkolla, a you always reference a connection; a connection which you previously logged into. Example, the same rest-call as above, you would specify it by referencing to connection `1` or `cc1.packnot.comdev`:
* `https://localhost:4433/get/1/stats/appliances`
* `https://localhost:4433/get/cc1.packnot.comdev/stats/appliances`

From the command line:
```shell
curl -k -X POST -H "Accept: application/json" https://localhost:4433/get/1/appliances
```
Use always method `POST` for the proxy call. The upstream HTTP method is included in the proxy call's path.

**Note: you need to drop the `/admin` part of the path.** Do not specify the `/admin` part in the path for the rest call. Conkolla adds it automatically on the upstream call.


| Upstream Call Method | Proxy Call Method | Description          | Proxy Call Parameters   |
|----------------------|:-----------------:|----------------------|-------------------------|
| GET                  |        POST       | Get entitites.       | `restCall`              |
| POST                 |        POST       | Create new entities. | `restCall` `objectData` |
| PUT                  |        POST       | Change entities.     | `restCall` `objectData` |
| DELETE               |        POST       | Delete entities      | `restCall`              |


**`restCall`**: the upstream rest call, example: _`/entitlements/0c32530f-e15b-4720-a261-acae73c3d417`_.
**`objectData`**: the payload (JSON data in the body).

Check the [examples page](/examples.md).


# Conkolla Reference 
## Login options and controls
Either you browse the login form from the UI or get the possible options/fields from a rest call. Note there might be more options from the rest call than on the web login form:
```shell
curl -k -H "Accept: application/json" https://localhost:4433/login/
```
Example:

```json
 "accept_header_suffix": "+json",
  "api_version": 16,
  "content_type_header": "application/json",
  "loginFormFields": {
    "controllerURL": "",
    "controllerPort": 0,
    "username": "",
    "password": "",
    "otp": "",
    "providerName": "",
    "apiVersion": 0,
    "acceptHeaderSuffix": "",
    "label": "",
    "machineId": "",
    "showToken": false,
    "skipVerifySSL": false,
    "dumpAGResponse": false,
    "autoTokenRenewal": false,
    "shortIntervalRenewal": 0,
    "renewToken": false,
    "contentTypeHeader": "",
    "promCollector": false,
    "promRelay": false,
    "promTargetName": "",
    "promLabelExporter": "",
    "promLabelEnvironment": "",
    "promLabelGroup": "",
    "promLabelSite": "",
    "PromLabelRegion": "",
    "promLabelCountry": "",
    "promLabelCustomer": "",
    "promLabelCostID": "",
    "kmsBlob": false,
    "kmsProvider": "",
    "kmsKey": "",
    "kmsRegion": "",
    "azureVault": false,
    "azureVaultName": "",
    "azureClientID": "",
    "azureSecretName": "",
    "autoConnectForce": false
  },
  "pageinfo": "login page"
}

```
From Version 7.4 JSON values are now separated from the form login, so they are treated all as proper data types (e.g booleans are bool and not string literals).


| Param                | Value                   | Description                                                                                                                                                                                                                                                                                                          |
|----------------------|:-----------------------:|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| label                | String:text             | Labels a connection with the given text. Allows you to connect many times to same Appgate Controller under different label. Labels allows you also to do bulk rest calls and help you sort and find connections. Any space in the will be replaced with `-`.                                                         |
| otp                  | String:number           | if the user account uses MFA, enter the next MFA code here. Supports only built-in MFA.                                                                                                                                                                                                                              |
| acceptHeaderSuffix   | `+json` or `+gpg`       | Default +json. +gpg is used for downloading backup files. You can change it after login. Since v. 7.4 headers switch automatically.                                                                                                                                                                                  |
| apiVersion           | Integer:number          | specifies what Appgate API version to indicate in the upstream headers, usually defaults ok. You can change it after login.                                                                                                                                                                                          |
| machineID            | String:UUIDv4           | A UUIDv4 string to identify Conkolla as a client ID towards Appgate Controller. None or faulty given, Conkolla generates a random one.                                                                                                                                                                               |
| showToken            | Bool:`true` or `false`  | Display Appgate token and, if used, the kms cipher.                                                                                                                                                                                                                                                                  |
| dumpAGResponse       | Bool:`true` or `false`  | Conkolla will log the the request and response send to the upstream server. Good for debugging or curious people.                                                                                                                                                                                                    |
| autoTokenRenewal     | Bool:`true` or `false`  | Conkolla will renew the token if it will expire in less than 5 minutes from now. Also, you will be able to force renewal of tokens by the `/renewtoken`call (see below). Auto Renew does not work when using MFA.                                                                                                    |
| renewToken           | Bool:`true` or `false`  | Setting this flag allows you to renew the token for an existing connection with a login request. The fields to identify existing connection are: `controllerURL` and `label`. The required field to renew token is:  `password` (and `otp` if required).                                                             |
| promCollector        | Bool:`true` or `false`  | Setting this flag allows you enable prometheus exporter for the connected collective. It acts like a pull gateway exported, reflecting all metrics of the Appgate collective                                                                                                                                         |
| promRelay            | Bool: `true` or `false` | Setting this flag will relay the snmp-prometheus metrics from all active appliances in Appgate. `promCollector` needs to be set to `true`. The target will be exported as usual but an additional target `targetname-synthetic` will be exported for synthetic connection tracing (same metrics as with the basic exporter). |
| kmsRegion            | String                  | Region string of the kms.                                                                                                                                                                                                                                                                                            |
| kmsKey               | String                  | KMS key ID.                                                                                                                                                                                                                                                                                                          |
| kmsProvider          | String:`"aws"`          | KMS provider. For now only AWS is supported.                                                                                                                                                                                                                                                                         |
| kmsBlob              | Bool:`true` or `false`  | Password is a KMS Blob.                                                                                                                                                                                                                                                                                              |
| azureVault           | Bool:`true` or `false`  | Use azure vault for password retrieval.                                                                                                                                                                                                                                                                              |
| azureSecretName      | String                  | The name of the secret which holds the password.                                                                                                                                                                                                                                                                     |
| azureVaultName       | string                  | The name of the vault.                                                                                                                                                                                                                                                                                               |
| autoConnectForce     | Bool:`true` or `false`  | Only used with the connection file, ignored in the login                                                                                                                                                                                                                                                             |
| shortIntervalRenewal | Integer:number          | Renew the tokens (auth&auth) every n minute. Lower bound is 3.                                                                                                                                                                                                                                                       |
# Security: Appgate user password handling
See the [dedicated password security page](./security.md) for this topic.

## Command line flags

```shell
Usage of /go/bin/conkolla:
  -address string
    	The IP of the proxy. Use 0.0.0.0 to share on network. Use localhost not to share. (default "localhost")
  -apiversion string
    	Default API version for new connections. (default "14")
  -authName string
    	Username for Basic Authentication.
  -authPassword string
    	Password for Basic Authentication.
  -certHosts string
    	Comma-separated hostnames and IPs to generate a certificate for. Generates new certs every time, ignoring the on-board cert.
  -combineLogs
    	Write connection logs to stdout instead of individual files.
  -conkollaID string
    	An arbitrary string to identify this running instance.
  -connectionsFile string
    	Specify file with path from which connections shall be read from.
  -fetchDataInterval int
    	The interval conkolla fetches data from controller (min 5 secs) (default 15)
  -getOnly
    	Allow only http method GET for all upstream call.
  -help
    	Display usage
  -http
    	Use http instead of https.
  -noAuth
    	Turn basic authentication off. If http is used instad of https, basic auth is turned off.
  -port string
    	The port of the proxy. (default "4433")
  -singlePromExporter
    	Export only the first connection under /metrics. Any other connection is ignored
  -version
    	8.2.1 (release-8:e548)
  -whiteListMonitoring
    	White list upstream calls required for monitoring. All Others are forbidden.
```
# Auto connection (connection file)
The auto connection functionality allows you to control connections configurations solely through a file. This is a rather large topic so it deserved its [own page](./autoconnect.md). 


# Prometheus metrics (basic metrics)
Conkolla translates and exposes metric information from connected controllers. 
![reference](./prometheus_metrics.png)
1. The collectors get the source data from the controllers. This is performed to all controllers in the collective, by using a probabilistic request balancer with constant back off time. Connections are traced as well from the conkolla to the controller (endpoint).
1. The data is transformed into prometheus gauges, counters and histograms and exposed as a target per connection.
1. Conkolla gets the data every 20secs.

The target metrics ar are exposed under its `target name`, the URL `https://localhost:4433/metrics?target=<target name>`. 

While these metrics are created in conkolla, they also contain synthetic transactions or so called information regarding the connection between the requests of conkolla in the controllers. These metrics are part exported together with the generated appgate metrics.

## Relaying snmp-exporter metrics (From v8.4.0)
Since v8.4.0 it is possible to relay all the snmp-exporter metrics from all active appliances in the collective. Note there are many unused or uninteresting metrics and a single appliance expose a data size of about 82KiB (Appgate SDP v5.5). However it contains metrics which the basic metrics do not deliver, so you might consider the metrics to `keep` which are of interest by using the `metric_relabel_configs` in your prometheus configuration.

Setting the flag `promRelay`:`true` for a connections together with `promCollector`:`true` does the following:
- conkolla does not create the "basic metrics"
- conkolla fetches for every active appliance the snmp-exported prometheus metrics
- conkolla relays all fetched metrics  in a single target
- conkolla exports an additional target named targetname`-synthetic` which contains the connection traces between conkollla and the controllers

## API user for prometheus/monitoring (basic metrics)
The api account does not need more privileges than:
- **View** all **Appliances**
- **View** all **Global settings**
- **Check Status** all **Appliances** 
- **View** all **Tokenrecords**
- **View** all **OnBoardedDevices**

The following is only needed if you fetch admin messages:
- **View** all **Admin messages**

## Deployment
- Launch conkolla with `-getOnly` and `whiteListMonitoring` flags.
- Note that there is no Basic Auth on the `/metrics` path even if Basic Auth is enabled.
- Use a reverse proxy such as traefik, nginx etc. to secure access to conkolla from  other networks/Internet.

For a full deployment example see:
* https://github.com/appgate/sdp-prom-monitoring  


## Prometheus scrape config example
```yml
- job_name: trent
    honor_timestamps: true
    metrics_path: /metrics
    scheme: https
    tls_config:
        insecure_skip_verify: true
    relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: target
    - target_label: __address__
      replacement: http://monitoring.packnot.intra:4433
    static_configs:
    - targets:
        - ctl1.packnot.comops
        - ctl2.packnot.comops
        - ctl3.packnot.comops
        - ctl4.packnot.comops
        - ctl5.packnot.comops
        - ctl6.packnot.comops
        - ctl7.packnot.comops
        - ctl8.packnot.comops
        - ctl9.packnot.comops
        - ctl10.packnot.comops
        - ctl11.packnot.comops
        - ctl12.packnot.comops
        - ctl13.packnot.comops
```

- honor time stamps: The metrics from the controller are in form of a pull-gateway, hence not from conkolla. Time stamps are created when data was pulled from controllers.
- All other metrics, the ones reflecting or created by conkolla, do not bare time stamps.


# Conkolla internals

## Proxy scheme (proxy call)
Proxy API to a connected Controller connection follows the scheme:
* `{HTTPS or HTTP}://{HOST}:{PORT}{UPSTREAM HTTP METHOD}/{CONNECTION}/{UPSTREAM RESTCALL} :: {BODY}`

`UPSTREAM HTTP METHOD` The method to be used in the upstream rest call:
* GET, POST, PUT, DELETE

`CONNECTION`: every connection has two references which can be used in Conkolla proxying calls, either 
* a numeric `ID` assigned by Conkolla at login time,  
* or the combined string of `{ControllerURL}{label}`.

`UPSTREAM RESTCALL`
* Appgate API path/resource, without the `/admin` in the path.

`BODY`
* JSON encoded payload if any

`Proxy Method`: Always use `POST` (this covers all upstream calls which are embedded in the proxy rest call).






## Conkolla API calls

### Global

| Path        | Method           | Description  |
| ------------- |:-------------:| -----|
|`/settings`    | GET | Displays Conkolla and runtime information.  |
|`/login` | POST GET | Login form, login JSON params, do login on a Controller.|
|`/apispec` | GET | Displays the on-board apispec (might be outdated, use the linked from the menu for reference).|

### Connection  
These are related to the Appgate connection and might do rest calls to upstream Appgate Controllers.

| Path        | Method           | Description  |
| ------------- |:-------------:| -----|
|`/agc/{connection}/conf`|GET POST| Display, change the connection settings for a Controller.|
|`/agc/{connection}/headers`|GET|Download a file containing the headers for upstream calls for this connection |
|`/agc/{connection}/renewtoken`|GET| Renews the user and entitlement token for this connection. Only supported if no MFA is used and connection is set to 'Auto renew tokens' at login time.|

Note: `/renewtoken` is currently controlled via a `GET` whereas it would be more reasonable with a `PUT`--  this might change in the future.

### Operations specific
Note these calls will always do rest calls to upstream Appgate Controllers.

| Path        | Method           | Description  |
| ------------- |:-------------:| -----|
|`/stats/{connection}/{label}`|GET|Retrieves stats from connected Controller(s). Displaying single stats: `/stats/{connection}/`, where connection is a reference for the connection. Display stats of all connected Controllers: `/stats/0/`. Display the stats for a certain label: `/stats/0/{label}`.|
|`/revoke/{connection}/{label}`|GET|Revokes the tokens for the connection. Revoking single connection: `/revoke/{connection}/`, where connection is a reference for the connection. Revoke all connected Controllers: `/revoke/0/`. Revoke for all Controllers under a certain label: `/revoke/0/{label}`.|
|`/forget/{connection}/{label}`|GET|Revokes and removes the connection. Revoking & remove single connection: `/revoke/{connection}/`, where connection is a reference for the connection. Revoke & remove all connected Controllers: `/revoke/0/`. Revoke & remove for all Controllers under a certain label: `/revoke/0/{label}`.|

Note: the `/stats`, `/revoke` and `/forget` are not typical rest resources and will operate on one or many connections. For now we use method `GET` while this might change in the future.




