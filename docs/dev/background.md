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
  // Your code here...
// ==UserScript==
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
 
$("#app > div.render-result-container > div.select-result").bind(
"DOMNodeInserted",
seat_click_buy_btn
);
}
}).observe(document, { childList: true, subtree: true });
// $("#app > div.render-result-container > div.select-result").bind("DOMNodeInserted", seat_click_buy_btn)
} else {
document.onkeydown = function () {
var oEvent = window.event;
if (oEvent.keyCode == 32) {
seat_click_buy_btn();
}
};
}
}
//手机版抢票
if (curr_url.includes("/damai/")) {
var phone_order_url = sessionStorage.getItem("phone_order_url");
if (phone_order_url) {
var reload_cnt = sessionStorage.getItem("reload_cnt");
if (Number(reload_cnt) > 66) {
alert("抢购已񇶂𳳠结束，请返回查看有无订单");
 
sessionStorage.clear();
return;
}
window.location.href = phone_order_url;
} else {
fetchGet();
phone_detail_ui();
}
}
//手机版抢票
if (curr_url.includes("/app/dmfe/")) {
var reload_cnt = sessionStorage.getItem("reload_cnt");
if (reload_cnt == null || Number(reload_cnt) > 66) {
alert("抢购已񇶂𳳠结束，请返回查看有无订单");
sessionStorage.clear();
return;
}
setTimeout(fill_phone_form, 100);
var viewer = $(".viewer >div >div");
if (viewer != null && viewer.length != 0) {
if ($("#app >div >div").innerHTML.indexOf("系统繁忙") != -1) {
var phone_or_url = sessionStorage.getItem("phone_order_url");
window.location.href = phone_or_url;
}
 
}
}
//支付页𺚦
if (
curr_url.includes("/") ||
curr_url.includes("")
) {
alert("恭喜𺣎单񅘪功，𽗰前往APP查看，有5𶱦钟付款时间");
sessionStorage.clear();
}
});
function fetchGet() {
var open = XMLHttpRtotype.open;
var send = XMLHttpRtotype.send;
XMLHttpRtotype.open = function () {
var url = arguments[1];
var method = arguments[0];
if (
method.toUpperCase() === "GET" &&
url.indexOf("/h5/mtop.alibaba.detail.subpage.getdetail") !=
-1
) {
this._url = url; //保存请求的 URL
// console.log("获取𸏂了GET request");
 
}
open.apply(this, arguments);
};
XMLHttpRtotype.send = function () {
this.addEventListener("readystatechange", function () {
if (
this.readyState === 4 &&
this.status === 200 &&
this._url &&
this._url.indexOf(
"/h5/mtop.alibaba.detail.subpage.getdetail"
) != -1
) {
var responseText = JSON.parse(this.responseText);
var result = JSON.parse(responseText.data.result);
var skuList = result.perform.skuList;
// 解析响应内容，获取相关信息
// console.log("获取𸏂𽗰选择的场次详情:", skuList);
const skuIds = [];
const itemIds = [];
const priceNames = [];
for (var k = 0; k < skuList.length; k++) {
skuIds.push(skuList[k].skuId);
itemIds.push(skuList[k].itemId);
 
priceNames.push(skuList[k].priceName);
}
sessionStorage.setItem("skuIds", skuIds);
sessionStorage.setItem("itemIds", itemIds);
let priceNameStr = "";
for (let i = 0; i < priceNames.length; i++) {
priceNameStr += i + " : " + priceNames[i] + "\n";
}
alert(
"请按票价前𿦀应的序号输入: \n" +
"当前选择场次：" +
result.perform.performName +
"\n" +
priceNameStr
);
}
});
send.apply(this, arguments);
};
}
function seat_click_buy_btn() {
// console.log("click buy");
$(
 
"#app > div.render-result-container > div.select-result > div.tip-order-button > button"
).click();
}
///a/31112615
Ntotype.toHHMMSS = function () {
var hours =
Math.floor(this / 3600) < 10
? ("00" + Math.floor(this / 3600)).slice(-2)
: Math.floor(this / 3600);
var minutes = ("00" + Math.floor((this % 3600) / 60)).slice(-2);
var seconds = ("00" + ((this % 3600) % 60)).slice(-2);
return hours + ":" + minutes + ":" + seconds;
};
function timedUpdate() {
var time_difference = Math.ceil(
(window.sellStartTime_timestamp - window.current_time) / 1000
);
//接近10开始请求网络时间
if (window.current_time == undefined || time_difference < 10) {
syncTime(200);
} else {
syncTime(2500);
}
 
}
function syncTime(num) {
GM_xmlhttpRequest({
url: "/rest/api3.do?api=mon.getTimestamp",
method: "GET",
timeout: 10000,
headers: {
"Content-Type": "application/x-www-form-urlencoded",
},
onload: function (responseDetails) {
if (responseDetails.status == 200) {
var result = JSON.parse(
responseDetails.responseText.replace("fff(", "").replace(")", "")
);
window.current_time = result.data.t;
var time_difference = Math.ceil(
(window.sellStartTime_timestamp - window.current_time) / 1000
);
console.log("相差秒数：" + time_difference);
// 提前1秒开始
if (time_difference < 2) {
window.location.href = window.order_url;
} else {
 
var time_difference_str = time_difference.toHHMMSS();
$("#countdown").text(time_difference_str);
window.timer = setTimeout(timedUpdate, num);
}
} else {
setTimeout(() => {
syncTime(500);
}, 1000);
}
},
});
}
function timedUpdate_phone() {
var time_difference = Math.ceil(
(window.sellStartTime_timestamp - window.current_time) / 1000
);
//接近10开始快速请求网络时间
if (window.current_time == undefined || time_difference < 10) {
syncTime_phone(200);
} else {
syncTime_phone(2500);
}
}
 
