# 主从复制（双主），MyCat自动切换和读写分离

标签（空格分隔）： MySql Mycat

---

准备：
 1. 安装两个以上Mysql
 2. 安装好Mycat

主从复制
----


 - 两边配置 /etc/my.cnf 文件 添加以下配置
 >    symbolic-links=0
    log-bin = mysql-bin
    binlog_format=row
    log_slave_updates=1
    gtid_mode=ON
    enforce_gtid_consistency=ON
    bind-address=xxx.xxx.xxx.xxx   数据库ip
    server_id = xxx    # 这个主从两边不一致
    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid

 - 授权给从库，若是有多个就授权给多个（现在示例的是双主模式）（命令行输入）
>  grant replication slave on *.* to 'root'@'xxx.xxx.xxx.xxx' identified by '123456';
    xxx就是从的ip地址

 - 从库配置（命令行输入）

>  change master to master_host='xxx.xxx.xxx.xxx',
master_user='user',master_password='123456', master_auto_position=1;
start slave;
可以使用 show slave status; 查看主从关系是否配置好。关键点是：SLAVE_IO_RUNNING 和 SLAVE_SQL_RUNNING 都为YES
注意：若是出现 注意：若是出现Slave SQL Running 是NO ,则继续输入
stop slave;
change master to master_auto_position=0;
change master to master_log_file='mysql-bin.00000x', master_log_pos=xxxxx;
start slave;

***master_log_file和master_log_pos 要在 主库中 输入 show master status查看，并对应输入。***

 - 互为主从

 就是两边都配置主从关系，同样配置就好。

Mycat自动切换
---------

 1. 安装
 http://blog.csdn.net/yuyuntan/article/details/53160960
 2. 配置
    Server.xml

> 
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mycat:server SYSTEM "server.dtd">
    <mycat:server xmlns:mycat="http://io.mycat/">
	<system>
	<property name="useSqlStat">1</property>  <!-- 1为开启实时统计、0为关闭 -->
	<property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为关闭 -->
		<property name="sequnceHandlerType">2</property>
      <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
        <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--设置模拟的MySQL版本号-->
	<!-- <property name="processorBufferChunk">40960</property> -->
	<!-- 
	<property name="processors">1</property> 
	<property name="processorExecutor">32</property> 
	 -->
		<!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena-->
		<property name="processorBufferPoolType">0</property>
		<!--默认是65535 64K 用于sql解析时最大文本长度 -->
		<property name="maxStringLiteralLength">65535</property>
		<property name="sequnceHandlerType">0</property>
		<property name="backSocketNoDelay">1</property>
		<property name="frontSocketNoDelay">1</property>
		<!--<property name="processorExecutor">16</property>-->
		<!--
			<property name="serverPort">8066</property> <property name="managerPort">9066</property> 
			<property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property> 
			<property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
		<!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
		<property name="handleDistributedTransactions">0</property>
			<!--
			off heap for merge/order/group/limit      1开启   0关闭
		-->
		<property name="useOffHeapForMerge">1</property>
		<!--
			单位为m
		-->
		<property name="memoryPageSize">1m</property>
		<!--
			单位为k
		-->
		<property name="spillsFileBufferSize">1k</property>
		<property name="useStreamOutput">0</property>
		<!-- 单位为m -->
		<property name="systemReserveMemorySize">384m</property>
		<!--是否采用zookeeper协调切换  -->
		<property name="useZKSwitch">true</property>
	</system>
	<!-- 全局SQL防火墙设置 -->
	<!-- 
	<firewall> 
	   <whitehost>
	      <host host="127.0.0.1" user="mycat"/>
	      <host host="127.0.0.2" user="mycat"/>
	   </whitehost>
       <blacklist check="false">
       </blacklist>
	</firewall>
	-->
	<user name="root">
		<property name="password">123456</property>
		<property name="schemas">scheduler</property>
		<!-- 表级 DML 权限设置 -->
		<!-- 		
		<privileges check="false">
			<schema name="TESTDB" dml="0110" >
				<table name="tb01" dml="0000"></table>
				<table name="tb02" dml="1111"></table>
			</schema>
		</privileges>		
		 -->
	</user>
	<user name="user">
		<property name="password">user</property>
	<!--	<property name="schemas">TESTDB</property>   -->
		<property name="schemas">scheduler</property>
		<property name="readOnly">true</property>
	</user>
    </mycat:server>

Rule.xml
Schema.xml

