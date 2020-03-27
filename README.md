# Disclaimer
© 2020, AppGate.  All rights reserved. 
Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met: (a) redistributions of source code must retain the above copyright notice, this list of conditions and the disclaimer below, and (b) redistributions in binary form must reproduce the above copyright notice, this list of conditions and the disclaimer below in the documentation and/or other materials provided with the distribution.
THE CODE AND SCRIPTS POSTED ON THIS WEBSITE ARE PROVIDED ON AN “AS IS” BASIS AND YOUR USE OF SUCH CODE AND/OR SCRIPTS IS AT YOUR OWN RISK.  APPGATE DISCLAIMS ALL EXPRESS AND IMPLIED WARRANTIES, EITHER IN FACT OR BY OPERATION OF LAW, STATUTORY OR OTHERWISE, INCLUDING, BUT NOT LIMITED TO, ALL WARRANTIES OF MERCHANTABILITY, TITLE, FITNESS FOR A PARTICULAR PURPOSE, NON-INFRINGEMENT, ACCURACY, COMPLETENESS, COMPATABILITY OF SOFTWARE OR EQUIPMENT OR ANY RESULTS TO BE ACHIEVED THEREFROM.  APPGATE DOES NOT WARRANT THAT SUCH CODE AND/OR SCRIPTS ARE OR WILL BE ERROR-FREE.  IN NO EVENT SHALL APPGATE BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, RELIANCE, EXEMPLARY, PUNITIVE OR CONSEQUENTIAL DAMAGES, OR ANY LOSS OF GOODWILL, LOSS OF ANTICIPATED SAVINGS, COST OF PURCHASING REPLACEMENT SERVICES, LOSS OF PROFITS, REVENUE, DATA OR DATA USE, ARISING IN ANY WAY OUT OF THE USE AND/OR REDISTRIBUTION OF SUCH CODE AND/OR SCRIPTS, REGARDLESS OF THE LEGAL THEORY UNDER WHICH SUCH LIABILITY IS ASSERTED AND REGARDLESS OF WHETHER APPGATE HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH LIABILITY.   

# Overview
A HTTP API Gateway (aka proxy) for interacting with the AppGate API. The name is a portmanteau for `Controller` and `kolla`, translated from a discovered rune stone in Gothenburg in Sweden meaning: "to look at the Controller".

While the Customer Reliability Team has been working with various integration projects which involved AppGate SDP API, we learned that a tool, which was used over and over, will be handy to rapid prototyping. Eventually "the tool" became more feature rich and we started to believe it is useful for anyone working with AppGate SDP integrations. 

From an educational standpoint, Conkolla allows you to:

* Discover the API against a real system.
* Model or craft API requests and work with real responses.
* [Report data on API responses in form of JSON or CSV](./example_reporting.md)
* Let's you retrieve tokens when you have not yet covered the auth process in your script.
* Can be used as a API gateway connecting to many AppGate Controllers at the same time.
* Gives you discovery with a Web based UI.
* Gives you all the scripting flexibility to use the http rest API (JSON).
* Administrate your AppGate.
* Copy from one collective to another.
* Do multi entity manipulation.
* One time system admin tasks that would be too heavy doing it in the UI.
* Prometheus pull Gateway, one target per connection
* and many more.... 

The AppGate API can be learned and discovered in different ways. One of the common ways is to use the developer tools in a web browser while browsing through the AppGate admin UI, or use existing API Gateway or API Mocker such as postman or prism. 
However, with Conkolla you get another option to choose from.

# Usage
Conkolla can be seen as a proxy, sitting between the user and the AppGate Controller:
![login form](/arch.png)



