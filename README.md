# ethereum-wallet

*********
# 创建项目
### 新建文件夹wallet
### cd到项目中 执行下面命令
` npm init -y `
### 安装项目依赖
1. ` npm install --save express`
#### express 作为项目的后端
2. ` npm install --save body-parser`
#### body-parser 用来支持post请求

3. `npm install --save web3`
#### 用来操作以太坊钱包

# 创建前端页面
## 前端依赖
#### bootstrap 和jQuery

# 创建后端
#### 在根目录下新建index.js，使用express作为服务器

```
 var express = require("express");  
var app = express();
app.listen(8081,function(){
    console.log('server start...@8081')
});

```

#### 设置静态文件路径

```
var path = require('path');
app.use(express.static(path.join(__dirname, 'public')));
```

##### 目的是把public设置问静态文件路径，这样再浏览器访问public目录下的文件的时候，可以不用添加 public，比如如果想访问index.html,就可以直接在地址栏输入http:/localhost:8081/index.html

### 使用body-parser中间件支持post请求

```
 var bodyParser = require('body-parser');
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```
### 写测试接口
```
app.get("/", function(req, res){
    res.sendFile(__dirname + "/public/index.html");
})
```
#### __dirname这是一个全局变量，用来获取当前文件的路劲，这里用它获取到的是index.js的路劲，我们通过拼接就得到了index.html的文件绝对路径了

### 接下来启动服务器，在命令行（注意要在当前项目目录下）执行
``` node .\index.js  ```
### 然后打开浏览器，输入http://localhost:8081/ 可以顺利访问

********************

# 功能实现

### 获取所有账户 进页面就全部显示
#### 在index.js中添加代码
```
//获取所有用户
app.get('/accounts',function(req,res){
    web3.eth.getAccounts(function(error, result){
        res.send(result)
    })
})
```
#### 前端页面调用这个接口
```
        var accounts = {};

        function gtAccounts(){
            $.get('http://localhost:8081/accounts',function(accs){
                console.log(accs)
                for(var i = 0;i < accs.length;i++){
                    accounts[accs[i]] = 0;
                }
                
                for( k in accounts){
                    getBalance(k)
                }
                
                
            })
        }
        gtAccounts()
```

### 注册新账户
#### 后端接口
```
//注册用户
app.post("/register", function(req, res){

    var password = req.body.password;
    console.log('password:');
    console.log(password);
    web3.eth.personal.newAccount(password)
    .then(function(addr){
        console.log('新增账户:',addr)
        res.send({address:addr,balance:0})
    });

})
```

#### 前端调用
```
$("#addaccount").click(function(){

            if($("#password").val() != ""){
                $.post('http://localhost:8081/register/',
                {password:$("#password").val()},
                function(res){
                    console.log(res)
                    accounts[res.address] = res.balance;
                    showAccountList();
                })
            }
            $("#password").val("")
        })
```

### 转账功能
#### 后端接口
```
//发送以太币
app.post("/sendcoin", function(req, res){

    var address_from = req.body.address_from;  
    var address_to = req.body.address_to;
    var trans_value = req.body.trans_value;
    var password = req.body.trans_password;
    
    web3.eth.personal.unlockAccount(address_from,password,9999,function(){
        console.log('unlock accounts ok')
        web3.eth.sendTransaction({
            from: address_from,
            to: address_to,
            value: web3.utils.toWei(trans_value,"ether"),
        },function(err,transactionHash){
            if(!err){
                console.log('transactionHash:',transactionHash)
                res.send({msg:"ok",hash:transactionHash});
            }else{
                console.log('-------------error-----------')
                console.log(err)
                console.log('-------------error-----------')
            }
        })
    });
    
}) 
```

#### 前端调用
```
$("#trans_btn").click(function(){

            var address_from = $("#address_from").val();
            var address_to = $("#address_to").val();
            var trans_value = $("#trans_value").val();
            var trans_password = $("#trans_password").val();
            if(address_from != '' && address_to != '' && trans_value != "" && trans_password != ""){
                $.post("http://localhost:8081/sendcoin/",
                {
                    address_from,
                    address_to,
                    trans_value,
                    trans_password
                },function(res){
                    if(res.msg == 'ok'){
                        accounts[address_to] = trans_value;
                        showAccountList();
                        // getBalance(address_from)
                        // getBalance(address_to)
                        
                    }
                })
            }
            // $("#address_from").val("");
            $("#address_to").val("");
            $("#trans_value").val("");
            $("#trans_password").val("");

        })
```









