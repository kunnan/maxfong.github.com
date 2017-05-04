---
layout: post
title:  "PHP虚拟空间支持多站点"
date:   2017-05-03 00:00:00
categories: PHP
published: true
---

```
<!Doctype html>
<html xmlns=http://www.w3.org/1999/xhtml>
<head>                          
 <meta http-equiv=Content-Type content="text/html;charset=utf-8">
 <meta content=always name=referrer>
 <title>welcome</title>
</head>
<body>
<?php
$host = $_SERVER['HTTP_HOST'];
if (strstr($host, "x.com") ) {
	require "x/index.html";
}
elseif (strstr($host, "y.com") ) {
	require "y/index.php";
}
else {
	echo "welcome";
}
?>
</body>
</html>
``` 
