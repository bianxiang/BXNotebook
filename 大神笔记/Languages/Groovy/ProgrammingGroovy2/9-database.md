## 9. 数据库 ##

Groovy SQL (GSQL)是对JDBC的封装。

### 9.1 准备数据 ###

建表语句：

	create database if not exists weatherinfo;
	use weatherinfo;
	
	drop table if exists weather;
	create table weather (
	  city varchar(100) not null,
	  temperature integer not null
	);
		
	insert into weather (city, temperature) values ('Austin', 48);
	insert into weather (city, temperature) values ('Baton Rouge', 57);
	insert into weather (city, temperature) values ('Jackson', 50);
	insert into weather (city, temperature) values ('Montgomery', 53);
	insert into weather (city, temperature) values ('Phoenix', 67);
	insert into weather (city, temperature) values ('Sacramento', 66);
	insert into weather (city, temperature) values ('Santa Fe', 27);
	insert into weather (city, temperature) values ('Tallahassee', 59);

### 9.2 连接数据 ###

要连接数据库，先创建一个`groovy.sql.Sql`对象。利用其静态方法`newInstance()`。

	def sql = groovy.sql.Sql.newInstance('jdbc:mysql://localhost:3306/weatherinfo',
		userid, password, 'com.mysql.jdbc.Driver')
	println sql.connection.catalog

### 9.3 查询 ###

	sql.eachRow('SELECT * from weather') {
		printf "%-20s%s\n", it.city, it[1]
	}

`eachRow()`传给闭包的是一个`GroovyResultSet`对象。可以利用列名（`it.city`）或索引（`it[1]`）访问。

如果想获取元数据，可以向`eachRow()`传两个闭包。第一个闭包只会被执行一次，它会收到一个`ResultSetMetaData`对象，第二个闭包收到每一行数据。

	processMeta = { metaData ->
	  metaData.columnCount.times { i ->
	    printf "%-21s", metaData.getColumnLabel(i+1)
	  }
	  println ""
	}
	  
	sql.eachRow('SELECT * from weather', processMeta) {
	  printf "%-20s %s\n", it.city, it[1]
	}

如果想处理所有行但不想使用迭代器，可以使用`rows()`方法。它返回一个`ArrayList`实例。

	rows = sql.rows('SELECT * from weather')
	println "Weather info available for ${rows.size()} cities"

`firstRow()`只返回结果集中的第一行。

### （未）9.4 Transforming Data to XML ###

### 9.5 使用 DataSet ###

`Sql`的`dataSet()`方法接受一个表名，返回一个虚拟代理——此时不会发出实际的查询直到我们开发遍历。遍历使用`each()`方法。可以利用`findAll()`过滤结果。调用`findAll()`也不会触发实际查询，于是，DataSet非常高效，只会返回我们需要的数据。
	
	dataSet = sql.dataSet('weather')
	citiesBelowFreezing = dataSet.findAll { it.temperature < 32 }
	println "Cities below freezing:"
	citiesBelowFreezing.each {
	  println it.city
	}

### 9.6 插入和更新 ###

DataSet还可以用于插入数据。`add()`接收一个Map创建一个数据库行。

	println "Number of cities: " + sql.rows('SELECT * from weather').size()
	dataSet.add(city: 'Denver', temperature: 19)
	println "Number of cities: " + sql.rows('SELECT * from weather').size()

或者使用传统方式，利用`Sql.execute()`或`executeInsert()`发SQL语句：

	temperature = 50
	sql.executeInsert("""INSERT INTO weather(city, temperature)
						VALUES('Oklahoma City', ${temperature})""")
	println sql.firstRow(	
		"SELECT temperature from weather WHERE city = 'Oklahoma City'")

### （未）9.7 访问 Microsoft Excel ###








