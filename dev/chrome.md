# Chrome浏览器信任自签名证书

右键快捷方式--->属性--->目标，在最后增加以下:

`--test-type --ignore-certificate-errors`

后缀通过该快捷方式打开后，就信任自签名证书了。