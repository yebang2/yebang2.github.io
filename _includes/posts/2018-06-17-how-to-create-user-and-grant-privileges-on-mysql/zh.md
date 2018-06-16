# 一. 创建用户
#### 命令：
```
CREATE USER 'username'@'host' IDENTIFIED BY 'password'
```
#### 说明：
- usernamme : 创建的用户名
- host : 指定用户在哪个主机上可以登录，如果是本地用户可以写localhost，如果你想要该用户可以再任意主机上都可以登录，可以使用通配符‘%’。假如： 你设置的host为192.168.2.100 这个用户只能在host为192.168.2.100这个主机上登录
- password： 该用户名的密码，密码可以为空。

#### 例子：
```
# 创建本地登录的用户jooua 密码为123456
CREATE USER 'jooua'@'localhost' IDENTIFIED BY '123456';

# 创建host为192.168.31.199主机上登录的用户jooua，密码是123456
CREATE USER 'jooua'@'192.168.31.199' IDENTIFIED BY '123456';

#创建可以任何地方登录的用户123456，密码是123465
CREATE USER 'jooua'@'%' IDENTIFIED BY '123456';

#创建可以再任何地方登录的用户jooua，密码为空
CREATE USER 'jooua'@'%' IDENTIFIED BY '';

```

# 二. 授权:
#### 命令:
```
GRANT PRIVILEGES ON database.table TO 'username'@'host'
```

#### 说明:
- PRIVILEGES: 指的是用户的操作权限，例如 SELECT,UPDATE,INSERT等，如果要全部授权就使用**ALL**
- database: 指的是数据库名
- table 指的是表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*

#### 例子:
```
# 授予jooua用户对larabbs数据库下的user表的SELECT和INSERT权限
GRANT SELECT,INSERT ON larabbs.user TO 'jooua'@'localhost';

# 授予jooua用户larabbs数据库全部表的操作权限
GRANT ALL ON larabbs.* TO 'jooua'@'localhost';

# 授予jooua用户全部数据库的操作权限
GRANT ALL ON *.* TO 'jooua'@'localhost';
```

#### 注意:
用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令:
```
GRANT PRIVILEGES ON databas.table TO 'username'@'host' WITH GRANT OPTION;
```

# 三.创建用户赋予权限
#### 命令：
```
GRANT ALL PRIVILEGES ON database.table TO 'username'@'host' IDENTIFIED BY 'password';
```

#### 说明:
- database: 表示数据库名
- table: 表示表名
- username: 表示用户名
- host: 主机
- password: 表示用户密码


#### 例子:

```
# 创建可以再任意主机上登录的用户jooua,并赋予larabbs数据库的所有操作权限
GRANT ALL PRIVILEGES ON larabbs.* TO 'jooua'@'%' IDENTIFIED BY '123456';
```

# 四. 撤销用户权限
#### 命令:
```
REVOKE PRIVILEGES ON database.table FROM 'username'@'host';
```
#### 说明:

- PRIVILEGES : 权限 
- database： 数据库名
- tablename： 表名

#### 例子:
```
REVOKE SELECT ON *.* FROM 'pig'@'%';
```
#### 注意:
假如你在给用户'jooua'@'%'授权的时候是这样的（或类似的）：GRANT SELECT ON test.user TO 'jooua'@'%'，则在使用REVOKE SELECT ON *.* FROM 'jooua'@'%';命令并不能撤销该用户对test数据库中user表的SELECT 操作。相反，如果授权使用的是GRANT SELECT ON *.* TO 'jooua'@'%';则REVOKE SELECT ON test.user FROM 'jooua'@'%';命令也不能撤销该用户对test数据库中user表的Select权限。

具体信息可以用命令SHOW GRANTS FOR 'pig'@'%'; 查看。

# 五.删除用户
#### 命令:
```
DROP USER 'username'@'host';
```