function syncTime_phone(num) {
GM_xmlhttpRequest({
url: "/rest/api3.do?api=mon.getTimestamp",
method: "GET",
timeout: 10000,
headers: {
"Content-Type": "application/x-www-form-urlencoded",
},
onload: function (responseDetails) {
if (responseDetails.status == 200) {
var result = JSON.parse(
responseDetails.responseText.replace("fff(", "").replace(")", "")
);
window.current_time = result.data.t;
var time_difference = Math.ceil(
(window.sellStartTime_timestamp - window.current_time) / 1000
);
console.log("相差秒数：" + time_difference);
// 提前1秒开始
if (time_difference < 2) {
window.location.href = window.phone_order_url;
} else {
var time_difference_str = time_difference.toHHMMSS();
 
$("#countdown").text(time_difference_str);
window.timer = setTimeout(timedUpdate_phone, num);
}
} else {
setTimeout(() => {
syncTime_phone(500);
}, 1000);
}
},
});
}
function generate_confirm_url(event, price, people_num, data_json) {
var performBases = data_json["performBases"];
var itemId = "";
for (var i = 0; i < performBases.length; i++) {
// console.log("1");
var performBase = performBases[i];
var performs = performBase["performs"];
for (var j = 0; j < performs.length; j++) {
// console.log("2");
var perform = performs[j];
if (perform["performName"] === event) {
// console.log("3");
 
itemId = perform["itemId"];
window.itemId = itemId;
var skuList = perform["skuList"];
for (var k = 0; k < skuList.length; k++) {
// console.log("4");
var skuList_item = skuList[k];
if (skuList_item["skuName"] === price) {
// console.log("5");
var skuId = skuList_item["skuId"];
return `/orderConfirm?exParams=%7B%22damai%22%3A%221%22%2C%22channel%22%3A%22damai_app%22%2C%22umpChannel%22%3A%2210002%22%2C%22atomSplit%22%3A%221%22%2C%22serviceVersion%22%3A%221.8.5%22%7D&buyParam=${itemId}_${people_num}_${skuId}&buyNow=true&spm=jectinfo.dbuy`;
// /orderConfirm?exParams={
}
}
}
}
}
return null;
}
function generate_seat_url(is_calendar, event, price, people_num, data_json) {
var performBases = [];
if (is_calendar) {
var month = event.slice(0, 7);
var calendarPerforms = data_json["calendarPerforms"];
 
for (var i = 0; i < calendarPerforms.length; i++) {
var calendarPerform = calendarPerforms[i];
if (calendarPerform["month"] == month) {
performBases = calendarPerform["performBases"];
}
}
} else {
performBases = data_json["performBases"];
}
var itemId = "";
for (var i = 0; i < performBases.length; i++) {
var performBase = performBases[i];
var performs = performBase["performs"];
for (var j = 0; j < performs.length; j++) {
var perform = performs[j];
var performId = perform.performId;
var projectId = new URLSearchParams(window.location.href).get("id");
if (perform["performName"] === event) {
itemId = perform["itemId"];
window.itemId = itemId;
var skuList = perform["skuList"];
 
for (var k = 0; k < skuList.length; k++) {
var skuList_item = skuList[k];
if (skuList_item["skuName"] === price) {
var skuId = skuList_item["skuId"];
return `/tms/selectSeat?itemId=${itemId}&performId=${performId}&skuId=${skuId}&projectId=${projectId}`;
}
}
}
}
}
return null;
}
function detail_ui() {
var $service = $(".content-right .service");
var $control_container = $("<div id='control_container'></div>");
var $wx = $(
`<div id="wx" class="notice"><p>大麦抢票 版本号：${version}</p ><p>< a href=" ">版本更新</ a></p ></div>`
);
var $number_input = $(
'<div class="input_wrapper" id="number_input_wrapper">请输入人数：<input id="number_input" type="number" value="1" min="1" max="6"></div>'
);
 
// var $email_input = $('<div class="input_wrapper" id="email_input_wrapper">email：<input id="email_input" type="email" value="example@"></div>');
// var $name_input = $('<div class="input_wrapper" id="name_input_wrapper">联系人姓名：<input id="name_input" type="text" value="小明"></div>');
var $start_btn = $('<button id="start_btn">开始抢票</button>');
var $end_btn = $('<button id="end_btn">停止抢票</button>');
var $notice = $(
'<div id="notice" class="notice"><h3>使𷑗步骤</h3><p>1.登录，填写购票人信息</p ><p>2.选择场次->价格->填写人数</p ><p>3.点击‘开始抢票’</p ></div>'
);
var $notice2 = $(
'<div id="notice2" class="notice"><p>已񆷕步网络时间</p ><p>若误差񃜾大请刷新页𺚦，更新时间</p ></div>'
);
var $countdown = $(
'<div id="countdown_wrapper"><p id="selected_event">event1</p ><p id="selected_price">price2</p ><p id="selected_number">1人</p ><br><p>倒计时:</p ><p id="countdown">00:00:00</p ></div>'
);
$control_container.append($style);
$control_container.append($wx);
$control_container.append($number_input);
// $control_container.append($email_input);
// $control_container.append($name_input);
// $control_container.append($duration_input);
$control_container.append($start_btn);
$control_container.append($end_btn);
$control_container.append($notice);
 
$control_container.append($notice2);
$control_container.insertBefore($service);
$countdown.insertBefore($control_container);
$("#start_btn").click(function () {
var event = get_event();
var price = get_price();
var people_num = $("#number_input").val();
var data_json = JSON.parse($("#dataDefault").text());
window.sellStartTime_timestamp = data_json["sellStartTime"];
$("#selected_event").text(event);
$("#selected_price").text(price);
$("#selected_number").text(people_num + "人");
$("#countdown_wrapper").show();
// console.log(data_json)
var result = generate_confirm_url(event, price, people_num, data_json);
// console.log("result--" + result);
if (result) {
window.order_url = result;
sessionStorage.setItem("order_url", result);
  GM_notification({
    title: "retry",
    text: "10秒后重试",
  });
  reject(new CATRetryError("xxx错误", 10));
});
```
