# Web
## JobOnCampus
>Can you help me reveal the IP Address of joboncampus.com?
>Flag format: CyberX{ip_address_here}

Given that domain is joboncampus.com. So we can find the ip address from [who.is](https://who.is)
```
CyberX{109.123.238.148}
```

## Robots
>Someone told me that this CTFd platform has a special file that tells web crawlers where they can and cannot go. Can you find where these instructions are stored?

As like the name of the challenge and hint which said file that tells web crawlers mean we need to go to [robots.txt](http://34.142.170.50/robots.txt) of CTFd platform

robots.txt contain URL which to tell web crawler if it allow to access to the site or not
```
CyberX{r0b0ts_3ndp01nt!}
```
## Know Your HTTP
>Think you know HTTP? Test your protocol manipulation skills by bypassing three layers of HTTP tricks.

In this challenge we need to manipulate the http request accroding to the requirement 

![image](https://github.com/user-attachments/assets/0741c25b-6b47-4919-b536-672aa6828a3e)

First stage is the request must come from https://google.com so we need to put this in our request form Referer: <URL>

Here I use burpsuite to modified and send request

```
GET / HTTP/1.1
Host: 34.142.253.73:1340
Accept-Language: en-US,en;q=0.9
referer: https://google.com
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: td_cookie=1420749027
Connection: keep-alive

```
Second stage is to add parameter secret with the value= protocol we use.

![image](https://github.com/user-attachments/assets/6b473213-2f29-48a8-a247-8f411bd08e16)

Since we use GET so we need to add parameter on the url not at the body of the request and protocol we use is HTTP (must use lowercase) I think so

```
GET /?secret=http HTTP/1.1
```
Last stage is to find the correct HTTP method so we refer to [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
![image](https://github.com/user-attachments/assets/7fa45ce0-42a2-4989-b125-2b1886bff1b7)

SO we try HEAD,PUT,DELETE,CONNECT,PATCH,TRACE Is not the right one then we try OPTIONS which show us the allow method can use

```
HTTP/1.1 200 OK
Date: Thu, 26 Dec 2024 04:46:03 GMT
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Allow: GET, POST, FLAG, OPTIONS
Access-Control-Allow-Methods: GET, POST, FLAG, OPTIONS
```
Also we see there is FLAG method which give the flag
![image](https://github.com/user-attachments/assets/48de5910-f25c-469b-b712-ca789b6e9f24)

```
CyberX{533m5-l1k3-y0u-kn0w-h77p-:))))}
```

##Let Him Cook
>It's midnight and you're hungry. Pixel Bites only takes mobile orders, but you only have your laptop. There's also a secret recipe hidden somewhere in their system - can you find it? Time to get creative!

![image](https://github.com/user-attachments/assets/ce165d00-65a7-4c72-97df-9a12e621a235)
