# EmailServer

## 构建

在构建此应用之前, 你需要安装 [Node.js](https://nodejs.org).<br>
基于 KOA 用 redis 的消息队列，发送邮件

```bash
$ git clone https://github.com/guyuezhai/EmailServer.git
$ cd EmailServer
$ npm install
$ node app
```

## 需要在config.js  配置自己的个人信息

```bash

 email:{

        host: 'smtp.163.com',  // 主机
        port:  465,             //SMTP端口
        from: 'from@163Email', // sender address
        authorizationCode: '发邮箱授权密码',  // 替换为自己邮箱的授权码
        queueEmail: "queue_email"  //邮件消息队列
  
    }

```

## 要发送邮件的信息及类型，\<由于发送邮件服务从项目中单独分离出来，向消息队列中添加消息内容不再该项目中,消息队列添加如下，如需要，自行补全

```bash

const sendResetPwdEmail = async function(toEmail){
    
    let subject = "邮件主题";
    let htmlContent = "邮件内容";

    let message = JSON.stringify({
        toEmail:toEmail,      // 发送给目标人物
        subject:subject,
        content:htmlContent
    });

    let success = await CacheUtils.rpush(QUEUE_EMAIL,message);  // 向消息队列中添加待发送邮件信息， redis.rpush() 方法
    
    if(success){

        return true;
    }

    return false;
}

```
##  email/email.js 获取消息队列中的内容

```bash

setInterval( async function(){

    let queueEmail = await RedisUtils.lpop(QUEUE_EMAIL);  // 根据 QUEUE_EMAIL 从 redis 的消息队列中获取，要发送邮件的信息及类型
    
    if(queueEmail){
        
        let queueMsg = JSON.parse(queueEmail);   // 解析JOSN字符串为对象
        let name = queueMsg.name;
        let toEmail = queueMsg.toEmail;         // 要发送给 ***
        let subject = queueMsg.subject;         // 邮件主题
        let emailContent = queueMsg.content;    // 邮件内容
        name = name?name:'默认主题名';
        return await sendEmail(FROM, name, toEmail, subject, emailContent);
        
    }

}, intervalTime);

```
## 真正发送邮件的方法在文件utils/email_utils.js 中

```bash
const send = async function(fromEmail, name, toEmail, subject, content){

    if(!fromEmail || !toEmail || !subject || !content || !name){
        console.error('必要参数不存在！');
        return;
    }
    if(!RegUtils.isEmail(fromEmail) || !RegUtils.isEmail(toEmail)){
        console.error('参数不合法');
        return;
    }
   
    let mail = {
  
        from: `${name}<${fromEmail}>`,
        to: toEmail,
        subject: subject,
        html: content
    
    }

    return  transporter.sendMail( mail )
            .then(function(data){
                console.log(data);
                return true;
            })
            .catch(function(err){
                console.error(err);
                return false
            });

   
}
```