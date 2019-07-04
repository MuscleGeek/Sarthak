# Task - 1 

Hey guys welcome to my article about source-code analysis and finding vulnerabilites on a PHP website and for the test we will be using [this](https://pentesterlab.com/exercises/xss_and_mysql_file/course), it's a basic web-app vulnerable program for learning the web-app but we will analyse the source code and automate the exploitation with python. [Link](https://pentesterlab.com/exercises/xss_and_mysql_file/iso) for iso. Kudos to [Wetw0rk](https://github.com/wetw0rk/AWAE-PREP/tree/master/XSS%20and%20MySQL).

## OBJECTIVES TO BE ACHIEVED

1) Trigger xss --> Find the vulnearble function <br/>
2) COOKIE Stealing <br/>
3) SQL Injection  --> Code analysis of PHP files under the <br/>
4) OUTFILE to upload shell <br/>
5) RCE <br/>

## Hunt Began


So we will begin with analysing the source code of the first page which is index.php

### Index.php

**Source Code**

```php
<?php
  $site = "PentesterLab vulnerable blog";
  require "header.php";
  $posts = Post::all();
?>
  <div class="block" id="block-text">
    <div class="secondary-navigation">
      <div class="content">
      <?php 
        foreach ($posts as $post) {
            echo $post->render(); 
      } ?> 
     </div>
 
    </div>
  </div>


<?php


  require "footer.php";
?>
```
<br/>

First thing I noticed is the calling the function ```all``` from the class ```Post``` let's find where this class exist by using grep 

```
root@debian:/var/www# grep -iRn "Class Post" --color .
./classes/post.php:3:class Post{
root@debian:/var/www# 
```
**Command explanation**
  - i (it ignore the ignore case distinctions)
  - R (Search Recursively)
  - n (Displays Line number)
  
### Classes/post.php

So let's look into the function ```all```
<br/>
**Source Code** (We will only show the snippet which is important to us or for further analysis)

