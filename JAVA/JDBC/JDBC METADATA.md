# JDBC METADATA

标签（空格分隔）： JDBC

---

JDBC回顾

    private Connection getMySqlConnection() {
		String URL = "jdbc:mysql://127.0.0.1:3306/scheduler?useUnicode=true&amp;characterEncoding=utf-8";
		String USER = "root";
		String PASSWORD = "123456";
		Connection conn = null;
		// 1.加载驱动程序
		try {
			Class.forName("com.mysql.jdbc.Driver");
			conn = DriverManager.getConnection(URL, USER, PASSWORD);
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		// 2.获得数据库链接
		return conn;
	}

获取元数据（一）

    // 获取数据库相关信息
    public void getDataBaseInfo() {    
        Connection conn =  getMySqlConnection();  
        ResultSet rs = null;  
        try{    
             DatabaseMetaData dbmd = conn.getMetaData();  
             System.out.println("数据库已知的用户: "+ dbmd.getUserName());      
             System.out.println("数据库的系统函数的逗号分隔列表: "+ dbmd.getSystemFunctions());      
             System.out.println("数据库的时间和日期函数的逗号分隔列表: "+ dbmd.getTimeDateFunctions());      
             System.out.println("数据库的字符串函数的逗号分隔列表: "+ dbmd.getStringFunctions());      
             System.out.println("数据库供应商用于 'schema' 的首选术语: "+ dbmd.getSchemaTerm());      
             System.out.println("数据库URL: " + dbmd.getURL());      
             System.out.println("是否允许只读:" + dbmd.isReadOnly());      
             System.out.println("数据库的产品名称:" + dbmd.getDatabaseProductName());      
             System.out.println("数据库的版本:" + dbmd.getDatabaseProductVersion());      
             System.out.println("驱动程序的名称:" + dbmd.getDriverName());      
             System.out.println("驱动程序的版本:" + dbmd.getDriverVersion());    
               
             System.out.println("数据库中使用的表类型");      
             rs = dbmd.getTableTypes();      
             while (rs.next()) {      
                 System.out.println(rs.getString("TABLE_TYPE"));      
             }      
        }catch (SQLException e){    
            e.printStackTrace();    
        } finally{  
            try {
				conn.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
        }   
    }   


hh