---
layout:     post
title:      uploab-labs靶机总结 
subtitle:   uploab-labs
date:       2018-08-14 				# 时间
author:     sh1llc0de					# 作者
catalog: true 						# 是否归档
tags:								#标签
    - CTF
---

# uploab-labs靶机总结
>前段时间花了点时间做了一下，学到了蛮多东西。  
[题目](/file/upload-labs.zip)

## upload-labs
* ### Pass-1
	核心代码:
	```
function checkFile() {
    var file = document.getElementsByName('upload_file')[0].value;
    if (file == null || file == "") {
        alert("请选择要上传的文件!");
        return false;
    }
    //定义允许上传的文件类型
    var allow_ext = ".jpg|.png|.gif";
    //提取上传文件的类型
    var ext_name = file.substring(file.lastIndexOf("."));
    //判断上传文件类型是否允许上传
    if (allow_ext.indexOf(ext_name + "|") == -1) {
        var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
        alert(errMsg);
        return false;
    }
}

	```
	这关是第一关，想想也不会太难，于是我直接上传了一张图片马，生成图片马也很简单，直接cmd下输入copy 1.jpg/b+1.php/a 2.jpg，然后我们直接上传这张图片，在burp下面改后缀获取返回包来尝试是不是前段验证。然后发现就是前段验证，直接菜刀连接就可以了。
* ### Pass-2
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . $_FILES['upload_file']['name'];
                $is_upload = true;

            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = $UPLOAD_ADDR.'文件夹不存在,请手工创建！';
    }
}

	```
	这关我用第一关的方法做居然过了。然后我就看了下源码发现了这关判断的是mime，因为我使用的是图片马，所以我的mime也就是image/jpeg，也就在白名单之内。
	![](https://s1.ax1x.com/2018/08/14/PgIHER.png)
* ### Pass-3
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if(!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR. '/' . $_FILES['upload_file']['name'])) {
                 $img_path = $UPLOAD_ADDR .'/'. $_FILES['upload_file']['name'];
                 $is_upload = true;
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
	```
	用之前的方法发现不行，然后00截断好像也不行，最好还是老老实实的看了一下源代码，发现是黑名单。然后绕过黑名单的话我收集了一些关于php的。php、php3、php4、php5、phtml、pht等。但是在这个环境下，只有php、phtml、php3可用。所以我们只要上传这些后缀结尾的文件就行。
* ### Pass-4
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2","php1",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2","pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . $_FILES['upload_file']['name'];
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传!';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}

	```
	这关我学到了蛮多东西的，首先刚开始我是黑盒测试，然后完全没思路的时候看了一下源码。发现所有的东西好像都过滤了。我实在不想看提示，想了想之前做的一道题目，好像可以上传.htaccess文件。当时我没做出来，因为不会编写。然后今天看了一下，这关就可以利用.htaccess文件来改变文件的解析情况。我们可以这样编写AddType application/x-httpd-php .sh1llc0de，这样所以的.sh1llc0de结尾的文件就会被解析成php文件。菜刀还是一样连接。
* ### Pass-5
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空

        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
	```
	这题首先把上面的.htaccess过滤了。然后感觉完全没思路。然后最后被自己的智商打败了。发现这个没写第4关的转换大小写。。。所以我们可以通过大小写绕过来getshell。
* ### Pass-6
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = $_FILES['upload_file']['name'];
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}

	```
	刚开始做的时候还是没有思路，后面看了源代码之后发现似乎少了去空格的一步，所以我们可以在上传文件的时候加上一个空格，比如a.php ，这样我们就绕过了。
* ### Pass-7
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
	```
	这关我发现好像都是比上一关少过滤点什么，虽然做题的时候都会想到，但是我觉得渗透时应该不会，我想弄个fuzz来试一试。先说这道题吧。没有过滤点，然后查了一下，发现windows的特性会自己去除末尾的点，所以我们只要传一个a.php.就行了
* ### Pass-8
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
	```
	这次有了上次的经验做起来蛮快的，可以知道少了一个对::$DATA的处理，然后百度一下发现没有，最后在大佬博客下看到了::DATA和.的原理一样都是windows特性。所以我们可以上传一个::$DATA结尾的文件就行。
* ### Pass-9
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $_FILES['upload_file']['name'])) {
                $img_path = $UPLOAD_ADDR . '/' . $file_name;
                $is_upload = true;
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}

	```
	这题我先测试了一下去点的那个功能，发现只能去除一个点(在有空格过滤情况下)，然后我们想到下面还要去除空格，这不正合我们的意吗，只要上传一个a.php. .首先.被去除了，然后空格被去除了，只剩下a.php.又被windows特性去除.所以我们就getshell了。
