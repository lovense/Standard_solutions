# Standard Solutions <!-- omit in toc -->

These solutions allow you to control users' Lovense toys by simple HTTPS request.

If you want to integrate with your website or web app, we suggest you use our [Standard JS API](#standard-js-api).

If you integrate a Standard Solution, your users must use Lovense Remote to pair their toys. 
Lovense Remote is available on Google Play and the App Store.

# Contents  <!-- omit in toc -->

<!-- TOC -->


- [Standard API](#standard-api)
	- [Configure the developer dashboard](#configure-the-developer-dashboard)
	- [Find your user toy](#find-your-user-toy)
	- [Command the toy](#command-the-toy)
	  - [By local application](#by-local-application)
	  	- [GetToys Request](#gettoys-request)
		- [Function Request](#function-request)
		- [Pattern Request](#pattern-request)
		- [Preset Request](#preset-request)
		- [Error Codes](#error-codes)
	  - [By server](#by-server)
		- [Function Request](#function-request)
		- [Pattern Request](#pattern-request)
		- [Preset Request](#preset-request)
		- [Server Error Codes](#server-error-codes)

- [Standard JS API](#standard-js-api)
	- [Set your Callback URL](#set-your-callback-url)
	- [Import LAN.JS to your user page](#import-lanjs-to-your-user-page)
	- [Handle the callback data from Lovense Remote](#handle-the-callback-data-from-lovense-remote)
	- [Command the toy](#command-the-toy)
	- [Other methods](#other-methods)
		- [getToys](#gettoys)
		- [getOnlineToys](#getonlinetoys)
		- [isToyOnline](#istoyonline)
		- [openMobileRemote](#openmobileremote)
			



# Standard API
> This API allows any application to access its users Lovense toys from the developer side.

## Configure the developer dashboard
Go to the developer dashboard at https://www.lovense.com/user/developer/info and set your Callback URL.

## Find your user toy
- Copy your developer token from the Lovense developer dashboard.
- Use a POST request to call Lovense server's API.
- Javascript call example:
```
const result = async axios.post('https://api.lovense.com/api/lan/getQrCode',
  {
    token: 'your developer token',  // Lovense developer token
    uid: '11111',  // user ID on your website
    uname: 'user name', // user nickname on your website
    utoken: md5(uid + 'salt')  // This is for your own verification purposes. We suggest you to generate a unique token/secret for each user. This allows you to verify the user and avoid others faking the calls.
    v: 2
  }
)
```
- Java call example
```
String url= "https://api.lovense.com/api/lan/getQrCode";
Map<String, String> requestParameter = new HashMap<String, String>();
//TODO initialize your parameters:
requestParameter.put("token", "{Lovense developer token}");
requestParameter.put("uid", "{user ID on your website}");
requestParameter.put("uname", "{user nickname on your website}");
requestParameter.put("utoken", "{Encrypted user token on your application. This is a security consideration, to avoid others stealing control of the toy.}");
requestParameter.put("v", 2);
HttpPost httpPost = new HttpPost(url);
List<NameValuePair> nameValuePairs = new ArrayList<NameValuePair>();
if (requestParameter != null && !requestParameter.isEmpty()) {
  Set<String> keys = requestParameter.keySet();
  for (String key : keys) {
    nameValuePairs.add(new BasicNameValuePair(key, requestParameter.get(key)));
  }
}
httpPost.setEntity(new UrlEncodedFormEntity(nameValuePairs, "utf-8"));
```
- You will receive a json response such as:
```
{
   code: 0
   message: "Success"
   result: true
   data: {
     "qr": "https://test2.lovense.com/UploadFiles/qr/20220106/xxx.jpg", // QR code picture
     "code": "xxxxxx"
   }
}
```

- After the user scans the QR code with the Lovense Remote app, the app will invoke the Callback URL you've provided in the developer dashboard. The Lovense server is no longer required. All communications will go from the app to your server directly.
The Lovense Remote app will post the following json to your server:
```
{
  "uid": "xxx",
  "appVersion": "4.0.3",
  "toys": {
    "xxxx": {
      "nickName": "",
      "name": "max",
      "id": "xxxx",
      "status": 1
    }
  },
  "wssPort": "34568",
  "httpPort": "34567",
  "wsPort": "34567",
  "appType": "remote",
  "domain": "192-168-1-44.lovense.club",
  "utoken": "xxxxxx",
  "httpsPort": "34568",
  "version": "101",
  "platform": "android"
}
```


## Command the toy
> Note: iOS Remote 5.1.4+, Android Remote 5.1.1+, or PC Remote 1.5.8+ is required.

### By local application
If the user's device is in the same LAN environment, a POST request to Lovense Remote can trigger a toy response. In this case, your server and Lovense's server are not required.

If the user uses the mobile version of Lovense Remote app, the `domain` and `httpsPort` are accessed from the callback information. If the user uses Lovense Remote for PC, the `domain` is `127-0-0-1.lovense.club` and the `httpsPort` is `30010`

With the same command line, different parameters will lead to different results. Requests are as follows:<br> 

|    |   |
|---|----|
|**API URL**| https://{domain}:{httpsPort}/command|
|**Request protocol**| HTTPS Request|
|**Method**| POST|
|**Request Content Type**| application/json|
|**Response Format**| JSON|

#### GetToys Request

**Parameters:***
|   Name   |    Description    |  Type  |  Note  |  Required  |
| -------- | ----------------- | ------ | ------ | ---------- |
| command  | Type of request   | String	|    /	 |	yes       |

Request example:

```
{
  "command": "GetToys"
}
```
Response example:
```
{
  "code": 200,
  "data": {
    "toys": "{  \"fc9f37e96593\" : {    \"id\" : \"fc9f37e96593\",    \"status\" : \"1\",    \"version\" : \"\",    \"name\" : \"nora\",    \"battery\" : 100,    \"nickName\" : \"\"  }}",
    "platform": "ios",
    "appType": "remote"
  },
  "type": "OK"
}
```
#### Function Request

**Parameters**
|	Name	   		| Description					|	Type	|	Note											  |	Required |
|-------------------|:------------------------------|:---------:|:----------------------------------------------------|----------|	
|	command	   		| Type of request				|	String	|	/												  |	yes		 |
|	action	   		| Control the function and strength of the toy	|	String		|Actions can be Vibrate, Rotate, Pump or Stop.	<br>Use Stop to stop the toy’s response. <br>`Range: Vibrate:0 \~ 20` <br>`Rotate: 0 \~ 20` <br>`Pump:0 \~ 3` |	yes 	 |								
|   timeSec			|	Total running time			|	double	|	0 = indefinite length <br> Otherwise, running time should be greater than 1. |	yes		 |
|loopRunningSec		| Running time					|	double	|	Should be greater than 1						  |	no		 |
|loopPauseSec		| Suspend time					|	double	|	Should be greater than 1						  |	no		 |
|toy				| Toy ID						|	string	|	Optional, if you don’t include this, it will be applied to all toys		|	no		 |
|apiVer				|	The version of the request	|	int		|	Always use 1									  |	yes		 |


Request example:
```
// Vibrate toy ff922f7fd345 at 16th strength, run 9 seconds then suspend 4 seconds. It will be looped. Total running time is 20 seconds.
{
  "command": "Function",
  "action": "Vibrate:16",
  "timeSec": 20,
  "loopRunningSec": 9,
  "loopPauseSec": 4,
  "toy": "ff922f7fd345",
  "apiVer": 1
}
```
```
// Vibrate 9 seconds at 2nd strength
// Rotate toys 9 seconds at 3rd strength
// Pump all toys 9 seconds at 4th strength
// For all toys, it will run 9 seconds then suspend 4 seconds. It will be looped. Total running time is 20 seconds.
{
  "command": "Function",
  "action": "Vibrate:2,Rotate:3,Pump:4",
  "timeSec": 20,
  "loopRunningSec": 9,
  "loopPauseSec": 4,
  "apiVer": 1
}
```
```
// Stop all toys
{
  "command": "Function",
  "action": "Stop",
  "timeSec": 0,
  "apiVer": 1
}
```
#### Pattern Request

**Parameters**


|Name	|	Description|	Type	|Note  |	Required|
|:---	|:-------------|:------------|:----|----------|
|command|	Type of request	|String|	/	|yes|
|rule	|"V:1;F:vrp;S:1000#" <br> V:1; Protocol version, this is static; <br> F:vrp; Features: v is vibrate, r is rotate, p is pump,this should match the strength below; <br> S:1000; Intervals in Milliseconds, should be greater than 100.	|string	|The strength of r and p will automatically correspond to v.|	yes|
|strength|	The pattern <br> For example: 20;20;5;20;10	|string|	No more than 50 parameters. Use semicolon ; to separate every strength.	|yes|
|timeSec|	Total running time|	double|	0 = indefinite length <br> Otherwise, running time should be greater than 1.	|yes|
|toy|	Toy ID	|string	|Optional, if you don’t include this, it will apply to all toys	|no|
|apiVer	|The version of the request	|int	|Always use 1	|yes|


Request example
```
// Vibrate the toy as defined. The interval between changes is 1 second. Total running time is 9 seconds.
{
  "command": "Pattern",
  "rule": "V:1;F:v;S:1000#",
  "strength": "20;20;5;20;10",
  "timeSec": 9,
  "toy": "ff922f7fd345",
  "apiVer": 1
}
```
```
// Vibrate the toys as defined. The interval between changes is 0.1 second. Total running time is 9 seconds.
// If the toys include Nora or Max, they will automatically rotate or pump, you don't need to define it.
{
  "command": "Pattern",
  "rule": "V:1;F:vrp;S:100#",
  "strength": "20;20;5;20;10",
  "timeSec": 9,
  "apiVer": 1
}
```

### Preset Request

**Parameters**

|Name	|Description|	Type|	Note|	Required|
|:------|:----------|:------|:------|-----------|
|command	|Type of request|	String	|/	|yes|
|name	|Preset pattern name	|string	|We provide four preset patterns in the Lovense Remote app: pulse, wave, fireworks, earthquake	|yes|
|timeSec	|Total running time	|double	|0 = indefinite length <br> Otherwise, running time should be greater than 1.	|yes|
|toy|	Toy ID	|string	|Optional, if you don’t include this, it will be applied to all toys	|no|
|apiVer	|The version of the request	|int|	Always use 1	|yes|


Request example

```
// Vibrate the toy with pulse pattern, the running time is 9 seconds.
{
  "command": "Preset",
  "name": "pulse",
  "timeSec": 9,
  "toy": "ff922f7fd345",
  "apiVer": 1
}
```
Response example

```
{
  "code": 200,
  "type": "ok"
}
```

#### Error codes

|Code	|	Message|
|:------|:-------|
|500|	HTTP server not started or disabled|
|400|	Invalid Command|
|401	|Toy Not Found|
|402	|Toy Not Connected|
|403	|Toy Doesn't Support This Command|
|404	|Invalid Parameter|
|506	|Server Error. Restart Lovense Connect|




### By server

If your application can’t establish a LAN connection to the user’s Lovense Remote app, you can use the Server API to connect the user’s toy.

> ⚠️ If you are using Lovense Remote for PC, you need to enter a code to establish connection. Use the code generated alongside the QR code in step 2 above.


Requests are as follows:<br>

|    |   |
|---|----|
|**API URL**| https://{domain}:{httpsPort}/command|
|**Request protocol**| HTTPS Request|
|**Method**| POST|
|**Request Content Type**| application/json|
|**Response Format**| JSON|



#### Function Request

**Parameters:**

|Name	|Description	|Type	|Note	|Required|
|:------|:--------------|:------|:------|--------|
|token	|Your developer |token	|string	|	yes|
|uid	|Your user’s ID	|string	|To send commands to multiple users at the same time, add all the user IDs separated by commas. The toy parameter below will be ignored and the commands will go to all user toys by default.	|yes|
|command|	Type of request|	String	|/	|yes|
|action	|Control the function and strength of the toy	|string	|Actions can be Vibrate, Rotate, Pump or Stop. Use Stop to stop the toy’s response. <br> Range: <br>```Vibrate:0 ~ 20``` <br> ```Rotate: 0~20``` <br> ```Pump:0~3```	|yes|
|timeSec	|Total running time	|double	|0 = indefinite length <br>Otherwise, running time should be greater than 1.	|yes|
|loopRunningSec	|Running time	|double|	Should be greater than 1	|no|
|loopPauseSec|	Suspend time	|double|	Should be greater than 1	|no|
|toy	|Toy ID|	string	|Optional, if you don’t include this, it will be applied to all toys	|no|
|stopPrevious	|Stop all previous commands and execute current commands	|int	|Optional, Default: `1`, If set to `0` , it will not stop the previous command.|	no|
|apiVer	|The version of the request	|int	|Always use 1	|yes|

Request example

```
// Vibrate toy ff922f7fd345 at 16th strength, run 9 seconds then suspend 4 seconds. It will be looped. Total running time is 20 seconds.
{
  "token": "FE1TxWpTciAl4E2QfYEfPLvo2jf8V6WJWkLJtzLqv/nB2AMos9XuWzgQNrbXSi6n",
  "uid": "1132fsdfsd",
  "command": "Function",
  "action": "Vibrate:16",
  "timeSec": 20,
  "loopRunningSec": 9,
  "loopPauseSec": 4,
  "apiVer": 1
}
```
```
// Vibrate 9 seconds at 2nd strength
// Rotate toys 9 seconds at 3rd strength
// Pump all toys 9 seconds at 4th strength
// For all toys, it will run 9 seconds then suspend 4 seconds. It will be looped. Total running time is 20 seconds.
{
  "token": "FE1TxWpTciAl4E2QfYEfPLvo2jf8V6WJWkLJtzLqv/nB2AMos9XuWzgQNrbXSi6n",
  "uid": "1132fsdfsd",
  "command": "Function",
  "action": "Vibrate:2,Rotate:3,Pump:4",
  "timeSec": 20,
  "loopRunningSec": 9,
  "loopPauseSec": 4,
  "apiVer": 1
}
```

#### Pattern request

If you want to change the way the toy responds very frequently you can use a pattern request. To avoid network pressure and obtain a stable response, use the commands below to send your predefined patterns at once.

**Parameters:**

|Name	|Description	|Type	|Note	|Required|
|:------|:--------------|:------|:------|--------|
|token	|Your developer token	|string	|	yes|
|uid	|Your user’s ID	|string		|yes|
|command	|Type of request	|String|	/	|yes|
|rule	|"V:1;F:vrp;S:1000#" <br> V:1; Protocol version, this is static; <br> F:vrp; Features: v is vibrate, r is rotate, p is pump,this should match the strength below; <br> S:1000; Intervals in Milliseconds, should be greater than 100.	|string	|The strength of r and p will automatically correspond to v.|	yes|
|strength|	The pattern <br> For example: 20;20;5;20;10	|string|	No more than 50 parameters. Use semicolon `; `to separate every strength.	|yes|
|timeSec|	Total running time|	double|	0 = indefinite length <br> Otherwise, running time should be greater than 1.	|yes|
|toy|	Toy ID	|string	|Optional, if you don’t include this, it will apply to all toys	|no|
|apiVer	|The version of the request	|int	|Always use 1	|yes|

Request example

```
// Vibrate the toy as defined. The interval between changes is 1 second. Total running time is 9 seconds.
{
  "token": "FE1TxWpTciAl4E2QfYEfPLvo2jf8V6WJWkLJtzLqv/nB2AMos9XuWzgQNrbXSi6n",
  "uid": "1ads22adsf",
  "command": "Pattern",
  "rule": "V:1;F:v;S:1000#",
  "strength": "20;20;5;20;10",
  "timeSec": 9,
  "apiVer": 1
}
```
```
// Vibrate the toys as defined. The interval between changes is 0.1 second. Total running time is 9 seconds.
// If the toys include Nora or Max, they will automatically rotate or pump, you don't need to define it.
{
  "token": "FE1TxWpTciAl4E2QfYEfPLvo2jf8V6WJWkLJtzLqv/nB2AMos9XuWzgQNrbXSi6n",
  "uid": "1ads22adsf",
  "command": "Pattern",
  "rule": "V:1;F:vrp;S:100#",
  "strength": "20;20;5;20;10",
  "timeSec": 9,
  "apiVer": 1
}
```

#### Preset request

**Parameters:**

|Name	|Description	|Type	|Note	|Required|
|:------|:--------------|:------|:------|--------|
|token	|Your developer token	|string	|	| |yes|
|uid	|Your user’s ID	|string		| ||yes|
|command	|Type of request	|String	|/	|yes|
|name	|Preset pattern name	|string	|We provide four preset patterns in the Lovense Remote app: pulse, wave, fireworks, earthquake|	yes|
|timeSec	|Total running time	|double	|0 = indefinite length <br> Otherwise, running time should be greater than 1.	|yes|
|toy	|Toy ID	|string	|Optional, if you don’t include this, it will be applied to all toys	|no|
|apiVer	|The version of the request	|int	|Always use 1	|yes|

Request example

```
// Vibrate the toy with pulse pattern, the running time is 9 seconds.
{
  "token": "FE1TxWpTciAl4E2QfYEfPLvo2jf8V6WJWkLJtzLqv/nB2AMos9XuWzgQNrbXSi6n",
  "uid": "1adsf2323",
  "command": "Preset",
  "name": "pulse",
  "timeSec": 9,
  "apiVer": 1
}
```
```
{
  "result": true,
  "code": 200,
  "message": "Success"
}
```

#### Server error codes

|Code	|Message|
|:------|:------|
|200	|Success|
|400	|Invalid command|
|404	|Invalid Parameter|
|501	|Invalid token|
|502	|You do not have permission to use this API|
|503	|Invalid User ID|
|507	|Lovense APP is offline|



#Standard JS API


# Standard JS API

This solution allows you to control Lovense toys through your webpage.

Your users must use the Lovense Remote mobile app.

>⚠️ iOS Remote v5.2.5+ or Android Remote v5.2.1+ is required.

>⚠️ The user's devices must be in the same LAN.

## Set your Callback URL
Go to the developer dashboard at https://www.lovense.com/user/developer/info and set your Callback URL.

## Import LAN.JS to your user page
Add the LAN.JS to your user's page:
```
<script src="https://api.lovense.com/api/lan/v2/lan.js"></script>
```

## Handle the callback data from Lovense Remote
You can listen to the callback from Lovense Remote as below:
```
// the path should be the same as you set in the Lovense developer dashboard
@RequestMapping(value = "/api/lovense/callback")
public @ResponseBody
MessageResponse callback(@RequestBody Map body) {
  //TODO
  //send the callback data to the user's page and call `lovense.setConnectCallbackData` there
  return new MessageResponse(true, "success");
}
```
Call `lovense.setConnectCallbackData` to set when you receive the callback data from Lovense Remote.
```
lovense.setConnectCallbackData(callbackData)
```
>⚠️ If the user uses Lovense Connect for PC, this step can be skipped because a QR code is not required.

## Command the toy(s)
Send commands/vibrations to Lovense toy(s) by calling `lovense.sendCommand(parameters)`.

For the parameters list, see [Command the toy(s)](https://developer.lovense.com/#step-3-command-the-toy-s-1)

Example:
```
lovense.sendCommand({
  command: "Function",
  action: "Vibrate:16",
  timeSec: 20,
  loopRunningSec: 9,
  loopPauseSec: 4,
  toy: "ff922f7fd345",
  apiVer: 1,
})
```

## Other methods

### getToys
Get all toys, it will return all toys as an array

```lovense.getToys()```

### getOnlineToys
Get all online toys, it will only return the connected toys as an array

```lovense.getOnlineToys()```

### isToyOnline
Determine whether the toy is connected, it will return true as long as there is a toy connected.

```lovense.isToyOnline()```

### openMobileRemote
Open Lovense Mobile Remote.

```lovense.openMobileRemote()```
