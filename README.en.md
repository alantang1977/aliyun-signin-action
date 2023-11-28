# 阿里云盘自动签到领取SVIP
## ⭐⭐⭐自动签到脚本实现功能⭐⭐⭐
## 利用金山文档每日任务自动签到
## 签到完成后发送签到成功邮件提醒
## 可实现多账号签到
## 实现多账号签到给不同邮箱发送提醒
## 新增给多账号发送邮件
## 自定义每个账号签到提醒的接收邮箱
## 自定义是否领取签到奖励
## 每月末自动领取所有未领取奖励

# 1.打开金山文档网页端，登录后，新建在线智能表格 https://www.kdocs.cn/latest
![1](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/0e0303be-a441-41df-95d0-44b3e3e3b698)
# 2.表格模板按照我发的如下格式给出(可以自己手打):然后选择效率→高级开发→AirScript脚本编辑器
![2](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/f7db3429-c712-4700-a804-af88d9d8f6d3)
# 3.选择创建脚本→文档共享脚本，可以重命名为(阿里云盘自动签到脚本)
![3](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/fb558445-de19-4308-987b-b68a2ef2b15b)
# 4.点击服务→添加服务→将三个服务全部添加
![4](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/1d2238a6-7b1b-45c9-81ff-f710d87c5f7e)
# 5.将自动签到代码复制到编辑器中，并点击保存，然后关闭脚本编辑器
var myDate = new Date();
var data_time = myDate.toLocaleDateString();

function sleep(d) {
  for (var t = Date.now(); Date.now() - t <= d;);
}

function log(message) {
  console.log(message); // 打印到控制台
  // TODO: 将日志写入文件
}

var tokenColumn = "A";
var signInColumn = "B";
var rewardColumn = "C";
var emailColumn = "F";
var sendEmailColumn = "G";
var customEmailColumn = "I";
var resultColumn = "J";

for (let row = 2; row <= 20; row++) {
  var refresh_token = Application.Range(tokenColumn + row).Text;
  var sflq = Application.Range(signInColumn + row).Text;
  var sflqReward = Application.Range(rewardColumn + row).Text;
  var jsyx = Application.Range(emailColumn + row).Text;
  var sendEmail = Application.Range(sendEmailColumn + row).Text;
  var customEmailResult = Application.Range(customEmailColumn + row).Text;

  var emailConfigured = Application.Range("J1").Text;
  var zdy_host = Application.Range("J2").Text;
  var zdy_post = parseInt(Application.Range("J3").Text);
  var zdy_username = Application.Range("J4").Text;
  var zdy_pasd = Application.Range("J5").Text;

  if (sflq == "是") {
    if (refresh_token != "") {
      // 发起网络请求-获取token
      let data = HTTP.post("https://auth.aliyundrive.com/v2/account/token",
        JSON.stringify({
          "grant_type": "refresh_token",
          "refresh_token": refresh_token
        })
      );
      data = data.json();
      var access_token = data['access_token'];
      var phone = data["user_name"];

      if (access_token == undefined) {
        log("单元格【" + tokenColumn + row + "】内的token值错误，程序执行失败，请重新复制正确的token值");
        continue; // 跳过当前行的后续操作
      }

      try {
        var access_token2 = 'Bearer ' + access_token;
        // 签到
        let data2 = HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_list",
          JSON.stringify({ "_rx-s": "mobile" }),
          { headers: { "Authorization": access_token2 } }
        );
        data2 = data2.json();
        var signin_count = data2['result']['signInCount'];

        var logMessage = "账号：" + phone + " - 签到成功，本月累计签到 " + signin_count + " 天";
        var rewardMessage = "";

        if (sflqReward == "是") {
          if (sflq == "是") {
            try {
              // 领取奖励
              let data3 = HTTP.post(
                "https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",
                JSON.stringify({ "signInDay": signin_count }),
                { headers: { "Authorization": access_token2 } }
              );
              data3 = data3.json();
              var rewardName = data3["result"]["name"];
              var rewardDescription = data3["result"]["description"];
              rewardMessage = " " + rewardName + " - " + rewardDescription;
            } catch (error) {
              if (error.response && error.response.data && error.response.data.error) {
                var errorMessage = error.response.data.error;
                if (errorMessage.includes(" - 今天奖励已领取")) {
                  rewardMessage = " - 今天奖励已领取";
                  log("账号：" + phone + " - " + rewardMessage);
                } else {
                  log("账号：" + phone + " - 奖励领取失败：" + errorMessage);
                }
              } else {
                log("账号：" + phone + " - 奖励领取失败");
              }
            }
          } else {
            rewardMessage = " - 奖励待领取";
          }
        } else {
          rewardMessage = " - 奖励待领取";
        }

        log(logMessage + rewardMessage);

        if (sendEmail == "是") {
          try {
            let mailer;
            if (customEmailResult == "是") {
              var customEmail = Application.Range(resultColumn + row).Text;
              if (emailConfigured === "是") {
                mailer = SMTP.login({
                  host: zdy_host,
                  port: zdy_post,
                  username: zdy_username,
                  password: zdy_pasd,
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<" + zdy_username + ">",
                  to: customEmail,
                  subject: "阿里云盘签到通知 - " + data_time,
                  text: logMessage + rewardMessage
                });
              } else {
                mailer = SMTP.login({
                  host: "smtp.qq.com",
                  port: 465,
                  username: "xxxxxxx@qq.com",
                  password: "xxxxxxxxxxxxxxxxxxxxx",
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<xxxxxxx@qq.com>",
                  to: customEmail,
                  subject: "阿里云盘签到通知 - " + data_time,
                  text: logMessage + rewardMessage
                });
              }
              log("账号：" + phone + " - 已发送邮件至：" + customEmail);
            } else {
              if (emailConfigured === "是") {
                mailer = SMTP.login({
                  host: "smtp.qq.com",
                  port: 465,
                  username: "xxxxxxx@qq.com",
                  password:"xxxxxxxxxxxxxxxxxxxxx",
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<" + zdy_username + ">",
                  to: jsyx,
                  subject: "阿里云盘签到通知 - " + data_time,
                  text: logMessage + rewardMessage
                });
              } else {
                mailer = SMTP.login({
                  host: "smtp.qq.com",
                  port: 465,
                  username: ,
                  password: "xxxxxxxxxxxxxxxxxxxxx",
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<xxxxxxx@qq.com>",
                  to: jsyx,
                  subject: "阿里云盘签到通知 - " + data_time,
                  text: logMessage + rewardMessage
                });
              }
              log("账号：" + phone + " - 已发送邮件至：" + jsyx);
            }
          } catch (error) {
            log("账号：" + phone + " - 发送邮件失败：" + error);
          }
        }
      } catch {
        log("单元格【" + tokenColumn + row + "】内的token签到失败");
        continue; // 跳过当前行的后续操作
      }
    } else {
      log("账号：" + phone + " 不签到");
    }
  }
}