> 先配置逻辑表， table 名就是对应数据库表的名 红色部分注意修改，其他地方不需改动
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

	<schema name="scheduler" checkSQLschema="false" sqlMaxLimit="3000">
		<!-- auto sharding by id (long) -->
		<table name="cgform_button" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_button_sql" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_enhance_java" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_enhance_js" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_field" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_ftl" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_head" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_index" dataNode="dn1" rule="sharding-by-murmur" /> 
		<table name="cgform_template" dataNode="dn1" rule="sharding-by-murmur" /> 
                                  .
                                  .
                                  .
		<!-- global table is auto cloned to all defined data nodes ,so can join
			with any table whose sharding node is in the same data node -->
	<!--	<table name="company" primaryKey="ID" type="global" dataNode="dn1,dn2,dn3" />
		<table name="goods" primaryKey="ID" type="global" dataNode="dn1,dn2" /> -->
		<!-- random sharding using mod sharind rule -->
	<!--	<table name="hotnews" primaryKey="ID" autoIncrement="true" dataNode="dn1,dn2,dn3"
			   rule="mod-long" />  -->
		<!-- <table name="dual" primaryKey="ID" dataNode="dnx,dnoracle2" type="global"
			needAddLimit="false"/> <table name="worker" primaryKey="ID" dataNode="jdbc_dn1,jdbc_dn2,jdbc_dn3"
			rule="mod-long" /> -->
	<!--	<table name="employee" primaryKey="ID" dataNode="dn1,dn2"
			   rule="sharding-by-intfile" />
		<table name="customer" primaryKey="ID" dataNode="dn1,dn2"
			   rule="sharding-by-intfile">
			<childTable name="orders" primaryKey="ID" joinKey="customer_id"
						parentKey="id">
				<childTable name="order_items" joinKey="order_id"
							parentKey="id" />
			</childTable>
			<childTable name="customer_addr" primaryKey="ID" joinKey="customer_id"
						parentKey="id" />
		</table>  -->
		<!-- <table name="oc_call" primaryKey="ID" dataNode="dn1$0-743" rule="latest-month-calldate"
			/> -->
	</schema>
	<!-- <dataNode name="dn1$0-743" dataHost="localhost1" database="db$0-743"
		/> -->
	<dataNode name="dn1" dataHost="localhost1" database="scheduler" />
     <dataNode name="dn2" dataHost="localhost1" database="scheduler" />
	<!-- <dataNode name="dn3" dataHost="localhost1" database="scheduler" /> -->
	<!--<dataNode name="dn4" dataHost="sequoiadb1" database="SAMPLE" />
	 <dataNode name="jdbc_dn1" dataHost="jdbchost" database="db1" />
	<dataNode	name="jdbc_dn2" dataHost="jdbchost" database="db2" />
	<dataNode name="jdbc_dn3" 	dataHost="jdbchost" database="db3" /> -->
		<dataHost name="localhost1" maxCon="1000" minCon="10" balance="3"
		writeType="0" dbType="mysql" dbDriver="native" switchType="2"  slaveThreshold="100">
		<heartbeat>show slave status</heartbeat>
		<!-- can have multi write hosts -->
		<writeHost host="hostM1" url="xxx.xxx.xxx.xxx:3306" user="root"
				   password="Root2017!"> 
			<!-- can have multi read hosts -->
			<readHost host="hostS2"  url="xxx.xxx.xxx.xxx:3306" user="root"
				   password="Root2017!"/>
		</writeHost>
		<writeHost host="hostM2" url="xxx.xxx.xxx.xxx:3306" user="root"
				   password="Root2017!"> 
		</writeHost>
	</dataHost>	<!--
		<dataHost name="sequoiadb1" maxCon="1000" minCon="1" balance="0" dbType="sequoiadb" dbDriver="jdbc">
		<heartbeat> 		</heartbeat>
		 <writeHost host="hostM1" url="sequoiadb://1426587161.dbaas.sequoialab.net:11920/SAMPLE" user="jifeng" 	password="jifeng"></writeHost>
		 </dataHost>

	  <dataHost name="oracle1" maxCon="1000" minCon="1" balance="0" writeType="0" 	dbType="oracle" dbDriver="jdbc">
	  <heartbeat>select 1 from dual</heartbeat>
		<connectionInitSql>alter session set nls_date_format='yyyy-mm-dd hh24:mi:ss'</connectionInitSql>
		<writeHost host="hostM1" url="jdbc:oracle:thin:@127.0.0.1:1521:nange" user="base" 	password="123456" > </writeHost> </dataHost>

		<dataHost name="jdbchost" maxCon="1000" 	minCon="1" balance="0" writeType="0" dbType="mongodb" dbDriver="jdbc">
		<heartbeat>select 	user()</heartbeat>
		<writeHost host="hostM" url="mongodb://192.168.0.99/test" user="admin" password="123456" ></writeHost> </dataHost>

		<dataHost name="sparksql" maxCon="1000" minCon="1" balance="0" dbType="spark" dbDriver="jdbc">
		<heartbeat> </heartbeat>
		 <writeHost host="hostM1" url="jdbc:hive2://feng01:10000" user="jifeng" 	password="jifeng"></writeHost> </dataHost> -->

	<!-- <dataHost name="jdbchost" maxCon="1000" minCon="10" balance="0" dbType="mysql"
		dbDriver="jdbc"> <heartbeat>select user()</heartbeat> <writeHost host="hostM1"
		url="jdbc:mysql://localhost:3306" user="root" password="123456"> </writeHost>
		</dataHost> -->
</mycat:schema>
 3.  Mycat 对应命令
启动MyCat： 
./mycat start 
查看启动状态： 
./mycat status 
停止： 
./mycat stop 
重启： 
./mycat restart


这时就可以插入数据，随意测试了，注意主从有没有断！

