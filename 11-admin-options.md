# 选项管理

在 admin 目录中添加一个 options.php 文件，处理获取和更新选项业务：

## 处理流程

```flow
s=>start: 开始
c1=>condition: 是否为 GET 请求
c2=>condition: 请求参数是否包含 key
c3=>condition: 是否缺少必要参数
c4=>condition: 判断是否存在该 key
o1=>operation: 获取全部选项
o2=>operation: 获取指定 key 对应的 value
o3=>operation: 更新指定 key 对应的 value
o4=>operation: 新增 option
o5=>operation: 响应 JSON
e=>end: 结束

s->c1
c1(yes)->c2
c2(yes)->o1->o5->e
c2(no)->o2->o5->e
c1(no)->c3
c3(yes)->e
c3(no)->c4
c4(yes)->o3->e
c4(no)->o4->e
```

## 具体实现

```php
<?php
/**
 * 获取或更新配置选项
 */

require '../functions.php';

// 设置响应类型为 JSON
header('Content-Type: application/json');

// 如果是 GET 请求，则获取指定配置
if ($_SERVER['REQUEST_METHOD'] == 'GET') {
  if (empty($_GET['key'])) {
    // echo json_encode(array(
    //   'success' => false,
    //   'message' => 'option key required'
    // ));
    // exit; // 不再往下执行
    // 查询全部数据
    $data = xiu_query('select * from `options`');
    echo json_encode(array(
      'success' => true,
      'data' => $data
    ));
    exit; // 不再往下执行
  }
  // 查询数据
  $data = xiu_query(sprintf("select `value` from `options` where `key` = '%s' limit 1;", $_GET['key']));
  // 返回
  if (isset($data[0][0])) {
    echo json_encode(array(
      'success' => true,
      'data' => $data[0][0]
    ));
  } else {
    echo json_encode(array(
      'success' => false,
      'message' => 'option key does not exist'
    ));
  }
  exit; // 不再往下执行
}

// 否则是更新或新增配置

if (empty($_POST['key']) || empty($_POST['value'])) {
  // 关键数据不存在
  echo json_encode(array(
    'success' => false,
    'message' => 'option key and value required'
  ));
  exit; // 不再往下执行
}

// 判断是否存在该属性
$exist = xiu_query(sprintf("select count(1) from `options` where `key` = '%s'", $_POST['key']))[0][0] > 0;

if ($exist) {
  $affected_rows = xiu_execute(sprintf("update `options` set `value` = '%s' where `key` = '%s'", $_POST['value'], $_POST['key']));
} else {
  $affected_rows = xiu_execute(sprintf("insert into `options` values (null, '%s', '%s')", $_POST['key'], $_POST['value']));
}

echo json_encode(array(
  'success' => $affected_rows > 0
));
```
