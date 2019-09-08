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

### 2.1 首先就是获取连接

``` java
// 动态加载 mysql 驱动
class.forName("com.mysql.cj.jdbc.Driver");
```

### 2.2 获取连接信息

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

### 2.3 加载驱动程序

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

### 2.4 完整代码

``` java
package mysql;

import java.sql.*;

/**
 * @author xys
 */
public class ConnectMysql {
    public static Connection getConnection() throws ClassNotFoundException, SQLException {
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
        if (connection != null) {
            System.out.println("数据库连接成功");
        } else {
            System.out.println("数据库连接失败");
            connection.close();
        }
        return connection;
    }

    public void getResult() throws ClassNotFoundException, SQLException {
        // 实例化 Statement 对象
        Statement statement = getConnection().createStatement();
        // 要执行的 Mysql 数据库操作语句（增、删、改、查）
        String sql = "";
        // 展开结果集数据库
        ResultSet resultSet = statement.executeQuery(sql);
        while (resultSet.next()) {
            // 通过字段检索
            int id = resultSet.getInt("id");
            String name = resultSet.getString("name");

            // 输出数据
            System.out.println("ID : " +id);
            System.out.println("name :" + name);
        }
        // 完成后需要依次关闭
        resultSet.close();
        statement.close();
        getConnection().close();
    }
}
```

## 3. 读取配置文件方式

[参考这篇文章](https://www.cnblogs.com/sebastian-tyd/p/7895182.html)

***第一种方式 `ClassLoder` 需要研究***

## 4. `c3p0` 连接池连接数据库

## 5. 数据库连接池