The user will basically do the following steps when using Conkolla:
1. Launch Conkolla.
2. Login to a AppGate Controller through Conkolla. 
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
$ conkolla --getOnly --whiteListMonitoring --address 0.0.0.0 -conkollaID test-aws-docker -authName maraboux --authPassword maraboux

 {
    "copyright": "AppGate Inc. 2019 (marx)",
    "date": "2019-12-20",
    "directory": "/home/ec2-user/conkolla",
    "certPath": "templates/cert.pem",
    "keyPath": "templates/key.pem",
    "url": "https://0.0.0.0:4433",
    "tls": true,
    "mode": "release",
    "version": "6.4.0 (ha:15c1)",
    "defaultAPIVersion": 11,
    "operatingSystem": "linux",
    "memoryUsageMBbase2": 68.43773651123047,
    "numberGoRoutines": 1,
    "systemTime": "12-20-2019 21:43:18",
    "systemTimeZone": "Local",
    "basicAuthentication": true,
    "conkollaID": "test-aws-docker",
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
    "upstreamHTTPGetOnly": true,
    "whiteListUpstream": [
        "/stats/appliances",
        "/stats/active-sessions",
        "/appliances",
        "/admin-messages/summarize",
        "/admin-message",
        "/license"
    ]
}
{
    "ip-172-1-12-22.eu-north-1.compute.internal": [
        "127.0.0.1/8",
        "::1/128",
        "172.1.12.22/20",
        "fe80::44:aff:fe79:e980/64",
        "172.17.0.1/16",
        "172.18.0.1/16",
        "fe80::42:c0ff:fe5a:41b4/64",
        "fe80::a0ca:4eff:fe0a:d18e/64",
        "fe80::dc23:d5ff:fe80:ca3f/64",
        "fe80::c8c2:ddff:fec2:5dc9/64",
        "fe80::60ae:20ff:fef9:e6d/64",
        "fe80::6427:9fff:fe6c:8959/64",
        "fe80::e828:bff:fe6d:c5b9/64"
    ]
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
  
### API user for prometheus/monitoring
The api account does not need more privileges than:
- **View** all **Appliances**
- **View** all **Global settings**
- **Check Status** all **Appliances** 

The following is only needed if you fetch admin messages:
- **View** all **Admin messages**



## Connect to a Controller
At this stage you will need to have a user with admin permission and privileges to read or write AppGate objects. For more information read:
* [SDP Help > rest api](https://sdphelp.appgate.com/adminguide/rest-apis.html)

Make sure you know where Conkolla is serving:
* look for "url" in start message, or
* use the default `https://localhost:4433`

### Connect to a Controller through web-based UI
* Open a browser and point to the url (look for "url" in start message) 
* Default is [https://localhost:4433/login/](https://localhost:4433/login/)
* Allow the exception for the self signed cert when (if you have not deployed seperate admin UI)


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
### Rest calls to an AppGate Controller
Rest calls towards an AppGate Controller are in the form of:
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


| Upstream Call Method       |       Proxy Call Method     | Description  |Proxy Call Parameters |
| ------------- |:-------------:|-----|----------|
|GET|POST| Get entitites.|`restCall`|
|POST|POST|Create new entities.|`restCall` `objectData`|
|PUT|POST|Change entities.|`restCall` `objectData`|
|DELETE|POST|Delete entities|`restCall`|


**`restCall`**: the upstream rest call, example: _`/entitlements/0c32530f-e15b-4720-a261-acae73c3d417`_.
**`objectData`**: the payload (JSON data in the body).

Check the [examples page](/examples.md).


# Conkolla Reference 
## Login options and controls
Either you browse the login form from the UI or get the possible options/fields from a rest call. Note there might be more options from the rest call than on the web login form:
```shell
curl -k -H "Accept: application/json" https://localhost:4433/login/
```
Exemple from v7.0.0:

```json
{
  "accept_header_suffix": "+json",
  "api_version": 11,
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
    "contentTypeHeader": "",
    "label": "",
    "machineId": "",
    "showToken": "",
    "skipVerifySSL": "",
    "dumpAGResponse": "",
    "autoTokenRenewal": "",
    "renewToken": "",
    "promCollector": "",
    "promTargetName": "",
    "promLabelExporter": "",
    "promLabelEnvironment": "",
    "promLabelGroup": "",
    "promLabelSite": "",
    "promLabelRegion": "",
    "promLabelCountry": "",
    "promLabelCustomer": "",
    "promLabelCostID": "",
    "kmsBlob": "",
    "kmsProvider": "",
    "kmsKey": "",
    "kmsRegion": ""
  },
  "pageinfo": "login page"
}
```
Note that the form data and the JSON data render to the same attributes and some boolean values might be therefore treated as a non-empty value instead as real booleans. Absence signifies `false`where presence of any value signifies `true`


| Param        | Value           | Description  |
| ------------- |:-------------:| -----|
|label|String:text|Labels a connection with the given text. Allows you to connect many times to same AppGate Controller under different label. Labels allows you also to do bulk rest calls and help you sort and find connections. Any space in the will be replaced with `-`.|
|otp|String:number|if the user account uses MFA, enter the next MFA code here. Supports only built-in MFA.|
|acceptHeaderSuffix|`+json` or `+gpg`| Default +json, +gpg is used for downloading backup files. You can change it after login.|
|apiVersion|Integer:number|specifies what AppGate API version to indicate in the upstream headers, usually defaults ok.You can change it after login. |
|machineID|String:UUIDv4| A UUIDv4 string to identify Conkolla as a client ID towards AppGate Controller. None or faulty given, Conkolla generates a random one.|
|showToken|String:`"true"` or `""`| Display AppGate token and, if used, the kms cipher.|
|dumpAGResponse|String:`"true"` or `""`| Conkolla will log the the request and response send to the upstream server. Good for debugging or curious people.|
|autoTokenRenewal|String:`"true"` or `""`| Conkolla will renew the token if it will expire in less than 5 minutes from now. Also, you will be able to force renewal of tokens by the `/renewtoken`call (see below). Auto Renew does not work when using MFA.|
|renewToken|String:`"true"` or `""`| Setting this flag allows you to renew the token for an existing connection with a login request. The fields to identify existing connection are: `controllerURL` and `label`. The required field to renew token is:  `password` (and `otp` if required).|
|promCollector|String:`"true"` or `""`| Setting this flag allows you enable prometheus exporter for the connected collective. It acts like a pull gateway exported, reflecting all metrics of the AppGate collective|
|kmsRegion|String| Region string of the kms.|
|kmsKey|String| KMS key ID.|
|kmsProvider|String:`"aws"`| KMS provider. For now only AWS is supported.|
|kmsBlob|String:`"true"` or `""`| Password is a KMS Blob.|


# Security: API user password security (KMS)
## local secrets handling
Auto-token renewal has until version 7 only been possible due to  a *built in kms* (referred as local), which did the encryption and decryption of the API user's password:
 - Encrypts password/secrets using 256-bit AES-GCM.  
 - This both hides the content of the password and provides a check that it hasn't been altered.

## KMS (external encryption/decryption provider)
Since version 7, support for key management service is available. Currently only AWS is supported. You can use KMS to:
1. login with a aws kms blob as the password (base64 encoded)
1. use kms for password encryption when using auto-renewal

In the second case, the password is encrypted by the  KMS and the cipher blob stored in memory:
- During token renewal, the blob is decrypted and the password (plaintext) is used to retrieve new token from AppGate. 
- After every renewal, if fail or succeeds, password is encrypted and stored again and the password is wiped.

Credentials for KMS is as to the [aws sdk-for-go](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html); from the section *Specifying Credentials*:

1. Environment variables.
2. Shared credentials file.
3. If your application is running on an Amazon EC2 instance, IAM role for Amazon EC2.
4. If your application uses an ECS task definition or RunTask API operation, IAM role for tasks

See also the companion [kmstool cli](./kmstool.md)

## Command line flags

```shell
-address string
    	The IP of the proxy. Use 0.0.0.0 to share on network. Use localhost not to share. (default "localhost")
  -apiversion string
    	Default API version for new connections. (default "11")
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
  -version
    	7.0.0 (release-7:3b9d)
  -whiteListMonitoring
    	White list upstream calls required for monitoring. All Others are forbidden.
```


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
* AppGate API path/resource, without the `/admin` in the path.

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
These are related to the AppGate connection and might do rest calls to upstream AppGate Controllers.

| Path        | Method           | Description  |
| ------------- |:-------------:| -----|
|`/agc/{connection}/conf`|GET POST| Display, change the connection settings for a Controller.|
|`/agc/{connection}/headers`|GET|Download a file containing the headers for upstream calls for this connection |
|`/agc/{connection}/renewtoken`|GET| Renews the user and entitlement token for this connection. Only supported if no MFA is used and connection is set to 'Auto renew tokens' at login time.|

Note: `/renewtoken` is currently controlled via a `GET` whereas it would be more reasonable with a `PUT`--  this might change in the future.

### Operations specific
Note these calls will always do rest calls to upstream AppGate Controllers.

| Path        | Method           | Description  |
| ------------- |:-------------:| -----|
|`/stats/{connection}/{label}`|GET|Retrieves stats from connected Controller(s). Displaying single stats: `/stats/{connection}/`, where connection is a reference for the connection. Display stats of all connected Controllers: `/stats/0/`. Display the stats for a certain label: `/stats/0/{label}`.|
|`/revoke/{connection}/{label}`|GET|Revokes the tokens for the connection. Revoking single connection: `/revoke/{connection}/`, where connection is a reference for the connection. Revoke all connected Controllers: `/revoke/0/`. Revoke for all Controllers under a certain label: `/revoke/0/{label}`.|
|`/forget/{connection}/{label}`|GET|Revokes and removes the connection. Revoking & remove single connection: `/revoke/{connection}/`, where connection is a reference for the connection. Revoke & remove all connected Controllers: `/revoke/0/`. Revoke & remove for all Controllers under a certain label: `/revoke/0/{label}`.|

Note: the `/stats`, `/revoke` and `/forget` are not typical rest resources and will operate on one or many connections. For now we use method `GET` while this might change in the future.




