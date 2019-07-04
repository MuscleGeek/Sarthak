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
```php
function all($cat=NULL,$order =NULL) {
    $sql = "SELECT * FROM posts";
    if (isset($order)) 
      $sql .= "order by ".mysql_real_escape_string($order);  
    $results= mysql_query($sql);
    $posts = Array();
    if ($results) {
      while ($row = mysql_fetch_assoc($results)) {
        $posts[] = new Post($row['id'],$row['title'],$row['text'],$row['published']);
      }
    }
    else {
      echo mysql_error();
    }
    return $posts;
  }
```
<br/>
Upon looking into this function we can't get anything useful to us for exploitation so let's see any other page from website interface... 

So next page is ```post.php```
Let's analyse it :)

### Post.php

**Source Code**
```php
<?php
  $site = "PentesterLab vulnerable blog";
  require "header.php";
  $post = Post::find(intval($_GET['id']));
?>
  <div class="block" id="block-text">
    <div class="secondary-navigation">
      <div class="content">
      <?php 
            echo $post->render_with_comments(); 
      ?> 
     </div>

      <form method="POST" action="/post_comment.php?id=<?php echo htmlentities($_GET['id']); ?>"> 
        Title: <input type="text" name="title" / ><br/>
        Author: <input type="text" name="author" / ><br/>
        Text: <textarea name="text" cols="80" rows="5">
        </textarea><br/>
        <input type="submit" name="submit" / >
      </form> 
    </div>

  </div>


<?php

  require "footer.php";
?>
```
<br/>

From the looks of this page we can clearly say that *id* parameter is not vulnerable to sql injection

```
Code:-$post = Post::find(intval($_GET['id']));
```
Anything which is being passed to ```id``` parameter will be converted to integer and even if we go ahead and try to insert a string or anything let's see what happens..

<<selection 007>>
<br/>

See that error now we know why it's happening so let's move forward ...

We shall see which file is handling those comments and to see that i will intercept with burp ...

<br/>
<<selection 008>>
<br/>
File which is handling the comments is ```post_comment.php``` So let's analyse it...

### post_comment.php

**Source Code**
```php
<?php
  $site = "PentesterLab vulnerable blog";
  require "header.php";
  $post = Post::find(intval($_GET['id']));
  if (isset($post)) {
    $ret = $post->add_comment();
  }
  header("Location: post.php?id=".intval($_GET['id']));
  die();
?>

```
On first look we can see again the ```classes\post.php```  was used and the functions were ```find``` as well as ```add_comment()```

Let's analyse both functions ...

**Find function**

```php
function find($id) {
    $result = mysql_query("SELECT * FROM posts where id=".$id);
    $row = mysql_fetch_assoc($result); 
    if (isset($row)){
      $post = new Post($row['id'],$row['title'],$row['text'],$row['published']);
    }
    return $post;
  
  }
```

We can see that this function is taking id parameter as argument and checking it if exists then after validiating it exists it's creating a object of ```Post``` class and passing the values to the constructor

***Constuctor***

```php
class Post{
  public $id, $title, $text, $published;
  function __construct($id, $title, $text, $published){
    $this->title= $title;
    $this->text = $text;
    $this->published= $published;
    $this->id = $id;
  }
```
We can see that this is accepting the values and assigning them to the local variables of this object...

Where the variables are the values which were passed from the comment section...<br/>
***Burpsuite captured***
```
title=test&author=sarthak&text=hello+world++++++++&submit=Submit
```
