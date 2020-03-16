# Disclaimer
© 2020, AppGate. All rights reserved. Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met: (a) redistributions of source code must retain the above copyright notice, this list of conditions and the disclaimer below, and (b) redistributions in binary form must reproduce the above copyright notice, this list of conditions and the disclaimer below in the documentation and/or other materials provided with the distribution. THE CODE AND SCRIPTS POSTED ON THIS WEBSITE ARE PROVIDED ON AN “AS IS” BASIS AND YOUR USE OF SUCH CODE AND/OR SCRIPTS IS AT YOUR OWN RISK. APPGATE DISCLAIMS ALL EXPRESS AND IMPLIED WARRANTIES, EITHER IN FACT OR BY OPERATION OF LAW, STATUTORY OR OTHERWISE, INCLUDING, BUT NOT LIMITED TO, ALL WARRANTIES OF MERCHANTABILITY, TITLE, FITNESS FOR A PARTICULAR PURPOSE, NON-INFRINGEMENT, ACCURACY, COMPLETENESS, COMPATABILITY OF SOFTWARE OR EQUIPMENT OR ANY RESULTS TO BE ACHIEVED THEREFROM. APPGATE DOES NOT WARRANT THAT SUCH CODE AND/OR SCRIPTS ARE OR WILL BE ERROR-FREE. IN NO EVENT SHALL APPGATE BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, RELIANCE, EXEMPLARY, PUNITIVE OR CONSEQUENTIAL DAMAGES, OR ANY LOSS OF GOODWILL, LOSS OF ANTICIPATED SAVINGS, COST OF PURCHASING REPLACEMENT SERVICES, LOSS OF PROFITS, REVENUE, DATA OR DATA USE, ARISING IN ANY WAY OUT OF THE USE AND/OR REDISTRIBUTION OF SUCH CODE AND/OR SCRIPTS, REGARDLESS OF THE LEGAL THEORY UNDER WHICH SUCH LIABILITY IS ASSERTED AND REGARDLESS OF WHETHER APPGATE HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH LIABILITY.

# Kmstool CLI (aws only for now)
The tool is a companion to conkolla. You can use it to encrypt/decrypt plaintext to/from a blob file or simply display its base64 value.

```
  -file string
    	The filename of the blob to be decrypted, or if --secret is given to be stored in
  -key string
    	The KMS key ID.
  -provider string
    	KMS provider. Supported are: aws
  -region string
    	The region for the KMS.
  -secret string
    	secret to encrypt
  -string
    	When encrypting, print only a base64 string of the blob (do not write file).    
 ```
 
Credentials for KMS is as to the aws [aws sdk-for-go](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html), see the section `Specifying Credentials`.
 
 
 | File        | shasum 256           | 
| ------------- |:-------------:| 
| [kmscli.macOS](https://appgate-material.s3.eu-central-1.amazonaws.com/bin/kmscli.macOS)|d7d0c086f36a7446bf77e1f48f664b2598a293a9e6b5039b5ff2563f11a21a0f|
|[kmscli.exe](https://appgate-material.s3.eu-central-1.amazonaws.com/bin/kmscli.exe)|5971263c76f9f85c7dae9fe3df5d6f12b82afddd5af4a04a6ee5ce40b608b16d|
|[kmscli linux](https://appgate-material.s3.eu-central-1.amazonaws.com/bin/kmscli)|ccd1e8fb9d7c3af5f0e68f8df4f13fb8a9b5887186c5ca5b7e310ed8ccee7319|
 
 
