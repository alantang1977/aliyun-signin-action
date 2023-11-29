## ⭐⭐⭐阿里云盘自动签到领取SVIP 签到完成后发送签到成功提醒⭐⭐⭐
## 1.打开金山文档网页端，登录后，新建在线智能表格 https://www.kdocs.cn/latest
![1](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/0e0303be-a441-41df-95d0-44b3e3e3b698)
## 2.表格模板按照我发的如下格式给出(可以自己手打):然后选择效率→高级开发→AirScript脚本编辑器
![2](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/f7db3429-c712-4700-a804-af88d9d8f6d3)
## 3.选择创建脚本→文档共享脚本，可以重命名为(阿里云盘自动签到脚本)
![3](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/fb558445-de19-4308-987b-b68a2ef2b15b)
## 4.点击服务→添加服务→将三个服务全部添加
![4](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/1d2238a6-7b1b-45c9-81ff-f710d87c5f7e)
## 5.将自动签到代码复制到编辑器中，并点击保存，然后关闭脚本编辑器

var myDate = new Date(); // 创建一个表示当前时间的 Date 对象
var data_time = myDate.toLocaleDateString(); // 获取当前日期的字符串表示

function sleep(d) {
  for (var t = Date.now(); Date.now() - t <= d;); // 使程序暂停执行一段时间
}

function log(message) {
  console.log(message); // 打印消息到控制台
  // TODO: 将日志写入文件
}

var tokenColumn = "A"; // 设置列号变量为 "A"
var signInColumn = "B"; // 设置列号变量为 "B"
var rewardColumn = "C"; // 设置列号变量为 "C"
var emailColumn = "F"; // 设置列号变量为 "F"
var sendEmailColumn = "G"; // 设置列号变量为 "G"
var resultColumn = "J"; // 设置列号变量为 "J"

