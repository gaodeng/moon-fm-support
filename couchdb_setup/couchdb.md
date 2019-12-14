####Couchdb安装(Ubuntu 18.04: bionic)
```
echo "deb https://apache.bintray.com/couchdb-deb bionic main" \
    | sudo tee -a /etc/apt/sources.list
    
curl -L https://couchdb.apache.org/repo/bintray-pubkey.asc \
    | sudo apt-key add -

sudo apt-get update && sudo apt-get install couchdb
```

![](1.png)

![](2.png)

设置 Couchdb 管理密码
![](3.png)



如果需要支持 https 协议,可以用 nginx 做反向代理 。具体配置看下面链接文档
https://docs.couchdb.org/en/stable/best-practices/reverse-proxies.html#reverse-proxying-with-nginx
另外 由于同步的时候会做一些批量操作,可能单个请求数据量比较大, nginx 做反向代理还需注意 413 – Request Entity Too Large 的问题  https://www.cyberciti.biz/faq/linux-unix-bsd-nginx-413-request-entity-too-large/
需要把 client_max_body_size  设置为比较大一点的值 比如 client_max_body_size 20M;


Couchdb 默认的端口是 5984. 假如你的服务器 IP 地址是 114.215.127.34
那么访问 http://114.215.127.34:5984/ 会看到类似下面的响应,你可以通过访问这个地址来看看 Couchdb 是否正确的安装了

```json
{
    "couchdb": "Welcome",
    "version": "2.3.1",
    "git_sha": "c298091a4",
    "uuid": "dca9d353459be384c6c309aacba1aa1f",
    "features": [
        "pluggable-storage-engines",
        "scheduler"
    ],
    "vendor": {
        "name": "The Apache Software Foundation"
    }
}

```



Couchdb 的 Web 管理端地址是 http://114.215.127.34:5984/_utils  用户名是 admin,密码是安装 couchdb 的时候设置的密码

先通过这个 Web 管理端把 Couchdb 配置为 couch_peruser 模式 (couch_peruser 模式下 couchdb 会为每个_users 数据库中的每个新用户创建一个数据库) 
![](6.png)


#### 创建 _users 数据库

```
curl -X PUT http://admin:admin_password@114.215.127.34:5984/_users \
     -H "Accept: application/json" \
     -H "Content-Type: application/json" \
     
```

#### 在 _users 数据库中创建用户

用 couchdb 管理员账户,创建一个用户名为 jan 密码为 apple 的用户
```
curl -X PUT http://admin:admin_password@114.215.127.34:5984/_users/org.couchdb.user:jan \
     -H "Accept: application/json" \
     -H "Content-Type: application/json" \
     -d '{"name": "jan", "password": "apple", "roles": [], "type": "user"}'
     
```

用户 jan 创建后，你发现 Couchdb 同时为这个用户创建了一个数据库 userdb-6a616e (数据库名是前缀 userdb-  加用户名的hex字符串)

那么这个用户的同步地址就是  http://jan:apple@114.215.127.34:5984/userdb-6a616e

另外 couchdb 默认的情况下是允许匿名创建用户的。可以修改 _users 数据库的权限来禁止匿名创建用户.具体可看这个帖子 https://serverfault.com/questions/742184/couchdb-user-creation-without-authentication-standard-behavior