* ### Pass-10
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists($UPLOAD_ADDR)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = str_ireplace($deny_ext,"", $file_name);
        if (move_uploaded_file($_FILES['upload_file']['tmp_name'], $UPLOAD_ADDR . '/' . $file_name)) {
            $img_path = $UPLOAD_ADDR . '/' .$file_name;
            $is_upload = true;
        }
    } else {
        $msg = $UPLOAD_ADDR . '文件夹不存在,请手工创建！';
    }
}
	```
	查看源代码的时候发现少了许多，仔细看了一下，发现变成了替换了，把所以黑名单里面的全部替换成空，这里就有一个文件了。文件双写问题，就是a.pphphp这样的，他会把php替换成空，然后最后剩下的还是a.php。不记得是什么时候做的一个sql注入过滤也是这样绕的。
* ### Pass-11
	核心代码:
	```
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        }
        else{
            $msg = '上传失败！';
        }
    }
    else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}

	```
	第一次白名单，发现自己连00截断玩的都不溜了，只能去看了下书。。然发现我的00截断没有记错，只是环境不行，这里的00截断主要是因为是甩$_GET方式直接拼接的，所以我们可以在url那里使用%00来截断。
	![](https://s1.ax1x.com/2018/08/14/PgoRZd.png)
	这样我们就上传了一个3.php文件，后面的通通被%00给截断了。
* ### Pass-12
	核心代码:
	```
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        }
        else{
            $msg = "上传失败";
        }
    }
    else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}

	```
	这里用的就是$_POST来直接拼接的，也就是我上题做错的那个方式，因为post不会像get对%00进行自动解码，所以我们应该改hex了。
	![](https://s1.ax1x.com/2018/08/14/Pgooz8.png)
	然后转到hex界面找到空格对应的hex码20并且改成00,就行。
	![](https://s1.ax1x.com/2018/08/14/Pgo7QS.png)
	然后就和上面一题一样连接了。
* ### Pass-13
	这里好像是单纯让我们上传一张图片马，考察的可能是图片马的生成吧。但是我第一直用的都是图片马。。所以这里就直接上传，也不要我们连接啥的。对了，它这个判断方式是读取2字节的文件头。
* ### Pass-14
	这里好像也是让我们上传图片马，用的getimagesize函数来判断文件类型的。还是可以用cmd生成的图片马直接绕过。。我测试的时候发现正常的jpg文件都无法上传，后面做了张png的图片马就行了。
* ### Pass-15
	和前面三题一样还是上传图片马，这里用的是php_exif扩展模块来判断文件类型的。还是可以直接用cmd生成的图片马绕过，这里的话jpg也可以用来 。
* ### Pass-16
	核心代码:
	```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])){
    // 获得上传文件的基本信息，文件名，类型，大小，临时文件路径
    $filename = $_FILES['upload_file']['name'];
    $filetype = $_FILES['upload_file']['type'];
    $tmpname = $_FILES['upload_file']['tmp_name'];

    $target_path=$UPLOAD_ADDR.basename($filename);

    // 获得上传文件的扩展名
    $fileext= substr(strrchr($filename,"."),1);

    //判断文件后缀与类型，合法才进行上传操作
    if(($fileext == "jpg") && ($filetype=="image/jpeg")){
        if(move_uploaded_file($tmpname,$target_path))
        {
            //使用上传的图片生成新的图片
            $im = imagecreatefromjpeg($target_path);

            if($im == false){
                $msg = "该文件不是jpg格式的图片！";
            }else{
                //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".jpg";
                $newimagepath = $UPLOAD_ADDR.$newfilename;
                imagejpeg($im,$newimagepath);
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = $UPLOAD_ADDR.$newfilename;
                unlink($target_path);
                $is_upload = true;
            }
        }
        else
        {
            $msg = "上传失败！";
        }

    }else if(($fileext == "png") && ($filetype=="image/png")){
        if(move_uploaded_file($tmpname,$target_path))
        {
            //使用上传的图片生成新的图片
            $im = imagecreatefrompng($target_path);

            if($im == false){
                $msg = "该文件不是png格式的图片！";
            }else{
                 //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".png";
                $newimagepath = $UPLOAD_ADDR.$newfilename;
                imagepng($im,$newimagepath);
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = $UPLOAD_ADDR.$newfilename;
                unlink($target_path);
                $is_upload = true;               
            }
        }
        else
        {
            $msg = "上传失败！";
        }

    }else if(($fileext == "gif") && ($filetype=="image/gif")){
        if(move_uploaded_file($tmpname,$target_path))
        {
            //使用上传的图片生成新的图片
            $im = imagecreatefromgif($target_path);
            if($im == false){
                $msg = "该文件不是gif格式的图片！";
            }else{
                //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".gif";
                $newimagepath = $UPLOAD_ADDR.$newfilename;
                imagegif($im,$newimagepath);
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = $UPLOAD_ADDR.$newfilename;
                unlink($target_path);
                $is_upload = true;
            }
        }
        else
        {
            $msg = "上传失败！";
        }
    }else{
        $msg = "只允许上传后缀为.jpg|.png|.gif的图片文件！";
    }
}
	```
	哇，居然还是图片马上传，用来3中方式来判断文件类型，文件后缀名，文件mime，和imagecreatefromgif函数来判断，但是好像还是可以直接过。。
* ### Pass-17
	这题的话就好玩了，我随便学习了一波时间竞争，原来记得也有一道题目，是包含文件的，当时也是因为不会，所以就没做出来，现在学习了一波，没什么问题了。首先我们burp先抓到上传的包，然后我们要设置一下。先放到爆破模块，然后设置为null payload，线程也要用20以上。
	![](https://s1.ax1x.com/2018/08/14/PgTSzT.png)
	然后就是用python脚本来替代人去访问网站了。
	```
import requests
url="http://127.0.0.1/upload/sj.php"  #要访问的网站目录
while 1:
	r = requests.get(url)
	if "flag" in r.text:     #设置关键字来判断是否打印
	print r.text
	```
	直接运行这个脚本，burp也开始跑，时间竞争讲的是运行和速度，500次里面最多可以访问10次。
* ### Pass-18
	这关通过查看源代码得知了还有一种特殊的情况就是文件上传成功，但是没有被重命名，所以猜想有条件竞争。用上关的方法测试了一下，完全没有问题。
* ### Pass-19
	首先在windows下，由于做前面的题做多了，还有很快的发现这里没有过滤大小写。所以我们完全可以上传test.PHP，还有一些win特性的方式。但是作者说必须在linux下面做，所以，上述通通不成立。在linux下的也挺好做的。查看源码下我们很快发现和上面一道post的00截断很想，所以我们可以直接那样做。


	
	


