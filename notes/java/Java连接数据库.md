# Java 连接 MySQL 8.0 数据库

## 1. 准备

### 1.1 安装 *Maven* 的驱动包

首先在 *pom.xml* 的文件中添加如下内容。

``` xml
    <dependencies>

        <!-- 导入 MySQL 的Maven依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.13</version>
        </dependency>

    </dependencies>
```

然后点击右下角的 `import` 即可，然后就是等待安装了。

## 2. Java 传统的连接方式

+ 首先就是获取连接

``` java
// 动态加载 mysql 驱动
class.forName("com.mysql.cj.jdbc.Driver");
```

+ 获取连接信息

``` java
// 驱动程序名(Mysql 8.0)
String driver = "com.mysql.cj.jdbc.Driver"
//数据库地址， "?serverTimezone=GMT%2B8"是解决时区的问题
String url = "jdbc:mysql://localhost:3306/databaseName?serverTimezone=GMT%2B8";
// 用户名
String user = "mysqluser";
// 密码
String password = "password";
```

+ 加载驱动程序

``` java
// 加载驱动程序
Class.forName(driver);
// getConnection() 方法，连接 Mysql 数据库
Connection connection = DriverManager.getConnection(url, user, password);
// 创建 Statement 类对象，用来执行 SQL 语句
Statement statement = connection.createStatement();
// ResultSet 类，用来存放获取的结果集
ResultSet resultSet = statement.executeQuery(sql);
```

+ 完整代码

``` java
import java.sql.*;

public class MysqlConnect {
    public static Connection getConnection() throws ClassNotFoundException {
        String url = "jdbc:mysql://localhost:3306/databaseName";
        String user = "mysqluser";
        String password = "password";
        String driverClass = "com.mysql.cj.jdbc.Driver";
        Connection connection = null;
        Class.forName(driverClass);
        try {
            connection = DriverManager.getConnection(url, user, password);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return connection;
    }
}
```

## 3. 读取配置文件方式

[参考这篇文章](https://www.cnblogs.com/sebastian-tyd/p/7895182.html)

***第一种方式 `ClassLoder` 需要研究***

## 4. `c3p0` 连接池连接数据库

## 5. 数据库连接池