for (let row = 2; row <= 20; row++) { // 循环遍历从第 2 行到第 20 行的数据
  var refresh_token = Application.Range(tokenColumn + row).Text; // 获取指定单元格的值
  var sflq = Application.Range(signInColumn + row).Text; // 获取指定单元格的值
  var sflqReward = Application.Range(rewardColumn + row).Text; // 获取指定单元格的值
  var jsyx = Application.Range(emailColumn + row).Text; // 获取指定单元格的值
  var sendEmail = Application.Range(sendEmailColumn + row).Text; // 获取指定单元格的值
  var customEmailResult = Application.Range(resultColumn + row).Text; // 获取指定单元格的值

  var emailConfigured = Application.Range("J1").Text; // 获取指定单元格的值
  var zdy_host = Application.Range("J2").Text; // 获取指定单元格的值
  var zdy_post = parseInt(Application.Range("J3").Text); // 获取指定单元格的值并转换为整数
  var zdy_username = Application.Range("J4").Text; // 获取指定单元格的值
  var zdy_pasd = Application.Range("J5").Text; // 获取指定单元格的值

  if (sflq == "是") { // 如果“是否签到”为“是”
    if (refresh_token != "") { // 如果刷新令牌不为空
      // 发起网络请求-获取token
      let data = HTTP.post("https://auth.aliyundrive.com/v2/account/token",
        JSON.stringify({
          "grant_type": "refresh_token",
          "refresh_token": refresh_token
        })
      );
      data = data.json(); // 将响应数据解析为 JSON 格式
      var access_token = data['access_token']; // 获取访问令牌
      var phone = data["user_name"]; // 获取用户名

      if (access_token == undefined) { // 如果访问令牌未定义
        log("单元格【" + tokenColumn + row + "】内的token值错误，程序执行失败，请重新复制正确的token值");
        ///
        if (sendEmail == "是") { // 如果“是否发送邮件”为“是”
          try {
            let mailer;
            if (customEmailResult == "是") { // 如果“是否自定义邮箱”为“是”
              var customEmail = Application.Range(resultColumn + row).Text; // 获取指定单元格的值
              if (emailConfigured === "是") { // 如果配置了自定义邮箱
                mailer = SMTP.login({
                  host: zdy_host,
                  port: zdy_post,
                  username: zdy_username,
                  password: zdy_pasd,
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到",
                  to: customEmail,
                  subject: "阿里云盘签到失败通知 - " + data_time,
                  text: "token值错误，签到失败，请及时更新token值\n具体操作步骤：\n1.如图获取token：https://attach.52pojie.cn/forum/202307/23/004425l9xoj0c69r9x6ot1.png\n2.将token告知我"
                });
              } else { // 如果未配置自定义邮箱，默认使用示例邮箱
                mailer = SMTP.login({
                  host: "smtp.163.com",
                  port: 465,
                  username: "fs8484848@163.com",
                  password: "QADSEMPKDHDAVWVD",
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<fs8484848@163.com>",
                  to: customEmail,
                  subject: "阿里云盘签到失败通知 - " + data_time,
                  text: "token值错误，签到失败，请及时更新token值\n具体操作步骤：\n1.如图获取token：https://attach.52pojie.cn/forum/202307/23/004425l9xoj0c69r9x6ot1.png\n2.将token告知我"
                });
              }
              log("账号签到失败 - 已发送邮件至：" + customEmail);
            } else { // 如果“是否自定义邮箱”为“否”
              if (emailConfigured === "是") { // 如果配置了自定义邮箱
                mailer = SMTP.login({
                  host: zdy_host,
                  port: zdy_post,
                  username: zdy_username,
                  password: zdy_pasd,
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<" + zdy_username + ">",
                  to: jsyx,
                  subject: "阿里云盘签到失败通知 - " + data_time,
                  text: "token值错误，签到失败，请及时更新token值\n具体操作步骤：\n1.如图获取token：https://attach.52pojie.cn/forum/202307/23/004425l9xoj0c69r9x6ot1.png\n2.将token告知我"
                });
              } else { // 如果未配置自定义邮箱，默认使用示例邮箱
                mailer = SMTP.login({
                  host: "smtp.163.com",
                  port: 465,
                  username: "fs8484848@163.com",
                  password: "QADSEMPKDHDAVWVD",
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<fs8484848@163.com>",
                  to: jsyx,
                  subject: "阿里云盘签到失败通知 - " + data_time,
                  text: "token值错误，签到失败，请及时更新token值\n具体操作步骤：\n1.如图获取token：https://attach.52pojie.cn/forum/202307/23/004425l9xoj0c69r9x6ot1.png\n2.将token告知我"
                });
              }
              log("账号签到失败 - 已发送邮件至：" + jsyx);
            }
          } catch (error) {
            log("账号签到失败 - 发送邮件失败：" + error.stack);
          }
        }
        ///
        continue; // 跳过当前行的后续操作
      }

      try {
        var access_token2 = 'Bearer ' + access_token; // 构建包含访问令牌的请求头
        // 签到
        let data2 = HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_list",
          JSON.stringify({ "_rx-s": "mobile" }),
          { headers: { "Authorization": access_token2 } }
        );
        data2 = data2.json(); // 将响应数据解析为 JSON 格式
        var signin_count = data2['result']['signInCount']; // 获取签到次数

        var logMessage = "账号：" + phone + " - 签到成功，本月累计签到 " + signin_count + " 天";
        var rewardMessage = "";

        if (sflqReward == "是") { // 如果“是否领取奖励”为“是”
          if (sflq == "是") { // 如果“是否签到”为“是”
            try {
              // 领取奖励
              let data3 = HTTP.post(
                "https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",
                JSON.stringify({ "signInDay": signin_count }),
                { headers: { "Authorization": access_token2 } }
              );
              data3 = data3.json(); // 将响应数据解析为 JSON 格式
              var rewardName = data3["result"]["name"]; // 获取奖励名称
              var rewardDescription = data3["result"]["notice"]; // 获取奖励描述
              rewardMessage = " " + rewardName + " - " + rewardDescription;
            } catch (error) {
              if (error.response && error.response.data && error.response.data.error) {
                var errorMessage = error.response.data.error; // 获取错误信息
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

        if (sendEmail == "是") { // 如果“是否发送邮件”为“是”
          try {
            let mailer;
            if (customEmailResult == "是") { // 如果“是否自定义邮箱”为“是”
              var customEmail = Application.Range(resultColumn + row).Text; // 获取指定单元格的值
              if (emailConfigured === "是") { // 如果配置了自定义邮箱
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
                  subject: "阿里云盘签到成功通知 - " + data_time,
                  text: logMessage + rewardMessage
                });
              } else { // 如果未配置自定义邮箱，默认使用示例邮箱
                mailer = SMTP.login({
                  host: "smtp.163.com",
                  port: 465,
                  username: "fs8484848@163.com",
                  password: "QADSEMPKDHDAVWVD",
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<fs8484848@163.com>",
                  to: customEmail,
                  subject: "阿里云盘签到成功通知 - " + data_time,
                  text: logMessage + rewardMessage
                });
              }
              log("账号：" + phone + " - 已发送邮件至：" + customEmail);
            } else { // 如果“是否自定义邮箱”为“否”
              if (emailConfigured === "是") { // 如果配置了自定义邮箱
                mailer = SMTP.login({
                  host: zdy_host,
                  port: zdy_post,
                  username: zdy_username,
                  password: zdy_pasd,
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<" + zdy_username + ">",
                  to: jsyx,
                  subject: "阿里云盘签到成功通知 - " + data_time,
                  text: logMessage + rewardMessage
                });
              } else { // 如果未配置自定义邮箱，默认使用示例邮箱
                mailer = SMTP.login({
                  host: "smtp.163.com",
                  port: 465,
                  username: "fs8484848@163.com",
                  password: "QADSEMPKDHDAVWVD",
                  secure: true
                });
                mailer.send({
                  from: "阿里云盘签到<fs8484848@163.com>",
                  to: jsyx,
                  subject: "阿里云盘签到成功通知 - " + data_time,
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

var currentDate = new Date(); // 创建一个表示当前时间的 Date 对象
var currentDay = currentDate.getDate(); // 获取当前日期的天数
var lastDayOfMonth = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0).getDate(); // 获取当月的最后一天的日期

if (currentDay === lastDayOfMonth) { // 如果当前日期是当月的最后一天
  log("强制领取本月奖励")
  for (let row = 2; row <= 20; row++) { // 循环遍历从第 2 行到第 20 行的数据
    var sflq = Application.Range(signInColumn + row).Text; // 获取指定单元格的值
    var sflqReward = Application.Range(rewardColumn + row).Text; // 获取指定单元格的值

    if (sflq === "是") { // 如果“是否签到”为“是”，则强制领取
      var refresh_token = Application.Range(tokenColumn + row).Text; // 获取指定单元格的值
      var jsyx = Application.Range(emailColumn + row).Text; // 获取指定单元格的值

      if (refresh_token !== "") { // 如果刷新令牌不为空
        // 发起网络请求-获取token
        let data = HTTP.post("https://auth.aliyundrive.com/v2/account/token",
          JSON.stringify({
            "grant_type": "refresh_token",
            "refresh_token": refresh_token
          })
        );
        data = data.json(); // 将响应数据解析为 JSON 格式
        var access_token = data['access_token']; // 获取访问令牌
        var phone = "账号：" + data["user_name"]; // 获取用户名
        if (access_token === undefined) { // 如果访问令牌未定义
          log("单元格【" + tokenColumn + row + "】内的token值错误，程序执行失败，请重新复制正确的token值");
          continue; // 跳过当前行的后续操作
        }
        var status = [];
        for (day = 1; day <= lastDayOfMonth; day++) {
          try {
            var access_token2 = 'Bearer ' + access_token; // 构建包含访问令牌的请求头
            // 领取奖励
            let data4 = HTTP.post(
              "https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",
              JSON.stringify({ "signInDay": day }),
              { headers: { "Authorization": access_token2 } }
            );
            data4 = data4.json(); // 将响应数据解析为 JSON 格式
            var claimStatus = data4["success"]; // 获取奖励状态
            if (claimStatus === false) {
              log("账号：" + phone + " - 第 " + day + " 天奖励领取失败");
              status.push(day)
            }
          } catch {
            log("单元格【" + tokenColumn + row + "】内的token签到失败");
            continue; // 跳过当前行的后续操作
          }
        }
        if (status.length==0) {
          log(phone + " - 本月奖励领取成功");
        }
        else {
          var text = "";
          for (i = 0; i < status.length-1; i++) {
            text += status[i]+"、";
          }
          text += status[status.length];
          log(phone + " - 除第 " + text + " 天奖励领取失败外，本月其余天数均成功");
        }
      } else {
        log(phone + " 不签到");
      }
    }
  }
  log("自动领取未领取奖励完成。");
}


![5](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/48376ebb-651a-41f8-9673-e42ba1c1485f)
## 6.取得阿里云盘token方法如下：先通过浏览器打开阿里云盘官网并登录网页版：https://www.aliyundrive.com/drive/ 登录成功后，打开开发者工具 点击 Console进入控制台 ② 在控制台输入  JSON.parse(localStorage.token).refresh_token  复制返回的32位字符串，不要复制双引号
![6](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/b8a89b54-356b-4895-9bf7-541f3d140d36)
## 7.将刚才复制的refresh_token粘贴到表格A1列中，并自定义其他信息
![7](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/212ccb4f-211a-4410-941b-de6de0e0e195)
## 8.打开脚本编辑器点击运行测试一下，如果出现运行成功日志，就可说明配置完成
![8](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/133f8dd8-b75e-47c2-a8e2-057f73e027de)
# 9.在文档页，打开效率→高级开发→定时任务→创建任务，选择每天，确定一个合适的自动执行时间，选择你创建的脚本，点击确认
![9](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/8639c733-0dbb-46e9-96d5-6feb3b34f043)
## PS: Token值会在一个月左右刷新一次，届时直接替换就可以了

## 利用pushplus推送签到消息 https://www.pushplus.plus/
![10](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/3b383356-7646-408c-84ac-c43a7e8440d8)
![11](https://github.com/alantang1977/aliyun-signin-action/assets/107459091/3e7996e9-f604-48a0-916e-b0d63cf9875c)


