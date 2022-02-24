### 0x01. Vulnerability details 
**vulnerability type**: **information leakage **

**vulnerability versions**: **piwigo 12.2.0 **

**vulnerability URL**: [http://x.x.x.x/piwigo/admin/maintenance_actions.php? action=phpinfo](http://10.92.66.148/piwigo/admin/maintenance_actions.php?action=phpinfo)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/496192/1645695186935-b1df7efa-9242-441e-87a0-f4be6943ba65.png#clientId=ub56e8d03-d575-4&from=paste&height=990&id=u0bb163a8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=990&originWidth=1652&originalType=binary&ratio=1&size=143644&status=done&style=none&taskId=ubdcffe58-3a6d-44ad-882b-afb33de0ba2&width=1652)
​

![image.png](https://cdn.nlark.com/yuque/0/2022/png/496192/1645695251836-85d5b470-1ca5-43f7-b607-1e28a6a15228.png#clientId=ub56e8d03-d575-4&from=paste&height=882&id=u4864f2e8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=882&originWidth=1615&originalType=binary&ratio=1&size=93174&status=done&style=none&taskId=u555d171e-d69f-4fdf-b81c-d03884b114a&width=1615)
### 0x02. Vulnerability Analysis
code for vulnerabilities: /admin/maintenance_actions.php
```bash
<?php
// +-----------------------------------------------------------------------+
// | This file is part of Piwigo.                                          |
// |                                                                       |
// | For copyright and license information, please view the COPYING.txt    |
// | file that was distributed with this source code.                      |
// +-----------------------------------------------------------------------+

// +-----------------------------------------------------------------------+
// |                                actions                                |
// +-----------------------------------------------------------------------+

$action = isset($_GET['action']) ? $_GET['action'] : '';

switch ($action)
{
  case 'phpinfo' :
  {
    phpinfo();
    exit();
  }
  case 'lock_gallery' :
  ...
```
When the value of the action parameter is phpinfo, the phpinfo() function is directly executed to print system information, which may cause information leakage. 
​

### 0x03. Vulnerability Fix
1. We recommend that you delete the phpinfo branch or enable a Cookie to verify whether it is an authorized user.
