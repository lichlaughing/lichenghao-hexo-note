---
title: 接口测试工具 Postman
categories: interfacetest
tags:
  - interfacetest
keywords: interfacetest
abbrlink: 1772a845
date: 2022-04-07 13:50:00
---
官网地址：https://www.postman.com/ 免费版本的足够日常使用，建议注册个账号，这样可以多端数据同步。

官方的手册：https://learning.postman.com/docs/getting-started/introduction/

发送各种各样的请求，直接参考官方的手册即可，记录些感觉比较有用的使用方式。

## 接口环境(Environments)

这里分为全局环境和自定义环境，通常会将一些接口通用的数据放在环境变量上，例如登录地址，用户名，密码，TOKEN等，然后通过Postman取环境参数的方式：{ {param} }，可以拿到我们定义好的环境变量。如下所示定义全局变量和定义环境变量。

![](https://blog.lichenghao.cn/upload/2022/07/20210619154003.png)

这样的话，我们就可以在发送请求的时候，使用环境变量了，如下所示：！记得右上角选择需要的环境。

![](https://blog.lichenghao.cn/upload/2022/07/20210619154643.png)


在请求Body中可以使用环境变量，同样在请求Header里面也可以同样的方式使用。



## Tests给环境变量赋值

可以在请求Tests里面，通过解析请求的响应结果，给环境变量赋值。例如登录成功后，获取响应结果的Token，放到当前环境变量中，其他请求就直接使用这个Token就可以了。下面是一个解析响应结果的示例：

假如响应结果如下：

```properties
{
    "status": true,
    "data": {
        "roleName": "admin",
        "token": "0d85893eb344e5a78afdd73ff51552c1f106f9dfa"
    }
}
```

Tests里面给环境变量赋值

```js
// 请求结果
var data=JSON.parse(responseBody);
// 获取结果的token
var accessToken=data.data.token;
// 设置到环境变量
pm.environment.set("token", token);
```

这样的话，我请求一次登录后，其他接口就可以拿着这个Token继续操作了。同理可以存session，cookie等



## 预请求(Pre-request Script)

字面意思，就是在发送请求之前，发送一个请求。

上面通过Tests把登录后得到的Token放到环境变量中，以便下个接口不用登录就可以使用。这样使用的话，可能每次开机或者服务重启，你都得从新获取一次Token。可以使用`Pre-request Script`来处理，每次请求之前都登录一次，获取Token放到环境变量中即可。

下面代码示例，请求前的登录请求，解析Token放到环境变量中：

```js
// 参数
var loginUrl = pm.environment.get("loginUrl");
var user = pm.environment.get("user");
var password = pm.environment.get("password");
// 请求体
const loginRequest = {
    url: loginUrl,
    method: "POST",
    header: 'Content-Type: application/json',
    body: {
        mode: 'raw',
        raw: JSON.stringify({ user: user, password: password }) 
    }
};
// 拿结果,放环境变量
pm.sendRequest(loginRequest, function (err, response) {
    var accessToken = response.json().data.token;
    pm.environment.set("token", token);
});
```

这里可以发送多个类型的请求：

普通的Get请求

```js
pm.sendRequest("https://postman-echo.com/get", function (err, response) {
    console.log(response.json());
});
```

表单提交的方式

```js
const orgRequest = {
  url: 'https://xxx',
  method: 'POST',
  header: [
      "Content-Type: application/x-www-form-urlencoded; charset=UTF-8"
      ],
  body: {
    mode: 'raw',
    raw: 'a=123&b=4568&c=xxx'
  }
};
pm.sendRequest(orgRequest, function (err, res) {
  console.log(res.json());
});
```

提交JSON数据

```js
// 参数
var loginUrl = pm.environment.get("loginUrl");
var user = pm.environment.get("user");
var password = pm.environment.get("password");
// 请求体
const loginRequest = {
    url: loginUrl,
    method: "POST",
    header: 'Content-Type: application/json',
    body: {
        mode: 'raw',
        raw: JSON.stringify({ user: user, password: password }) 
    }
};
// 拿结果,放环境变量
pm.sendRequest(loginRequest, function (err, response) {
    var accessToken = response.json().data.token;
    pm.environment.set("token", token);
});
```

## 各种随机数

使用很简单，例如：{ {$randomFirstName} }

各种各样的内置的随机如下所示：

### 通用

| 格式                | 描述   | 示例                                                         |
| ------------------- | ------ | ------------------------------------------------------------ |
| **`$guid`**         | ID     | "611c2e81-2ccb-42d8-9ddc-2d0bfa65c1b4"    "3a721b7f-7dc9-4c45-9777-516942b98e0d" |
| **`$timestamp`**    | 时间戳 | 1562757107 , 1562757108, 1562757109                          |
| **`$isoTimestamp`** | 时间   | 2020-06-09T21:10:36.177Z                                     |
| **`$randomUUID`**   | UUID   | "6929bb52-3ab2-448a-9796-d6480ecad36b" "53151b27-034f-45a0-9f0a-d7b6075b67d0" |



### 随机字母数字

| **格式**                  | **描述**         | **示例**                              |
| ------------------------- | ---------------- | ------------------------------------- |
| **`$randomAlphaNumeric`** | 随机字母数字     | `6`, `"y"`, `"z"`                     |
| **`$randomBoolean`**      | 随机true/false   | `true`, `false`, `false`, `true`      |
| **`$randomInt`**          | 0 到 1000 随机数 | `802`, `494`, `200`                   |
| **`$randomColor`**        | 随机颜色字符串   | `"red"`, `"fuchsia"`, `"grey"`        |
| **`$randomHexColor`**     | 随机颜色16进制   | `"#47594a"`, `"#431e48"`, `"#106f21"` |
| **`$randomAbbreviation`** | 随机的缩写       | `SQL`, `PCI`, `JSON`                  |



### 网络相关

| **格式**                | **描述**                        | **示例**                                                     |
| ----------------------- | ------------------------------- | ------------------------------------------------------------ |
| **`$randomIP`**         | IPv4                            | `241.102.234.100`, `216.7.27.38`                             |
| **`$randomIPV6`**             |IPv6|dbe2:7ae6:119b:c161:1560:6dda:3a9b:90a9|
| **`$randomMACAddress`** | MAC                             | `33:d4:68:5f:b4:c7`, `1f:6e:db:3d:ed:fa`                     |
| **`$randomPassword`**   | 15位字母数字密码                | `t9iXe7COoDKv8k3`, `QAzNFQtvR9cg2rq`                         |
| **`$randomLocale`**     | 随机双字母语言代码（ISO 639-1） | `"ny"`, `"sr"`, `"si"`                                       |
| **`$randomUserAgent`**  | 随机用户代理                    | `Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.9.8; rv:15.6) Gecko/20100101 Firefox/15.6.6` |
|                         |                                 | `Opera/10.27 (Windows NT 5.3; U; AB Presto/2.9.177 Version/10.00)` |
|                         |                                 | `Mozilla/5.0 (Windows NT 6.2; rv:13.5) Gecko/20100101 Firefox/13.5.6` |
| **`$randomProtocol`**   | http / https                    | `"http"`, `"https"`                                          |
| **`$randomSemver`**     | 随机版本号                      | `7.0.5`, `2.5.8`, `6.4.9`                                    |







### 名字

| **格式**                | **描述**                     | **示例**                                               |
| ----------------------- | ---------------------------- | ------------------------------------------------------ |
| **`$randomFirstName`**  | A random first name          | `Ethan`, `Chandler`, `Megane`                          |
| **`$randomLastName`**   | A random last name           | `Schaden`, `Schneider`, `Willms`                       |
| **`$randomFullName`**   | A random first and last name | `Connie Runolfsdottir`, `Sylvan Fay`, `Jonathon Kunze` |
| **`$randomNamePrefix`** | A random name prefix         | `Dr.`, `Ms.`, `Mr.`                                    |
| **`$randomNameSuffix`** | A random name suffix         | `I`, `MD`, `DDS`                                       |





### 职业

| **Variable Name**          | **Description**         | **Examples**                            |
| -------------------------- | ----------------------- | --------------------------------------- |
| **`$randomJobArea`**       | A random job area       | `Mobility`, `Intranet`, `Configuration` |
| **`$randomJobDescriptor`** | A random job descriptor | `Forward`, `Corporate`, `Senior`        |
| **`$randomJobTitle`**      | A random job title      | `International Creative Liaison`,       |
|                            |                         | `Product Factors Officer`,              |
|                            |                         | `Future Interactions Executive`         |
| **`$randomJobType`**       | A random job type       | `Supervisor`, `Manager`, `Coordinator`  |





### 电话,地址,位置

| **Variable Name**           | **Description**                                     | **Examples**                                                |
| --------------------------- | --------------------------------------------------- | ----------------------------------------------------------- |
| **`$randomPhoneNumber`**    | A random 10-digit phone number                      | `700-008-5275`, `494-261-3424`, `662-302-7817`              |
| **`$randomPhoneNumberExt`** | A random phone number with extension (12 digits)    | `27-199-983-3864`, `99-841-448-2775`                        |
| **`$randomCity`**           | A random city name                                  | `Spinkahaven`, `Korbinburgh`, `Lefflerport`                 |
| **`$randomStreetName`**     | A random street name                                | `Kuhic Island`, `General Street`, `Kendrick Springs`        |
| **`$randomStreetAddress`**  | A random street address                             | `5742 Harvey Streets`, `47906 Wilmer Orchard`               |
| **`$randomCountry`**        | A random country                                    | `Lao People's Democratic Republic`, `Kazakhstan`, `Austria` |
| **`$randomCountryCode`**    | A random 2-letter country code (ISO 3166-1 alpha-2) | `CV`, `MD`, `TD`                                            |
| **`$randomLatitude`**       | A random latitude coordinate                        | `55.2099`, `27.3644`, `-84.7514`                            |
| **`$randomLongitude`**      | A random longitude coordinate                       | `40.6609`, `171.7139`, `-159.9757`                          |



### 图片

| **Variable Name**           | **Description**                         | **Examples**                                                 |
| --------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| **`$randomAvatarImage`**    | A random avatar image                   | `https://s3.amazonaws.com/uifaces/faces/twitter/johnsmithagency/128.jpg` |
|                             |                                         | `https://s3.amazonaws.com/uifaces/faces/twitter/xadhix/128.jpg` |
|                             |                                         | `https://s3.amazonaws.com/uifaces/faces/twitter/martip07/128.jpg` |
| **`$randomImageUrl`**       | A URL for a random image                | `http://lorempixel.com/640/480`                              |
| **`$randomAbstractImage`**  | A URL for a random abstract image       | `http://lorempixel.com/640/480/abstract`                     |
| **`$randomAnimalsImage`**   | A URL for a random animal image         | `http://lorempixel.com/640/480/animals`                      |
| **`$randomBusinessImage`**  | A URL for a random stock business image | `http://lorempixel.com/640/480/business`                     |
| **`$randomCatsImage`**      | A URL for a random cat image            | `http://lorempixel.com/640/480/cats`                         |
| **`$randomCityImage`**      | A URL for a random city image           | `http://lorempixel.com/640/480/city`                         |
| **`$randomFoodImage`**      | A URL for a random food image           | `http://lorempixel.com/640/480/food`                         |
| **`$randomNightlifeImage`** | A URL for a random nightlife image      | `http://lorempixel.com/640/480/nightlife`                    |
| **`$randomFashionImage`**   | A URL for a random fashion image        | `http://lorempixel.com/640/480/fashion`                      |
| **`$randomPeopleImage`**    | A URL for a random image of a person    | `http://lorempixel.com/640/480/people`                       |
| **`$randomNatureImage`**    | A URL for a random nature image         | `http://lorempixel.com/640/480/nature`                       |
| **`$randomSportsImage`**    | A URL for a random sports image         | `http://lorempixel.com/640/480/sports`                       |
| **`$randomTransportImage`** | A URL for a random transportation image | `http://lorempixel.com/640/480/transport`                    |
| **`$randomImageDataUri`**   | A random image data URI                 | `data:image/svg+xml;charset=UTF-8,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20version%3D%221.1%22%20baseProfile%3D%22full%22%20width%3D%22undefined%22%20height%3D%22undefined%22%3E%20%3Crect%20width%3D%22100%25%22%20height%3D%22100%25%22%20fill%3D%22grey%22%2F%3E%20%20%3Ctext%20x%3D%220%22%20y%3D%2220%22%20font-size%3D%2220%22%20text-anchor%3D%22start%22%20fill%3D%22white%22%3Eundefinedxundefined%3C%2Ftext%3E%20%3C%2Fsvg%3E` |



### Business

| **Variable Name**          | **Description**                                | **Examples**                          |
| -------------------------- | ---------------------------------------------- | ------------------------------------- |
| **`$randomCompanyName`**   | A random company name                          | `Johns - Kassulke`, `Grady LLC`       |
| **`$randomCompanySuffix`** | A random company suffix (e.g. Inc, LLC, Group) | `Inc`, `LLC`, `Group`                 |
| **`$randomBs`**            | A random phrase of business speak              | `killer leverage schemas`,            |
|                            |                                                | `bricks-and-clicks deploy markets`,   |
|                            |                                                | `world-class unleash platforms`       |
| **`$randomBsAdjective`**   | A random business speak adjective              | `viral`, `24/7`, `24/365`             |
| **`$randomBsBuzz`**        | A random business speak buzzword               | `repurpose`, `harness`, `transition`  |
| **`$randomBsNoun`**        | A random business speak noun                   | `e-services`, `markets`, `interfaces` |



### 日期

| **Variable Name**       | **Description**          | **Examples**                                               |
| ----------------------- | ------------------------ | ---------------------------------------------------------- |
| **`$randomDateFuture`** | A random future datetime | `Tue Mar 17 2020 13:11:50 GMT+0530 (India Standard Time)`, |
|                         |                          | `Fri Sep 20 2019 23:51:18 GMT+0530 (India Standard Time)`, |
|                         |                          | `Thu Nov 07 2019 19:20:06 GMT+0530 (India Standard Time)`  |
| **`$randomDatePast`**   | A random past datetime   | `Sat Mar 02 2019 09:09:26 GMT+0530 (India Standard Time)`, |
|                         |                          | `Sat Feb 02 2019 00:12:17 GMT+0530 (India Standard Time)`, |
|                         |                          | `Thu Jun 13 2019 03:08:43 GMT+0530 (India Standard Time)`  |
| **`$randomDateRecent`** | A random recent datetime | `Tue Jul 09 2019 23:12:37 GMT+0530 (India Standard Time)`, |
|                         |                          | `Wed Jul 10 2019 15:27:11 GMT+0530 (India Standard Time)`, |
|                         |                          | `Wed Jul 10 2019 01:28:31 GMT+0530 (India Standard Time)`  |
| **`$randomWeekday`**    | A random weekday         | `Thursday`, `Friday`, `Monday`                             |
| **`$randomMonth`**      | A random month           | `February`, `May`, `January`                               |





### Domains, Emails and Usernames

| **Variable Name**         | **Description**                                 | **Examples**                                                 |
| ------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| **`$randomDomainName`**   | A random domain name                            | `gracie.biz`, `armando.biz`, `trevor.info`                   |
| **`$randomDomainSuffix`** | A random domain suffix                          | `org`, `net`, `com`                                          |
| **`$randomDomainWord`**   | A random unqualified domain name                | `gwen`, `jaden`, `donnell`                                   |
| **`$randomEmail`**        | A random email address                          | `Pablo62@gmail.com`, `Ruthe42@hotmail.com`, `Iva.Kovacek61@hotmail.com` |
| **`$randomExampleEmail`** | A random email address from an “example” domain | `Talon28@example.com`, `Quinten_Kerluke45@example.net`, `Casey81@example.net` |
| **`$randomUserName`**     | A random username                               | `Jarrell.Gutkowski`, `Lottie.Smitham24`, `Alia99`            |
| **`$randomUrl`**          | A random URL                                    | `https://anais.net`, `https://tristin.net`, `http://jakob.name` |





### 文件，目录

| **Variable Name**           | **Description**                                        | **Examples**                                           |
| --------------------------- | ------------------------------------------------------ | ------------------------------------------------------ |
| **`$randomFileName`**       | A random file name (includes uncommon extensions)      | `neural_sri_lanka_rupee_gloves.gdoc`,                  |
|                             |                                                        | `plastic_awesome_garden.tif`,                          |
|                             |                                                        | `incredible_ivory_agent.lzh`                           |
| **`$randomFileType`**       | A random file type (includes uncommon file types)      | `model`, `application`, `video`                        |
| **`$randomFileExt`**        | A random file extension (includes uncommon extensions) | `war`, `book`, `fsc`                                   |
| **`$randomCommonFileName`** | A random file name                                     | `well_modulated.mpg4`,                                 |
|                             |                                                        | `rustic_plastic_tuna.gif`,                             |
|                             |                                                        | `checking_account_end_to_end_robust.wav`               |
| **`$randomCommonFileType`** | A random, common file type                             | `application`, `audio`                                 |
| **`$randomCommonFileExt`**  | A random, common file extension                        | `m2v`, `wav`, `png`                                    |
| **`$randomFilePath`**       | A random file path                                     | `/home/programming_chicken.cpio`,                      |
|                             |                                                        | `/usr/obj/fresh_bandwidth_monitored_beauty.onetoc`,    |
|                             |                                                        | `/dev/css_rustic.pm`                                   |
| **`$randomDirectoryPath`**  | A random directory path                                | `/usr/bin`, `/root`, `/usr/local/bin`                  |
| **`$randomMimeType`**       | A random MIME type                                     | `audio/vnd.vmx.cvsd`,                                  |
|                             |                                                        | `application/vnd.groove-identity-message`,             |
|                             |                                                        | `application/vnd.oasis.opendocument.graphics-template` |

### Lorem Ipsum

| **Variable Name**            | **Description**                            | **Examples**                                                 |
| ---------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **`$randomLoremWord`**       | A random word of lorem ipsum text          | `est`                                                        |
| **`$randomLoremWords`**      | Some random words of lorem ipsum text      | `vel repellat nobis`                                         |
| **`$randomLoremSentence`**   | A random sentence of lorem ipsum text      | `Molestias consequuntur nisi non quod.`                      |
| **`$randomLoremSentences`**  | A random 2-6 sentences of lorem ipsum text | `Et sint voluptas similique iure amet perspiciatis vero sequi atque. Ut porro sit et hic. Neque aspernatur vitae fugiat ut dolore et veritatis. Ab iusto ex delectus animi. Voluptates nisi iusto. Impedit quod quae voluptate qui.` |
| **`$randomLoremParagraph`**  | A random paragraph of lorem ipsum text     | `Ab aliquid odio iste quo voluptas voluptatem dignissimos velit. Recusandae facilis qui commodi ea magnam enim nostrum quia quis. Nihil est suscipit assumenda ut voluptatem sed. Esse ab voluptas odit qui molestiae. Rem est nesciunt est quis ipsam expedita consequuntur.` |
| **`$randomLoremParagraphs`** | 3 random paragraphs of lorem ipsum text    | `Voluptatem rem magnam aliquam ab id aut quaerat. Placeat provident possimus voluptatibus dicta velit non aut quasi. Mollitia et aliquam expedita sunt dolores nam consequuntur. Nam dolorum delectus ipsam repudiandae et ipsam ut voluptatum totam. Nobis labore labore recusandae ipsam quo.` |
|                              |                                            | `Voluptatem occaecati omnis debitis eum libero. Veniam et cum unde. Nisi facere repudiandae error aperiam expedita optio quae consequatur qui. Vel ut sit aliquid omnis. Est placeat ducimus. Libero voluptatem eius occaecati ad sint voluptatibus laborum provident iure.` |
|                              |                                            | `Autem est sequi ut tenetur omnis enim. Fuga nisi dolor expedita. Ea dolore ut et a nostrum quae ut reprehenderit iste. Numquam optio magnam omnis architecto non. Est cumque laboriosam quibusdam eos voluptatibus velit omnis. Voluptatem officiis nulla omnis ratione excepturi.` |
| **`$randomLoremText`**       | A random amount of lorem ipsum text        | `Quisquam asperiores exercitationem ut ipsum. Aut eius nesciunt. Et reiciendis aut alias eaque. Nihil amet laboriosam pariatur eligendi. Sunt ullam ut sint natus ducimus. Voluptas harum aspernatur soluta rem nam.` |
| **`$randomLoremSlug`**       | A random lorem ipsum URL slug              | `eos-aperiam-accusamus`, `beatae-id-molestiae`, `qui-est-repellat` |
| **`$randomLoremLines`**      | 1-5 random lines of lorem ipsum            | `Ducimus in ut mollitia.\nA itaque non.\nHarum temporibus nihil voluptas.\nIste in sed et nesciunt in quaerat sed.` |



更多见官网：[Dynamic variables](https://learning.postman.com/docs/writing-scripts/script-references/variables-list/)

