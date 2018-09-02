---
layout:     post
title:      updataxml报错注入
subtitle:   sql
date:       2018-09-02 				# 时间
author:     sh1llc0de					# 作者
catalog: true 						# 是否归档
tags:								#标签
    - SQL
---

>取材来源:https://xz.aliyun.com/t/2451

## 搭建环境
首先我们贴下题目的源码吧
* index.php

```php
//index.php
<?php
include 'config.php';
$conn = new mysqli($servername, $username, $password, $dbname);
if ($conn->connect_error) {
    die("连接失败: ");
}

$sql = "SELECT COUNT(*) FROM users";
$whitelist = array();
$result = $conn->query($sql);
if($result->num_rows > 0){
    $row = $result->fetch_assoc();
    $whitelist = range(1, $row['COUNT(*)']);
}

$id = stop_hack($_GET['id']);
$sql = "SELECT * FROM users WHERE id=$id";

if (!in_array($id, $whitelist)) {
    die("id $id is not in whitelist.");
}

$result = $conn->query($sql);
if($result->num_rows > 0){
    $row = $result->fetch_assoc();
    echo "<center><table border='1'>";
    foreach ($row as $key => $value) {
        echo "<tr><td><center>$key</center></td><br>";
        echo "<td><center>$value</center></td></tr><br>";
    }
    echo "</table></center>";
}
else{
    die($conn->error);
}

?>
```

* config.php

```php
//config.php
<?php  
$servername = "localhost";
$username = "fire";
$password = "fire";
$dbname = "day1";

function stop_hack($value){
    $pattern = "insert|delete|or|concat|concat_ws|group_concat|join|floor|\/\*|\*|\.\.\/|\.\/|union|into|load_file|outfile|dumpfile|sub|hex|file_put_contents|fwrite|curl|system|eval";
    $back_list = explode("|",$pattern);
    foreach($back_list as $hack){
        if(preg_match("/$hack/i", $value))
            die("$hack detected!");
    }
    return $value;
}
?>
```

* 搭建CTF环境使用的sql语句

```sql
create database day1;
use day1;
create table users (
id int(6) unsigned auto_increment primary key,
name varchar(20) not null,
email varchar(30) not null,
salary int(8) unsigned not null );

INSERT INTO users VALUES(1,'Lucia','Lucia@hongri.com',3000);
INSERT INTO users VALUES(2,'Danny','Danny@hongri.com',4500);
INSERT INTO users VALUES(3,'Alina','Alina@hongri.com',2700);
INSERT INTO users VALUES(4,'Jameson','Jameson@hongri.com',10000);
INSERT INTO users VALUES(5,'Allie','Allie@hongri.com',6000);

create table flag(flag varchar(30) not null);
INSERT INTO flag VALUES('HRCTF{1n0rrY_i3_Vu1n3rab13}');
```

## 分析
根据and 1=1和and 1=2发现存在注入，然后进行爆表发现失败了。过滤了一些关键函数，这里最主
要是过滤了union，使我们不能见联合查询，盲注的话也被一些关键字限制了，所以就用上了报错注
入。

* updatexml
我也不总结了，有大佬写的挺好的  
<https://xz.aliyun.com/t/2160></https://xz.aliyun.com/t/2160>

* make_set  
<https://www.sh1llc0de.xyz/2018/09/02/make_set%E5%AD%A6%E4%B9%A0>

知道这些函数之后就可以进行注入了，这里我心里还有一个问题，怎么爆表
名？information_schema里面有or关键字。

payload:```and updatexml(1,make_set(3,"~",(select flag from flag)),1)```
