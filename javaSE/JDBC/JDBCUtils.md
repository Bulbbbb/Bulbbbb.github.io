---
aliases: 
tags: 
---
*这里使用单例模式，保证程序中只有一个 connection，因为 connection 频繁关闭和创建会极大浪费资源。*
*并不是保证 JDBCUtils 只有一个对象。*
*注意资源释放的正确方式。*

```java
public class JDBCUtils_1 {  
	private static String driver;  
	private static String url;  
	private static String name;  
	private static String pwd;  
	private static Connection connection;
	// 在静态代码块内注册驱动，静态代码块只执行一次  
	static {  
		try {  
			Class.forName(driver);  
		} catch (ClassNotFoundException e) {  
			throw new RuntimeException(e);  
		}  
	}
	private JDBCUtils_1() {  
	}
	// 获取连接,单例，减少创建开销  
	public static Connection getConnection() {  
		try {  
			if (connection == null) {  
				synchronized (Connection.class) {  
					if (connection == null) {  
						connection = DriverManager.getConnection(url, name, pwd);  
					}  
				}  
			}  
			return connection;  
		} catch (SQLException e) {  
			throw new RuntimeException(e);  
		}  
	}
	// 释放资源  
	public void closeAll(Connection connection, Statement statement, ResultSet resultSet) {  
	// 正确释放资源  
		try {  
			if (resultSet != null) {  
				resultSet.close();  
			}  
		} catch (SQLException e) {  
			throw new RuntimeException(e);  
		} finally {  
			try {  
				if (statement != null) {  
					statement.close();  
				}  
			} catch (SQLException e) {  
				throw new RuntimeException(e);  
			} finally {  
				try {  
					if (connection != null) {  
						connection.close();  
					}  
				} catch (SQLException e) {  
					throw new RuntimeException(e);  
				}  
			}  
		}  
	}  
}
```