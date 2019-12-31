---
cover: https://blog-1256184194.file.myqcloud.com/2019/12/27/590c427344948.jpg
title: think-rbac for thinkphp6.0
date: 2019-12-27 20:46
tags:
  - php
  - thinkphp
  - tools
categories:
  - 分享
keywords:
  - rbac
  - thinkphp
description: think-rbac for thinkphp6.0

---

# think-rbac for thinkphp6.0

> 之前做了一个thinkphp5.1版本的rbac接口管理，最近开始使用thinkphp6.0了，所以就可以开始升级我的rbac啦，哈哈哈

# 使用

1. 首先用composer安装

```shell
composer require lihq1403/think-rbac:^1.0
```

2. 发布配置文件

```shell
php think lihq1403:rbac-publish
```

会生成一个rbac.php的配置文件在config目录下

3. rbac数据库迁移

```
php think lihq1403:rbac-migrate
```

然后就会发现数据库多了5张表，表字段具体内容可看代码

```mysql
rbac_role  角色 表
rbac_user_role 用户角色 中间表
rbac_permission_group 权限组 表
rbac_permission 权限规则 表
rbac_role_permission_group 角色权限组 中间表
rbac_log 请求日志 表
```

4. thinkphp6里面使用
   在你需要用到权限的用户model里面，引入`RBACUser`，参考如下：

```php
<?php

namespace app\common\models;

use Lihq1403\ThinkRbac\traits\RBACUser;

/**
 * Class AdminUser
 * @package app\common\models
 */
class AdminUser extends BaseModel
{
    /**
     * 超级管理员id
     */
    const SUPER_ADMINISTRATOR_ID = 1;

    use RBACUser;
}
```

5. 配合路由使用

> 如果不想使用的话，可以参考```Lihq1403\ThinkRbac\controller\RBACController```做自己的自定义方法

路由参考如下：

```
// rbac 管理
Route::group('rbac', function () {

    // 角色管理
    Route::post('role', 'Lihq1403\ThinkRbac\controller\RBACController@addRole'); // 添加角色
    Route::put('role', 'Lihq1403\ThinkRbac\controller\RBACController@editRole'); // 修改角色
    Route::delete('role', 'Lihq1403\ThinkRbac\controller\RBACController@delRole'); // 删除角色
    Route::get('roles', 'Lihq1403\ThinkRbac\controller\RBACController@getRoles'); // 角色列表
    Route::get('role/permission-group', 'Lihq1403\ThinkRbac\controller\RBACController@roleHoldPermissionGroup'); // 角色拥有的权限列表
    Route::post('role/change-permission-group', 'Lihq1403\ThinkRbac\controller\RBACController@diffPermissionGroup'); // 角色更换的权限列表

    // 权限组管理
    Route::post('permission_group', 'Lihq1403\ThinkRbac\controller\RBACController@addPermissionGroup'); // 权限组新增
    Route::put('permission_group', 'Lihq1403\ThinkRbac\controller\RBACController@editPermissionGroup'); // 权限组编辑
    Route::delete('permission_group', 'Lihq1403\ThinkRbac\controller\RBACController@delPermissionGroup');// 权限组删除
    Route::get('permission_groups', 'Lihq1403\ThinkRbac\controller\RBACController@getPermissionGroups'); // 权限组列表

    // 权限管理
    Route::post('permission', 'Lihq1403\ThinkRbac\controller\RBACController@addPermission'); // 权限新增
    Route::put('permission', 'Lihq1403\ThinkRbac\controller\RBACController@editPermission'); // 权限编辑
    Route::delete('permission', 'Lihq1403\ThinkRbac\controller\RBACController@delPermission'); // 权限删除
    Route::get('permissions', 'Lihq1403\ThinkRbac\controller\RBACController@getPermissions'); // 权限列表

    // 管理员管理
    Route::post('admin-user/role', 'Lihq1403\ThinkRbac\controller\RBACController@userAssignRoles'); // 给管理员分配角色
    Route::delete('admin-user/role', 'Lihq1403\ThinkRbac\controller\RBACController@userCancelRoles'); // 给管理员删除角色
    Route::post('admin-user/sync-role', 'Lihq1403\ThinkRbac\controller\RBACController@userSyncRoles'); // 同步管理员角色

    // 日志管理
    Route::get('logs', 'Lihq1403\ThinkRbac\controller\RBACController@getLog'); // 获取日志
});
```

![](https://blog-1256184194.file.myqcloud.com/2019/12/27/5c73436844f49.png)

6. 配合中间件进行权限验证
   参考中间件如下：

```php
<?php

namespace app\middleware;

use app\common\facades\AdminAuth;
use app\common\models\AdminUser;
use Lihq1403\ThinkRbac\exception\ForbiddenException;
use Lihq1403\ThinkRbac\facade\RBAC;
use think\Request;

class AdminUserRbac
{
    /**
     * rbac中间件
     * @param Request $request
     * @param \Closure $next
     * @return mixed
     * @throws ForbiddenException
     */
    public function handle(Request $request, \Closure $next)
    {
        // 获取要验证的用户id
        $uid = AdminAuth::uid();

        // 超级管理员不需要进行权限控制
        if ($uid !== AdminUser::SUPER_ADMINISTRATOR_ID) {
            // 检查权限
            try {
                RBAC::can($uid);
            } catch (\Lihq1403\ThinkRbac\exception\ForbiddenException $exception){
                // 如果没有权限，会抛出Lihq1403\ThinkRbac\exception\ForbiddenException异常
                throw new ForbiddenException($exception->getMessage());
            }
        }

        // 记录日志
        RBAC::log($uid);

        return $next($request);
    }
}
```

7. 生成规则

> 在config/rbac.php里面，会有两个数组，用于权限初始化，```permission_group_list```和```permission_list```

对自己的系统进行权限配置之后，刷新权限规则

```shell
php think lihq1403:rbac-refresh -f yes
```

# 常用API

## 实例化

```php
$rbac = new Rbac();
// 或者使用门面
\Lihq1403\ThinkRbac\facade\RBAC::action();
```

## 具体方法

请参考```Lihq1403\ThinkRbac\lib\RBACLib```和```Lihq1403\ThinkRbac\Rbac```

# 源码地址
{% githubCard user:lihq1403 repo:think-rbac width:100% height:200 theme:default %}