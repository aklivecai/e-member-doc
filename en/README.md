
# 1.Introduction
This document is used to provide membership solution on the specific customer H&M. In order to have ISV Wosai application work properly with H&M CRM open APIs, please read the following cooperation workflow.


## Step 1 - H&M confirm new features on CRM system.

H&M CRM Team will provide a release deadline for all the new features.

## Step 2 - Qualified ISV Registration for Singature authentication.

Wosai will provide required materials as H&M instructed for review. Once approved, Wosai will get `appid` and `secret` for using H&M CRM API services with singature. 

Singature authentication secure communications between ISV application and H&M CRM servers. To help client applications refresh the secret, `check-in` service will be required. 

Now, Wosai team can be ready to develop application.

## Step 3 - ISV Develop and Test Client Application

Once Wosai decide on a debugging and testing plan, H&M CRM Team will provide relevant technical support resources to guide Wosai team debugging through the development phase.

Wosai will also provide test cases to perform a system testing and supervise the whole process in order to confirm the testing quality.

## Step 4 - Release

Once the whole system passed the test process, we will launch it in the production environment. 

Firstly, H&M CRM will release new features, the CRM service side will be ready.

Secondly, ISV Wosai will release client application and perform online verifications to make sure all the test cases work properly with H&M CRM open API.

To help H&M launch service as smoothly and efficiently as possible, Wosai will also provide on-site support (Mainland China for now)

# Requirements to be confirmed.
 > * Develop a function to validate the secret. 
 > * Develop a function to update the secret. (to chapter 2.1)
 > * Develop a function to pre-allocation card code. (to chapter 5)
 > * Develop a function to receive information of member. (to chapter 3)
 > * Develop a function to query information of member. (to chapter 4)


## 1.1 Network Architecture

