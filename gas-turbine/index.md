# GAS-TURBINE

gas-turbine is a custom REPL environment for manipulating google drive, docs, and sheets from the command-line. It can be used either interactively or to execute scripts written in Javascript. It also has basic assertion capabilties so can function as an automated testing framework for Google Application Scripts.

## Installation

### Prerequisites
gas-turbine is a node application, and needs nodejs and npm in order to run.

```
> npm i -g @impleta/gas-turbine
```

gas-turbine can now be executed by invoking 
```
> gas-turbine
``` 
from the commandline, though it can't do much until we set up authentication.

## Setting up authentication
<details>
<summary>Downloading credential files</summary>

**This is a one-time setup**

1. While logged into your google account, visit https://console.cloud.google.com/cloud-resource-manager and create a new project named `gas-turbine` with ID `gas-turbine`.
1. Visit https://console.cloud.google.com/apis/library/drive.googleapis.com?project=gas-turbine and click `ENABLE`

### Option 1: OAuth2
This option allows scripts executed by Gas Turbine the same access as your primary Google account over all of Google drive. Any folders or files created will be owned by your user account.

1. Visit https://console.cloud.google.com/apis/credentials/consent?project=gas-turbine
1. Select External if you are not a Google Workspace User (i.e, you're using a free account), or Internal if you are and you want to limit access to your organization only. Click `CREATE`
1. Fill in the `App name`, `User support email`, and `Developer contact Information` required fields. Use your own email address for the latter two. Click `SAVE AND CONTINUE`.
1. On the `Scopes` page, click `SAVE AND CONTINUE`. 
1. Add your email (and any other users) on the `Test users` page, then click `SAVE AND CONTINUE`.
1. Click on `Credentials` on the left (or visit https://console.cloud.google.com/apis/credentials?project=gas-turbine)
1. Click `CREATE CREDENTIALS` at the top and select `OAuth client ID`.
1. For `Application type`, select `Web application`
1. For `Name`, enter `GasTurbine Web Client`.
1. Under `Authorized redirect URIs`, click `ADD URI`, and enter `http://localhost:3000/oauth2callback`
1. Select Create.
1. On the `OAuth client created` popup click `DOWNLOAD JSON`. A JSON file will be downloaded to the downloads folder of your browser. Save the file to a safe location. Do not share this or commit this file to source control. 

### Option 2: Service Account
This option can be used to restrict access only to certain shared folders and documents on your Google drive. Any folders or files created will be owned by the Service Account, though your main account will have write access.

1. Visit https://console.cloud.google.com/iam-admin/serviceaccounts?project=gas-turbine
1. Click on `CREATE SERVICE ACCOUNT` at the top.
1. Set the `Service account name` to `gas-turbine-service-account`, and click `CREATE AND CONTINUE`
1. Set the Role as `Owner` under `Grant this service account access to project`
1. Back on the `Service account for project "Gas Turbine"` page, click on the email address.
1. Click on `KEYS` on the top, then `ADD KEY` > `Create new key`
1. Select `JSON` on the dialog that pops up, then `CREATE`
1. A JSON file will be downloaded to the downloads folder of your browser. Move this to a safe location on your computer. Do not share this or commit this file to source control. 
1. Note: You will not be able to download this file again in the future, but you can create a new key should this file be lost.

</details>

## Using gas-turbine
```
> gas-turbine --credentials <path-to-oauth2-or-service-account-keys.json>
Logged in as <email-address>
```

If authenticated via OAuth2, You will be prompted to `Choose an account` on a new browser tab. Click on your email address if already logged into Google (or enter email address and password if prompted). Click `Continue` on the `Google hasn't verified this app` page, and click `Select all`, and click `Continue`. You should now see a page with `Authentication successful! Please return to the console.` message.

### Persisting credentials
```
> gas-turbine --add-credential <path-to-oauth2-or-service-account-keys.json>
Logged in as <email-address>
```

Once added, subseqeunt invocations of `gas-turbine` will use the saved credentials. This can be over-ridden for a given session by specifying `--credential <file-name>` as a parameter.

## Reference

Code snippets in this section all assume that credentials have been persisted using `--add-credential`. The code can either be run in interactive mode or saved as a .js file and executed from the command-line. eg. `gas-turbine path/to/myscript.js`. To use different credentials for a given session, use `gas-turbine --credentials path/to/myscript.js`

### GT

This is the entry point to the built in functionality in Gas Turbine.
 

### Drive
Functionality associated with Google Drive can be accesed via `GT.drive`. The methods allow for creation of folders, copying, moving, and deleting files and folders. 

**Note:** A parameter of type `Resource` in the following examples is either a string representing the ID of a file or folder, or an object representing a file or folder.

---
#### Creating a folder
**createFolder(name: string, parentFolder?: Resource)**

```javascript
 // create a folder in the root
GT.drive.createFolder('New Folder')

 // create a child folder of the speciifed parent folder
GT.drive.createFolder('New Folder', '<parent-folder-or-id>')
```
---
#### Copying a file or folder
**createCopy(resource: Resource)** 
Creates a copy of the speficied file or folder.

```javascript
// create a copy by id
let newFile = await GT.drive.createCopy('<source id string>');

// create a copy by resource.
let newestFile = await GT.drive.createCopy(newFile) 
```
---
#### Get the contents of the root folder
**getMyDriveContents()**
```javascript
let contents = await GT.drive.getMyDriveContents();
```
---
#### Move a file or folder
**move(resource: DriveFile, destinationFolder: Folder)**
```javascript
// Move the specified reosource to the destination folder
let file = GT.drive.getResourceById('<resource id>');
let folder = GT.drive.getResourceById('<destination folder id>');

await GT.drive.move(file, destinationFolder);
```
---
#### Trash a file or folder
**trashResource(resource: Resource)**
```javascript
await GT.drive.trashResource('<resrouce id>');
```
