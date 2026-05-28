low level在很多时候在传递数据时需要传递二进制。

但传递二进制非常繁琐，所以都使用Base64编码后进行传递。

**Base64编码本质**上是一种将二进制数据转成文本数据的方案。

在收到Base64后，使用Base64解码得到二进制后，**再解析此二进制得到你期望的**。

```
//power shell中使用此命令可以将base64 string解码得到16进制
[System.Convert]::FromBase64String("base64 string") | Format-Hex
```
