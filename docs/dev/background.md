---
id: backgroud
---

# 后台脚本

后台脚本适用于持续运行类型的脚本.后台脚本是脚本猫特有的脚本,后台脚本运行在沙盒中,无法操作
dom 对象.可使用与油猴一致的 GM API 进行开发,对于兼容性会在文档中标出.

## 后台脚本

后台脚本由`@background`属性声明,后台脚本将允许在开启脚本或者浏览器启动后,让脚本在后台持续运行.

## 定时脚本

> 定时脚本也属于后台脚本的一种,定时脚本适用于每隔一段时间执行一次的脚本.

定时脚本由`@crontab`属性声明,可以精确到秒级调用,提供了一个`once`参数,表示某个时间段内会执行一次(考虑浏览器未打开的情况).建议脚本的运行时间和重试时间不要大于定时时间的间隔.

可使用在线工具测试 cron
表达式,扩展中的`once`替换成`*`:[cron 在线测试](https://tool.lu/crontab/)

在控制台页面,鼠标放置`运行状态`栏时会显示下一次的运行时间.

### Crontab 例子

> 请注意一个脚本中只有第一个 crontab
> 表达式生效,`once`语义为只在当前的时间位上运行一次

```javascript
//@crontab * * * * * * 每秒运行一次
//@crontab * * * * * 每分钟运行一次
//@crontab 0 */6 * * * 每6小时的0分时执行一次
//@crontab 15 */6 * * * 每6小时的15分时执行一次
//@crontab * once * * * 每小时运行一次
//@crontab * * once * * 每天运行一次
//@crontab * 10 once * * 每天10点-10:59中运行一次,假设当10:04时运行了一次,10:05-10:59的后续的时间将不会再运行
//@crontab * 1,3,5 once * * 每天1点3点5点中运行一次,假设当1点时运行了一次,3,5点将不会再运行
//@crontab * */4 once * * 每天每隔4小时检测运行一次,假设当4点时运行了一次,8,12,16,20,24点等后续的时间将不会再运行
//@crontab * 10-23 once * * 每天10点-23:59中运行一次,假设当10:04时运行了一次,10:05-23:59的后续时间将不会再运行
//@crontab * once 13 * * 每个月的13号的每小时运行一次
```

## 日志

在脚本列表页面，鼠标放置`运行状态栏`会提示脚本的运行状态,点击可弹出通过`GM_log`打印的日志内容

![](./background.assets/image-20210621214143661.png)

![](./background.assets/image-20210621214124685.png)

## 脚本调试

后台脚本可直接在脚本编辑页面进行调试,但是存在下面问题: value
无法正常同步,registerMenu 菜单无法正常触发.

![](./background.assets/image-20210903141601057.png)

如果是运行的环境请前往扩展开启**开发人员模式**然后在点击扩展的 background.html
进行调试，运行时产生的错误也可以在运行日志中查看

![image-20210903144155450](./background.assets/image-20210903144155450.png)

## Promise

十分推荐这种写法,也便于脚本管理器的脚本监控,对于有异步执行的,必须使用`Promise`.

脚本返回`Promise`对象,管理器也可以将返回的内容当作日志记录下来.

```ts
// ==UserScript==
// @name         后台脚本
// @namespace    wyz
// @version      1.0.0
// @author       wyz
// @background
// ==/UserScript==
return new Promise((resolve, reject) => {
  if (Math.round((Math.random() * 10) % 2)) {
    resolve("ok"); // 执行成功
  } else {
    reject("error"); // 执行失败,并返回错误原因
  }
});
```

```js
// ==UserScript==
// @name         每天运行一次的定时脚本
// @namespace    wyz
// @version      1.0.0
// @author       wyz
// @crontab      * * once * *
// ==/UserScript==
return new Promise((resolve, reject) => {
  if (Math.round((Math.random() * 10) % 2)) {
    resolve("ok"); // 执行成功
  } else {
    reject("error"); // 执行失败,并返回错误原因
  }
});
```

请注意将`resolve/reject`的操作放入执行完毕后的步骤中,`resolve/reject`后管理器会认为脚本执行完毕,后续的所有
GM 操作将不会生效.如果是希望持续运行的后台脚本,可以不调用`resolve`.

```js
// ==UserScript==
// @name         请求API
// @namespace    wyz
// @version      1.0.0
// @author       wyz
// @crontab      * * once * *
// ==/UserScript==
return new Promise((resolve, reject) => {
  GM_xmlhttpRequest({
    url: "https://bbs.tampermonkey.net.cn/",
    onload() {
      resolve("ok"); // 执行成功
    },
    onerror() {
      reject("error"); // 执行失败,并返回错误原因
    },
  });
});
```

## 错误重试

脚本猫后台脚本可以进行重试, 当你的脚本出现错误时, 可以 reject 返回`CATRetryError`,
以便脚本猫重试 重试时间请注意不要与脚本执行时间冲突, 否则可能会导致重复执行,
最小重试时间为 5s

```js
// ==UserScript==
// @name         重试示例
// @namespace    https://bbs.tampermonkey.net.cn/
// @version      0.1.0
// @description  try to take over the world!
// @author       You
// @crontab      * * once * *
// @grant        GM_notification
// ==/UserScript==

return new Promise((resolve, reject) => {
  // // ==UserScript==
// @name 大麦抢票
// @namespace ZnG大麦
// @version 8.5.0
// @description 全񇶂𳳠抢票，񇶂𳳠提交订单
// @author ZnG大麦
 
// @match /*
// @match /*
// @match /*
// @match /*
// @match /*
// @match </h5/mtop.alibaba.detail.subpage.getdetail/*>
// @grant GM_xmlhttpRequest
// @connect
// @require /jquery/3.5.1/jquery.min.js
// @license wxywxy
// ==/UserScript==
// /tms/selectSeat?itemId=624499596673&hasPromotion=true&performId=210252620&skuId=4427321477193&projectId=624499596673&spm=jectinfo.dbuy
// /tms/selectSeat?itemId=624499596673&performId=210252601&skuId=4427321477292&projectId=624499596673
var version = "8.5.0";
var $style = $(
"<style>" +
"#control_container{margin: 20px 0;background:#e9e9e9;padding:20px;overflow: hidden;}" +
"p{margin:10px 0;}" +
"#control_container button{width:80%;height: 60px;line-height:60px;margin:10px 10%;font-size:30px;border-radius:30px}" +
"#start_btn{color:#fff;background:#ff457a;}" +
"#bp_btn{color:#fff;background: #ff457a;}" +
"#end_btn{color:#666;background: #cfcfcf;}" +
 
".control_container_box{display: flex;align-items: center;flex-wrap: wrap;padding-right: 20px;border: 1px solid #ccc;}" +
".input_wrapper{display: flex;justify-content:center;font-size: 16px; margin-bottom:10px;}" +
".input_wrapper_box{flex: 4;}" +
".input_wrapper_phone{display: flex;justify-content: flex-end;font-size: 25px;padding:20px 0; text-align:center;}" +
".input_wrapper_phone input{width: 50%;}" +
".notice{margin:10px 10px;padding:10px 10px;color:darkslategrey;}" +
"#wx{text-align: center; flex:1;color: #333;}" +
"#countdown_wrapper {display:none; font-size: 30px; text-align:center; background:#ffeaf1;}" +
"#countdown_wrapper p{width:100%;}" +
"#countdown {font-size: 50px; color:#ff1268;}" +
".warning {color:red; font-weight:400;}" +
"h3 {font-weight:800;}" +
"</style>"
);
$(document).ready(function () {
var curr_url = window.location.href;
if (curr_url.includes("/")) {
var order_url = sessionStorage.getItem("order_url");
if (order_url) {
window.location.href = order_url;
} else {
if (
$("div.buybtn").text() == "选座购买" ||
$(".service-note .service-note-name").text().includes("𽗰选座")
 
) {
alert(
"无𸽏全񇶂𳳠选座，请看“注意”𹟔𶱦。不要忘了先登录，填好联系人信息，删除𶖊余联系人。"
);
detail_seat_ui();
} else {
detail_ui();
}
}
}
if (curr_url.includes("/")) {
if (curr_url.includes("/multi/flow")) {
//𺣎单挤爆了
var order_url = curr_url.substring(curr_url.indexOf("=") + 1);
sessionStorage.setItem("order_url", order_url);
window.location.href = order_url;
} else {
if ($(".error-msg").length > 0) {
if ($(".error-msg").text().includes("已񃜾期")) {
document
.getElementsByClassName("next-row error-reload")[0]
.children[0].click();
} else {
var order_url = sessionStorage.getItem("order_url");
 
if (order_url) {
window.location.href = order_url;
} else {
window.location.reload();
}
}
} else {
setTimeout(fill_form, 200);
}
}
}
if (curr_url.includes("/")) {
// console.log("seat");
var people_num = new URLSearchParams(window.location.href).get(
"people_num"
);
if (people_num == 1) {
new MutationObserver(function (mutations) {
if (
document.querySelector(
"#app > div.render-result-container > div.select-result"
)
) {
// console.log("found ele");
  GM_notification({
    title: "retry",
    text: "10秒后重试",
  });
  reject(new CATRetryError("xxx错误", 10));
});
```
