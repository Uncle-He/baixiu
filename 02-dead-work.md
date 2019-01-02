# 准备工作

* 数据库设计
* 基础结构搭建

### 数据库设计
> 根据我们的业务需要设计数据库的结构，这个过程是每个项目开始时所必须的，一般由专门的 DBA 角色完成（很多没有划分的非常具体的公司由后端开发人员兼任）。

![数据库结构](https://github.com/Uncle-He/baixiu/blob/master/media/database.png)

### 选项表（options）
用于记录网站的一些配置属性信息，如：站点标题，站点描述等

| 字段 | 描述 | 备注 |
| --- | --- | --- |
| id | 🔑主键 |  |
| key | 属性值 | snake_case |
| value | 属性值 | JSON格式 |

### 用户表
用于记录用户信息

| 字段 | 描述 | 备注 |
| --- | --- | --- |
| id | 🔑主键 |  |
| slug | URL别名 |  |
| email | 邮箱 | 亦做登录名 |
| password | 密码 |  |
| nickname | 昵称 |  |
| avatar | 头像 | 头像URL路径 |
| bio | 简介 |  |
| status | 状态 | 未激活(unactivated)/激活(activated)/禁止(forbidden)/回收站(trashed) |

### 文章表（posts）
用于记录文章信息


| 字段 | 描述 | 备注 |
| --- | --- | --- |
| id | 🔑主键 |  |
| slug | URL别名 |  |
| title | 标题 |  |
| feature | 特色图像 | 图片URL路径 |
| created | 创建时间 |  |
| content | 内容 |  |
| views | 浏览次数 |  |
| likes | 点赞数 |  |
| status | 状态 | 草稿(drafted)/已发布(published)/回收站(trashed) |
| user_id | 🔗用户ID | 当前文章的作者ID |
| category_id | 🔗分类ID | 当前文章的分类ID |

### 分类表（categories）
用于记录文章分类信息

| 字段 | 描述 | 备注 |
| --- | --- | --- |
| id | 🔑主键 |  |
| slug | URL别名 |  |
| name | 分类名称 |  |

### 评论表（comments）
用于记录文章评论信息

| 字段 | 描述 | 备注 |
| --- | --- | --- |
| id | 🔑主键 |  |
| author | 作者 |  |
| email | 邮箱 |  |
| created | 创建时间 |  |
| content | 内容 |  |
| status | 状态 | 待审核(held)/准许(approved)/拒绝(rejected)/回收站(trashed) |
| post_id | 🔗文章ID |  |
| parent_id | 🔗父级ID |  |

[初始化数据库脚本](https://github.com/Uncle-He/baixiu/blob/master/media/baixiu.sql)

### 搭建项目结构
项目最基本的分为两个大块，前台（对大众开放）和后台（仅对管理员开放）。

一般在实际项目中，很多人会吧前台和后台分为两个项目去做，部署时也会分开部署：

这样做相互独立不容易乱，也更加安全。但是有点麻烦，所以我们采用更为常见的方案：让后台作为一个子目录出现。这样的话，大体结构就是：

### 基本的目录结构
```
└── baixiu ······································ 项目文件夹（网站根目录）
    ├── admin ··································· 后台文件夹
    │   └── index.php ··························· 后台脚本文件
    ├── static ·································· 静态文件夹
    │   ├── assets ······························ 资源文件夹
    │   └── uploads ····························· 上传文件夹
    └── index.php ······························· 前台脚本文件
```


```
└── baixiu ······································ 项目文件夹（网站根目录）
    ├── ......
    ├── static ·································· 静态文件夹
    │   ├── assets ······························ 资源文件夹
+   │   │   ├── css ····························· 样式文件夹
+   │   │   ├── img ····························· 图片文件夹
+   │   │   ├── js ······························ 脚本文件夹
+   │   │   └── venders ························· 第三方资源
    │   └── uploads ····························· 上传文件夹
+   │       └── 2017 ···························· 2017 年上传文件目录
    ├── ......
```

* `static` 目录中只允许出现静态文件。
* `assets` 目录中放置网页中所需的资源文件。
* `uploads` 目录中放置网站运营过程中上传的文件，如果担心文件过多，可以按年归档（一年一个文件夹）。

### 项目配置文件

由于在接下来的开发过程中，肯定又一部分公共的成员，例如数据库名称，数据库主机，数据库用户名密码等，这些数据应该放到公共的地方，抽象成一个配置文件 config.php 放到项目根目录下。

这个配置文件采用定义常量的方式定义配置成员：

```php
/**
 * 数据库主机
 */
define('DB_HOST', '127.0.0.1');
/**
 * 数据库用户名
 */
define('DB_USER', 'root');
/**
 * 数据库密码
 */
define('DB_PASS', 'wanglei');
/**
 * 数据库名称
 */
define('DB_NAME', 'baixiu');
```

在需要的时候可以通过`require`载入:

```php
// 载入配置文件
require 'config.php';

...

// 用到的时候
echo DB_NAME;
```

### 载入脚本的几种方式对比

* `require`
* `require_once`
* `include`
* `include_once`

* 共同点：

   * 都可以在当前 PHP 脚本文件执行时载入另外一个 PHP 脚本文件。
* `require` 和 `include` 不同点：

   * 当载入的脚本文件不存在时，`require` 会报一个致命错误（结束程序执行），而 `include` 不会
* 有 `once` 后缀的特点：

   * 判断当前载入的脚本文件是否已经载入过，如果载入了就不在执行
   
### 显示PHP错误信息
当执行 PHP 文件发生错误时，如果在页面上不显示错误信息，只是提示 500 Internal Server Error 错误，应该是 PHP 配置的问题，解决方案就是：找到 `php.ini` 配置文件中 `display_errors` 选项，将值设置为 `On`

```php
  ; http://php.net/display-errors
  ; display_errors = Off
  display_errors = On
  ; The display of errors which occur during PHP's startup sequence are handled
```

### 抽离公共部分
由于每一个页面中都有一部分代码（侧边栏）是相同的，分散到各个文件中，不便于维护，所以应该抽象到一个公共的文件中。

于是我们在 `admin` 目录中创建一个 `inc`（includes 的简称）子目录，在这个目录里创建一个 `sidebar.php` 文件，用于抽象公共的侧边栏 `<div class="aside"> ... </div>`，然后在每一个需要这个模块的页面中通过 `include` 载入：

```php
  <?php include 'inc/sidebar.php' ;?>
```

### 侧边栏的焦点状态
由于侧边栏在不同页面时，active class name 出现的位置不尽相同，所以我们需要区分当前 `sidebar.php` 文件是在哪个页面中载入的，从而明确焦点状态。
所以目前的关键问题就出现在了如何在 `sidebar.php` 中知道当前被哪个文件载入了。

通过查看 `include` 函数的文档发现：如果 `a.php` 通过 `include` 载入了 `b.php` 文件，那么在 `b.php` 文件中可以访问到 `a.php` 中定义的变量。

> [http://php.net/manual/zh/function.include.php](http://php.net/manual/zh/function.include.php)

借助这个特性，我们可以在各个页面中定义一个标识变量，然后在 sidebar.php 中通过这个标识变量区别不同页面的载入：

每一个菜单项 `<li>` 元素：

```php
<li<?php echo $current_page == 'dashboard' ? ' class="active"' : ''; ?>>
  <a href="index.php"><i class="fa fa-dashboard"></i>仪表盘</a>
</li>
```

对于有子菜单的菜单项，有一点例外：

```php
<li<?php echo in_array($current_page, array('posts', 'post-add', 'categories')) ? ' class="active"' : ''; ?>>
  <a href="#menu-posts"<?php echo in_array($current_page, array('posts', 'post-add', 'categories')) ? '' : ' class="collapsed"'; ?> data-toggle="collapse">
    <i class="fa fa-thumb-tack"></i>文章<i class="fa fa-angle-right"></i>
  </a>
  <ul id="menu-posts" class="collapse<?php echo in_array($current_page, array('posts', 'post-add', 'categories')) ? ' in' : ''; ?>">
    <li<?php echo $current_page == 'posts' ? ' class="active"' : ''; ?>><a href="posts.php">所有文章</a></li>
    <li<?php echo $current_page == 'post-add' ? ' class="active"' : ''; ?>><a href="post-add.php">写文章</a></li>
    <li<?php echo $current_page == 'categories' ? ' class="active"' : ''; ?>><a href="categories.php">分类目录</a></li>
  </ul>
</li>
```
