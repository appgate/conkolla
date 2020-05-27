# Disclaimer
© 2019, AppGate.  All rights reserved. 
Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met: (a) redistributions of source code must retain the above copyright notice, this list of conditions and the disclaimer below, and (b) redistributions in binary form must reproduce the above copyright notice, this list of conditions and the disclaimer below in the documentation and/or other materials provided with the distribution.
THE CODE AND SCRIPTS POSTED ON THIS WEBSITE ARE PROVIDED ON AN “AS IS” BASIS AND YOUR USE OF SUCH CODE AND/OR SCRIPTS IS AT YOUR OWN RISK.  APPGATE DISCLAIMS ALL EXPRESS AND IMPLIED WARRANTIES, EITHER IN FACT OR BY OPERATION OF LAW, STATUTORY OR OTHERWISE, INCLUDING, BUT NOT LIMITED TO, ALL WARRANTIES OF MERCHANTABILITY, TITLE, FITNESS FOR A PARTICULAR PURPOSE, NON-INFRINGEMENT, ACCURACY, COMPLETENESS, COMPATABILITY OF SOFTWARE OR EQUIPMENT OR ANY RESULTS TO BE ACHIEVED THEREFROM.  APPGATE DOES NOT WARRANT THAT SUCH CODE AND/OR SCRIPTS ARE OR WILL BE ERROR-FREE.  IN NO EVENT SHALL APPGATE BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, RELIANCE, EXEMPLARY, PUNITIVE OR CONSEQUENTIAL DAMAGES, OR ANY LOSS OF GOODWILL, LOSS OF ANTICIPATED SAVINGS, COST OF PURCHASING REPLACEMENT SERVICES, LOSS OF PROFITS, REVENUE, DATA OR DATA USE, ARISING IN ANY WAY OUT OF THE USE AND/OR REDISTRIBUTION OF SUCH CODE AND/OR SCRIPTS, REGARDLESS OF THE LEGAL THEORY UNDER WHICH SUCH LIABILITY IS ASSERTED AND REGARDLESS OF WHETHER APPGATE HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH LIABILITY.   