var currentDate = new Date();
var currentDay = currentDate.getDate();
var lastDayOfMonth = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0).getDate();

if (currentDay === lastDayOfMonth) {
  for (let row = 2; row <= 20; row++) {
    var sflq = Application.Range(signInColumn + row).Text;
    var sflqReward = Application.Range(rewardColumn + row).Text;

    if (sflq === "是" && sflqReward === "是") {
      var refresh_token = Application.Range(tokenColumn + row).Text;
      var jsyx = Application.Range(emailColumn + row).Text;
      var phone = "账号：" + phone;

      if (refresh_token !== "") {
        // 发起网络请求-获取token
        let data = HTTP.post("https://auth.aliyundrive.com/v2/account/token",
          JSON.stringify({
            "grant_type": "refresh_token",
            "refresh_token": refresh_token
          })
        );
        data = data.json();
        var access_token = data['access_token'];

        if (access_token === undefined) {
          log("单元格【" + tokenColumn + row + "】内的token值错误，程序执行失败，请重新复制正确的token值");
          continue; // 跳过当前行的后续操作
        }

        try {
          var access_token2 = 'Bearer ' + access_token;
          // 领取奖励
          let data4 = HTTP.post(
            "https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",
            JSON.stringify({ "signInDay": lastDayOfMonth }),
            { headers: { "Authorization": access_token2 } }
          );
          data4 = data4.json();
          var claimStatus = data4["result"]["status"];
          var day = lastDayOfMonth;

          if (claimStatus === "CLAIMED") {
            log("账号：" + phone + " - 第 " + day + " 天奖励领取成功");
          } else {
            log("账号：" + phone + " - 第 " + day + " 天奖励领取失败");
          }
        } catch {
          log("单元格【" + tokenColumn + row + "】内的token签到失败");
          continue; // 跳过当前行的后续操作
        }
      } else {
        log("账号：" + phone + " 不签到");
      }
    }
  }

  log("自动领取未领取奖励完成。");
}
![5](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/48376ebb-651a-41f8-9673-e42ba1c1485f)
# 6.取得阿里云盘token方法如下：先通过浏览器打开阿里云盘官网并登录网页版：https://www.aliyundrive.com/drive/ 登录成功后，打开开发者工具 ① 点击 Console进入控制台 ② 在控制台输入  JSON.parse(localStorage.token).refresh_token  复制返回的32位字符串，不要复制双引号
![6](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/b8a89b54-356b-4895-9bf7-541f3d140d36)
# 7.将刚才复制的refresh_token粘贴到表格A1列中，并自定义其他信息
![7](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/212ccb4f-211a-4410-941b-de6de0e0e195)
# 8.打开脚本编辑器点击运行测试一下，如果出现运行成功日志，就可说明配置完成
![8](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/133f8dd8-b75e-47c2-a8e2-057f73e027de)
# 9.在文档页，打开效率→高级开发→定时任务→创建任务，选择每天，确定一个合适的自动执行时间，选择你创建的脚本，点击确认
![9](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/8639c733-0dbb-46e9-96d5-6feb3b34f043)


# PS: Token值会在一个月左右刷新一次，届时直接替换就可以了