![struture](https://raw.githubusercontent.com/Wosai/e-member-doc/master/img/struture.png)

## 1.2 The process of business interaction

According to channel Wechat/Alipay membership card activation process, the ISV will activate card by member-activate interface by pre-allocated card code. So, the ISV application will be response for pre-request card code range from H&M CRM System, allocate unique card code to each member after activation.

The Sequence will be designed into two parts,
- One part is designed for H&M member card pre-allocation.

![image](https://raw.githubusercontent.com/WoSai/e-member-doc/master/img/seq-en-one.png)

- Second part is designed for activation and registeration.

![image](https://raw.githubusercontent.com/WoSai/e-member-doc/master/img/seq-en-two.png)

## 2.Signature verification
In the Internet environment, the ISV must to use HTTPS protocol communication，H&M will provide  application layer signature mechanism. 
All requests must be signed.

### 2.1 Refresh secret
 The secret is usually valid for one month. Before it's expire time, client appliction will check-in to refresh secret.
 - api url:https://member.hm.com/api/sign/v1
 - request type: post
 - parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|appid|Unique identity from ISV|varchar(20)|Y||
|expired|the useful-life of secret|bigint|Y|unit in seconds|


 - example:
 
```javascript
{
    "appid":"2200000001",
    "expired":"86400"
}
```


 - return parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|code|response code|varchar(6)|Y|200 success,otherwise exception|
|message|error message|varchar(6)|N|exist when code is not 200|
|data|new secret|varchar(32)|N|exist when code is 200|



 - example:
 
```javascript
{
    "code":"200",
    "data":{
        "appid":"2200000001",
        “secret”:”34719280830192”,
        "expired":"86400"
    }
}
```

### 2.2 Request Sigature

 - Refer to Wosai UPay API Guide

Web API domain: `https://member.hm.com`

Web API accepts only JSON formatted HTTP requests. Please make sure to add `Content-Type` header and set its value to `application/json` for all requests.

All requests must be `UTF-8` encoded; all responses are `UTF-8` encoded as well.

All requests must be signed as instructed below: 

### 2.3 Request Signature

* Web API uses application layer signature mechanism. The `UTF-8` encoded body (regardless of any formatting) byte stream is used for signature.
* The client `appid` and signature value should be in the `Authorization` header of HTTP requests. It will be validated by Web API service.
* Signature algorithm: `sign = MD5(CONCAT(body + secret))`
* HTTP header: `Authorization: appid + " " + sign`

## 3.Push information of membership card
    Customers get/activate/update their information, E-Member will keep sending this information to H&M and the frequency is real-time/2min/10min/1hour/6hours/12hours/24hours until it get the response "SUCCESS".

 - api url: https://member.hm.com/api/member/notify/v1
 - request type: post
 - parameters format: json
 - parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|notifyTime|time of notify|timestamp|Y|1468780992|
|notifyType|type of notify|varchar(10)|Y|active/update|
|channel|channel|varchar(10)|Y|activate  member must fill in, otherwise not. wechat/alipay|
|id|channel identification|varchar(32)|Y||
|storeId|store identification|varchar(32)|Y|activate  member must fill in, otherwise not|
|name|nickname|varchar(20)|Y||
|birthday|birthday|varchar(10)|Y|1977-04-18|
|mobile|cell-phone number|varchar(13)|Y||
|email|email|varchar(40)|N||
|gender|gender|Integer(1)|N|1:male/2:female/0:unknown|
|headImg|path of picture|varchar(200)|N||
|country|country|varchar(20)|N||
|province|province|varchar(20)|N||
|city|city|varchar(20)|N||
|address|address|varchar(200)|N||
|industry|industry|varchar(40)|N||
|memberId|member id|varchar(40)|N|Merchant can know customer have or not the membership according to customer’s old member id and cell-phone number.|


 - example:
 
```javascript
{
    "notifyTime":"1468780992",
    "notifyType":"active",
    "channel":"wechat",
    "id":"691822b1cab64c879feed51045821ab1",
    "storeId":"da6692c0c1f04f0986d9bf687b7d9908",
    "name":"Tom",
    "birthday":"1977-04-18",
    "mobile":"13112121212",
    "email":"dakang@rmmy.com",
    "gender":"1",
    "headImg":"https://m.wosai.cn/resource/img/dakangshuji/200",
    "country":"China",
    "province":"ZheJiang",
    "city":"Shanghai",
    "address":"XX street XX number",
    "industry":"IT",
    "memberId":"f8a3dae64f74431dbcc68216af82bdb7"
}
```


 - return:



 - example:
 
```javascript
SUCCESS
```

## 4.Query membership card according to card identification
    Customers can get their base information in the page of membership card. And it can also get all information from the H&M CRM Service. The information such as member points, grade, special offers.

 - api url: https://member.hm.com/api/member/query/v1
 - request type: post
 - parameters format: json
 - parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|id|channel identification|varchar(32)|Y|||


 - example:
 
```javascript
{
    "id":"691822b1cab64c879feed51045821ab1"
}
```


 - return parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|code|response code|varchar(6)|Y||
|message|response message|varchar(60)|N||
|data|service entity|{}||
| - id|channel identification|varchar(32)|Y||
| - storeId|store identification|varchar(32)|Y|activate  member must fill in, otherwise not|
| - name|nickname|varchar(20)|Y||
| - birthday|birthday|varchar(10)|Y|1977-04-18|
| - mobile|cell-phone number|varchar(13)|Y||
| - email|email|varchar(40)|N||
| - gender|gender|Integer(1)|N|1：male/2：female/0：unknown|
| - headImg|path of picture|varchar(200)|N||
| - country|country|varchar(20)|N||
| - province|province|varchar(20)|N||
| - city|city|varchar(20)|N||
| - address|address|varchar(200)|N||
| - industry|industry|varchar(40)|N||
| - card|card information|[{}]|Y||
| -  - no|card number|varchar(12)|Y||
| -  - score|card points|bigint|Y||
| -  - level|card grade|varchar(10)|Y||
| -  - expire|the useful-life of card|varchar(10)|Y||
| -  - phone|phone|varchar(20)|N||
| -  - rights|rights|[]|N||
| -  - instructions|instructions|[]|N|||


 - example:
 
```javascript
{
    "code":"200",
    "message":"SUCCESS",
    "data":{
        "id":"691822b1cab64c879feed51045821ab1",
        "name":"Tom",
        "birthday":"1977-04-18",
        "mobile":"13112121212",
        "email":"dakang@rmmy.com",
        "gender":"1",
        "headImg":"https://m.wosai.cn/resource/img/dakangshuji/200",
        "country":"China",
        "province":"ZheJiang",
        "city":"Shanghai",
        "address":"XX street XX number",
        "industry":"IT",
        "card":[{
            "no":"3000 0000 0001",
            "score":"2000",
            "level":"gold",
            "expire":"lifetime",
            "phone":"400-800800",
            "rights":["shopping discount","free to park"],
            "instructions":["card use by youself","card can not share with other people","H&M reserve the right of final interpretation"]
        }]
    }
}
```



## 5.pre-allocated card code

 - api url: https://member.hm.com/api/member/pre/v1
 - request type: post
 - parameters format: json
 - parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|min|min-range|bigint|Y||
|max|max-range|bigint|Y||

 - example:
 
```javascript
{
    "min":"1000001",
    "max":"2000000"
}
```

 - return parameters:

|name|meaning|type|required|desc|
|----|:---|:---|:--:|--------|
|code|response code|varchar(6)|Y|200 success,otherwise exception|
|message|error message|varchar(6)|N|exist when code is not 200|
|data|new secret|varchar(32)|N|exist when code is 200|



 - example:
 
```javascript
{
    "code":"200",
    "message":"SUCCESS",
    "data":"SUCCESS"
}
```