# Examples
Conkolla has a API spec on board and parses it for quick access: If you open the `help` page in the Conkolla UI, it will generate for every connection you have a list of clickable API examples, primitively generated from the on-board spec (which might be out-dated, but good as basic examples). The on board API spec does also feed the _autocomplete_ for the editor entity entry field; check [apispec](https://localhost:4433/apispec) to display the file and version.

 * The link to the Reference API in the top menu will always point to the latest version.
 * Some examples use the developer mode of your browser, more instruction can be found [here](./developermode.md).
 
 
## Login through curl (command line)

```shell
curl -s -H "Accept: application/json" -X POST \
-kL https://localhost:4433/login \
--data '{"username":"admin", "password":"secret sauce","ControllerURL":"cc1.packnot.com", "apiVersion":9, "skipVerifySSL":"true","label":"dev"}'
```

Response example:
```json
 {
  "EntitlementTokenExpires": "2019-04-10T10:24:31.246Z",
  "authBody": {
    "machineId": "0e9076cd-e09e-4db7-6d24-480196700a42",
    "providerName": "local",
    "username": "admin",
    "password": ""
  },
  "connectionID": "1",
  "loginParameters": {
    "ControllerURL": "cc1.packnot.com",
    "ControllerPort": 444,
    "username": "admin",
    "password": "removed",
    "otp": "",
    "providerName": "local",
    "apiVersion": 9,
    "acceptHeaderSuffix": "+json",
    "contentTypeHeader": "application/json",
    "label": "dev",
    "machineId": "0e9076cd-e09e-4db7-6d24-480196700a42",
    "hideToken": "",
    "skipVerifySSL": "true",
    "dumpAGResponse": "",
    "autoTokenRenewal": ""
  },
  "message": "Successfully connected."
```

## Example query in the Conkolla UI
Use the following query parameters to search for entities: `query`. Example `/entitlements?query=dns`. 
![ui_query](./ui_query.png)

# Browser developer tools: wrappers for multi entity manipulation
With the developer tools (use crhome, at least ECMAScript2018) you can easily do multi entity manipulation from/to/on same/other connected controller. See the [reference documentation for developer mode](./developermode.md).

![use js console for multi entity edit](./console_multi.png)


## Example: Add a rsyslog destination to all appliances in the collective

1. Select the records you need to work on (here all appliances): *GET* `/appliances`. Result is displayed in editor and available as an array of records (appliances) in the variable `dataRecords`.
2. Open devleoper tools > console
3. Write the code to update the value
4. Run the code in the console.

You can use the built in `putEntity(conID, entity, record)` function to update existing appliances entities:

```javascript

newRsyslogDest = { // the new syslog config to be added
  "destination": "@@loc.logs.packnot.com:5555",
  "selector": "# certificate files - just CA for a client\n$DefaultNetstreamDriverCAFile /etc/ssl/certs/rsyslog.pem\n \n# set up the action\n$DefaultNetstreamDriver gtls # use gtls netstream driver\n$ActionSendStreamDriverMode 1 \n$ActionSendStreamDriverAuthMode anon # server is NOT authenticated\n:msg, contains, \"[AUDIT]\"",
  "template": "%msg:9:$%\\n"
};

// append config to rsyslog destinations
for (let record of dataRecords){ // dataRecords contains the result of the last GET request from UI
  var entity = "appliances"; // the entities we operate on
  var conID = 1; // the controller conection
  var dests = record["rsyslogDestinations"];
  dests.push(newRsyslogDest);
  record["rsyslogDestinations"] = dests;
  //update the entitiy
  let result = putEntity(conID, entity, record)
  console.log(JSON.stringify(result, null, 4))) 
 
}

```

## Examples of manipulating entities
### Update notes field on all entitlements, using UI and JS console.

*1. load all the entities into the editor: _GET_ `/entitlements`.*

The results is stored in `dataRecords`

*2. Write the modify function:*

```javascript 
function update(record){
  record["notes"] += "another confusing note!";
  return record; 
}
```
*3. Write the update code*

```javascript
for (let record of dataRecords){
   putEntity(1, "entitlements", update(record));
}
```

*3.1 An alternativ update code: Use `getAndUpdatEntity` function* 

You would use this function when the ID is referenced from another entity type, for example from a policy. It is shown as an example here

```javascript
for (let record of dataRecords){
  getAndUpdateEntity(1,"entitlements",record["id"], update);
}
```

*3.2 Change by hand: when you manually changed data in your window*

The `dataRecords` is only updated when you use the UI for the request. Now, if you by hand changed data in the editor, the new values will not update the content of `dataRecords` -- it will always contain the original.

If you want to use the current data from the editor, then use `entitiesFromEditor`:

```javascript
entitiesFromEditor(1, "entitlements", putEntity)
```

### Update notes field on all entitlements, using only console
Note you need to be on the editor page for the necessary JS code (used function etc) to be loaded loaded.

This code goes as to the above steps, with the difference of doing all the steps programmatically. Here the code:

```javascript
function update(record){
  record["notes"] += "another confusing note!";
  return record; 
}

// now this is the major difference: block until the promise is resolved.
var result = await getEntity(1,"entitlements");

for (let record of result.data){
   putEntity(1, "entitlements", update(record));
}
```



## Update global settings on many controllers

```javascript
function updateGlobalSettings(gs){
  gs["claimsTokenExpiration"] = 720;
  return gs
}

for (var i=1; i< 21; i++) {
  getAndUpdateEntity(i,"global-settings", "", updateGlobalSettings);

}

```

## Copy entities from one system to another
This example demonstrates how you can copy entities from one to another system in a controlled and assisted way. The below example copies entitlements by using the in-editor loaded entitlements:

- Log-in to both controllers `src` and `dst`, assure you have the privileges to write entitlements on dst.
- Open the Chrome Developer console, then open the editor with `/entitlements` for the source system.
- Look through the entitlements, check if any adjustments will be needed:
  -  Mapping attribute values from src to dst (e.g site id, conditions etc)
  -  Dropping/adding any attributes if src and dst do not have same entity scheme due to different AppGate versions; however you should always run src and dst on same version, but sometimes this is not an option.

Either you do changes directly in the editor window by hand, or do it programmatically (as in the below example). If you do all changes in the window, remember to use `entitiesFromEditor` instead of `postEntity`.

```javascript
var destination = 2 // connection ID for the destination
site_mappings = {"8a4add9e-0e99-4bb1-949c-c9faf9a49ad4":"8a4add9e-0e99-4bb1-949c-c9faf9a49ad4"}

for (let record of dataRecords){
  // adjust site mapping
  record["site"] = site_mappings[record["site"]]
  // add tag
  record["tags"].push("imported")
  // remove any admin roles
  delete record["administrativeRoleLinks"] 
  delete record["administrativeRoles"]
  // note you could instead create a new entitlement with the minimal required fields, 
  // and add any fields on the fly.

  // Add new entity. Use await to let the promise be resolve (throttles the nr of requests).
  await putEntity(destination, "entitlements", record)
}
``` 

Alternatively use pure javascript, e.g not using the entitlements loaded into the editor:
```javascript
var src = X // where X is the nr or name of the connection with the source
var dst = Y // where Y is the nr or name of the connection with the destination

getEntity(src, "entitlements")
for (let record of consoleDataRecords) {
  // 1. change/create new entity
  // 2. put entity
}
```


## Update NTP servers on all appliances in a cluster for many clusters
This is an example drawn from our training environment where we have multiple clusters, where one cluster is given per training attendee:

```javascript
cons = [1,2,3,4,5,6,7,8,8,10,11,12,13]

servers =  [{"hostname": "0.ubuntu.pool.ntp.org"},{"hostname": "1.ubuntu.pool.ntp.org"},{"hostname": "2.ubuntu.pool.ntp.org"},{"hostname": "3.ubuntu.pool.ntp.org"}]

function update(r){
    r["ntp"]["servers"] = servers
    return r
}

for (let con of cons){
    console.log(con)
    var result = await getEntity(con, "appliances")
    
    for (let record of result.data){
        putEntity(con, "appliances", update(record))
     }
}

```
## Clean-up user records
When a user is terminated (the identity shall be removed from the IdP), clean-up scripts will usually take care of removing all the parts and pieces for that user. Admins can hook in and use for example power-shell scripts. In one of the clean-up stages you would have then a script which does the clean-up in AppGate for that user:

- free the license
- delete all on boarded devices
- remove otp seeds

Sometimes you need to run a one-off to clean out a user in AppGate. This code will do it:

```javascript
// config
let username = "Raymond.Reddington" // the user for which the clean-up is done
let conId = 1
entitiesKey = [ { "key": "userDistinguishedName", "path": "license/users"},
                { "key": "distinguishedName", "path": "on-boarded-devices"},
                { "key": "userDistinguishedName", "path": "otp/seeds"}]

// setup
var reExactly = new RegExp("^" + username + "$");
let candidates = []

// get the possible entities 
const workEntities = async (entity) =>{
  console.log("get entities: " +  entity.path)
  let result = await Promise.all([getEntity(conId, entity.path +  "?query=" + username)])
  // filter for exact match
  for (let rec of result[0].data){
    console.log(rec)
    if (rec.username && rec.username.match(reExactly)){
      candidates.push({"key": rec[entity.key], "path": entity.path})
    }
  }
}
// get and work out the entities to be deleted
await Promise.all(entitiesKey.map(workEntities))

// just review if needed on console log
function printCandidates(){
  console.log("Candidates in list:")
  candidates.map( (candidate) => console.log(candidate))
}

// deletes the entity stored in candidates
function cleanup(candidate){
  console.log("delete: " + candidate.path + "::" + candidate.key)
  deleteEntity(conId, candidate.path, candidate.key)   
}

// do delete
printCandidates()
candidates.map(cleanup)

```




# Administration Examples

## Upgrade an appliance (using the Conkolla UI)
The following example depicts how you can upgrade an appliance from the conkolla UI.

1. First find the ID of the appliance you want to upgrade: *GET* `/appliances`
2. Get the download link for the update zip file. Make sure the appliance can download it.
3. Init the upgrade:

*POST* `/appliances/<appliance-ID>/upgrade`
```json
{
  "imageUrl": "https://user:pass@sdpdownloads.cyxtera.com/AppGate-SDP-4.2/bin/AppGate-SDP-4.2.3.img.zip"
}
```
Response should be: "202 ACCEPTED"

Status can only be checked by pulling: *GET* `/<appliance-ID>/upgrade`. Note that there can be error messages during different states of the upgraded such as 
database connection or time-outs during service restarts etc. All these are currently natural for an upgrade.

Check the status of the appliance to verify if all went as should.

## Create Backup and Download Backup File (command line)
Note: you can do basically follow the steps and do them in Conkolla UI, which is simpler.

1. Get the appliance ID.
2. Initiate an appliance backup.
3. Poll status of backup.
4. Download the backup.



1. Get the appliance ID of the controller:
 
 ```shell
 curl -k -H "Accept: application/json" https://localhost:4433/get/6/appliances
 ```
 
 ```json
 :
 "controller": {
        "database": {
          "location": "internal"
        },
        "enabled": true
      },
      :
      "id": "f7ab1be3-8d32-4027-bcf3-91da6f0c0560",
 
 ```
 

2. Initiate an appliance backup using the appliance ID
```shell
curl -k -H "Accept: application/json" -X POST --data \
'{"objectData":"{\"logs\": true,\"audit\": true,\"opt\": true}",\
"restCall":"/appliances/f7ab1be3-8d32-4027-bcf3-91da6f0c0560/backup"}' \
"https://localhost:4433/post/6/appliances"
```
**objectData**: the body content. Note this must be a inner JSON string, hence it is between quotes. **restCall**: the upstream restcall. Translate to Conkolla as:

```log
[AGC] [pix-1.packnot.com] [dev] 2019/05/21 11:21:34 Upstream rest call: /appliances/f7ab1be3-8d32-4027-bcf3-91da6f0c0560/backup
[AGC] [pix-1.packnot.com] [dev] 2019/05/21 11:21:34 Upstream body: {"logs": true,"audit": true,"opt": true}
```

Response:
```json
{
  "id": "dbb45e6c-22fb-11e9-ab06-065aaf365c98"
}
```

3. Check the status of the backup
``` shell
curl -k -H "Accept: application/json" \
https://localhost:4433/get/6/appliances/f7ab1be3-8d32-4027-bcf3-91da6f0c0560/backup/dbb45e6c-22fb-11e9-ab06-065aaf365c98/status
```

Example:
```json
{
  "status": "done"
}
```

4 Fetch the backup file
``` shell
 curl -o backup.gpg -H "Accept: application/json" -k https://localhost:4433/get/1/appliances/f7ab1be3-8d32-4027-bcf3-91da6f0c0560/backup/dbb45e6c-22fb-11e9-ab06-065aaf365c98
```
Note: the `AcceptHeader suffix` does automatically change from the suffix `+json` to `+gpg`. If you implement your own handler, you will need to take this into account.

```shell
-rw-r--r--   1 ron  staff   118M 21 Mai 11:50 backup.gpg
```
