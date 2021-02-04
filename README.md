# sqlitedemo

How to integrate SQLite Database with Spring Boot?

Spring Boot doesn’t provide a straightforward way to integrate SQLite database compared to other databases such as MySQL , MongoDB etc.

SQLite is the most used database engine in the world as SQLite website claims.

And yet it is a surprise Spring Boot doesn’t treat it the same way as it treats other databases.

To integrate SQLite into your spring boot application you need to do one major step which you don’t have to do for other databases.

That is , to add an SQL Dialect.

Let’s see step by step , how to integrate SQLite database with Spring Boot starting from scratch.

**STEP 1:**

Let’s create a base project using Spring Initializr.

Go to ***https://start.spring.io/*** and generate a project template.

Add Spring Web and Spring Data JPA dependencies as shown below:

**STEP 2:**

Add sqlite jdbc driver dependency to your pom.xml:
```
<dependency>
        <groupId>org.xerial</groupId>
        <artifactId>sqlite-jdbc</artifactId>
        <version>3.16.1</version>
 
    </dependency>
```  

Note : It is important to include the version number else spring boot complains that driver class is not present.

STEP 3:

Add SQLDialect for SQLite database :

Create a class and include the below content :
```
package com.springboot.sqlite;
 
import java.sql.Types;
 
import org.hibernate.dialect.Dialect;
import org.hibernate.dialect.function.StandardSQLFunction;
import org.hibernate.dialect.function.SQLFunctionTemplate;
import org.hibernate.dialect.function.VarArgsSQLFunction;
import org.hibernate.Hibernate;
import org.hibernate.type.StringType;
 
public class SQLDialect extends Dialect {
    public SQLDialect() {
        registerColumnType(Types.BIT, "integer");
        registerColumnType(Types.TINYINT, "tinyint");
        registerColumnType(Types.SMALLINT, "smallint");
        registerColumnType(Types.INTEGER, "integer");
        registerColumnType(Types.BIGINT, "bigint");
        registerColumnType(Types.FLOAT, "float");
        registerColumnType(Types.REAL, "real");
        registerColumnType(Types.DOUBLE, "double");
        registerColumnType(Types.NUMERIC, "numeric");
        registerColumnType(Types.DECIMAL, "decimal");
        registerColumnType(Types.CHAR, "char");
        registerColumnType(Types.VARCHAR, "varchar");
        registerColumnType(Types.LONGVARCHAR, "longvarchar");
        registerColumnType(Types.DATE, "date");
        registerColumnType(Types.TIME, "time");
        registerColumnType(Types.TIMESTAMP, "timestamp");
        registerColumnType(Types.BINARY, "blob");
        registerColumnType(Types.VARBINARY, "blob");
        registerColumnType(Types.LONGVARBINARY, "blob");
        // registerColumnType(Types.NULL, "null");
        registerColumnType(Types.BLOB, "blob");
        registerColumnType(Types.CLOB, "clob");
        registerColumnType(Types.BOOLEAN, "integer");
 
        registerFunction("concat", new VarArgsSQLFunction(StringType.INSTANCE, "", "||", ""));
        registerFunction("mod", new SQLFunctionTemplate(StringType.INSTANCE, "?1 % ?2"));
        registerFunction("substr", new StandardSQLFunction("substr", StringType.INSTANCE));
        registerFunction("substring", new StandardSQLFunction("substr", StringType.INSTANCE));
    }
 
    public boolean supportsIdentityColumns() {
        return true;
    }
 
    public boolean hasDataTypeInIdentityColumn() {
        return false; // As specify in NHibernate dialect
    }
 
    public String getIdentityColumnString() {
        // return "integer primary key autoincrement";
        return "integer";
    }
 
    public String getIdentitySelectString() {
        return "select last_insert_rowid()";
    }
 
    public boolean supportsLimit() {
        return true;
    }
 
    protected String getLimitString(String query, boolean hasOffset) {
        return new StringBuffer(query.length() + 20).append(query).append(hasOffset ? " limit ? offset ?" : " limit ?")
                .toString();
    }
 
    public boolean supportsTemporaryTables() {
        return true;
    }
 
    public String getCreateTemporaryTableString() {
        return "create temporary table if not exists";
    }
 
    public boolean dropTemporaryTableAfterUse() {
        return false;
    }
 
    public boolean supportsCurrentTimestampSelection() {
        return true;
    }
 
    public boolean isCurrentTimestampSelectStringCallable() {
        return false;
    }
 
    public String getCurrentTimestampSelectString() {
        return "select current_timestamp";
    }
 
    public boolean supportsUnionAll() {
        return true;
    }
 
    public boolean hasAlterTable() {
        return false; // As specify in NHibernate dialect
    }
 
    public boolean dropConstraints() {
        return false;
    }
 
    public String getAddColumnString() {
        return "add column";
    }
 
    public String getForUpdateString() {
        return "";
    }
 
    public boolean supportsOuterJoinForUpdate() {
        return false;
    }
 
    public String getDropForeignKeyString() {
        throw new UnsupportedOperationException("No drop foreign key syntax supported by SQLiteDialect");
    }
 
    public String getAddForeignKeyConstraintString(String constraintName, String[] foreignKey, String referencedTable,
            String[] primaryKey, boolean referencesPrimaryKey) {
        throw new UnsupportedOperationException("No add foreign key syntax supported by SQLiteDialect");
    }
 
    public String getAddPrimaryKeyConstraintString(String constraintName) {
        throw new UnsupportedOperationException("No add primary key syntax supported by SQLiteDialect");
    }
 
    public boolean supportsIfExistsBeforeTableName() {
        return true;
    }
 
    public boolean supportsCascadeDelete() {
        return false;
    }
}
```
**STEP 4:**

Configure database details in ***application.properties*** file.

Include the below properties:
```	
spring.jpa.database-platform=com.springboot.sqlite.SQLDialect
spring.jpa.hibernate.ddl-auto=update
 
 
spring.datasource.url = jdbc:sqlite:sqlitesample.db
spring.datasource.driver-class-name = org.sqlite.JDBC
 
spring.datasource.username = admin
spring.datasource.password = admin
```
***spring.jpa.database-platform*** refers to the dialect file created in the previous step

***spring.jpa.hibernate.ddl-auto = update*** , says to update the tables whenever they are modified , if not present create them

***spring.datasource.url*** refers to the url of the database. Here we are creating a database named sqlitesample.db inside the project root folder itself.

***spring.datasource.driver-class-name*** refers to the jdbc driver for sqlite.

***spring.datasource.username*** is the user name for the database

***spring.datasource.password*** is the password for the database

**STEP 5:**

Run the **SpringBoot application**.

Now you can see the database created at the root folder:

**STEP 6:**

Now let’s create a table through Spring Boot (it uses hibernate by default).

Let’s create an entity class named Person :
```	
package com.springboot.sqlite;
 
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
 
@Entity
public class Person {
 
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;
 
    private String name;
 
    private String message;
 
    /**
     * @return the id
     */
    public Integer getId() {
        return id;
    }
 
    /**
     * @param id the id to set
     */
    public void setId(Integer id) {
        this.id = id;
    }
 
    /**
     * @return the name
     */
    public String getName() {
        return name;
    }
 
    /**
     * @param name the name to set
     */
    public void setName(String name) {
        this.name = name;
    }
 
    /**
     * @return the message
     */
    public String getMessage() {
        return message;
    }
 
    /**
     * @param message the message to set
     */
    public void setMessage(String message) {
        this.message = message;
    }
 
}
```
Now run the application again.

Spring Boot automatically generates a table for you.

This is because of the below statement in ***application.properties*** file :
```	
spring.jpa.hibernate.ddl-auto=update
```
Let’s confirm it.

I have installed SQLiteStudio in my machine.

Here is the table generated viewed using the studio:

**STEP 7:**

Let’s add some data through Spring Boot and check if it inserts them into the table.

To load data into a table when Spring Boot application starts , do the following :

Create a file named ***data.sql*** under resources folder and include your insert queries there:

data.sql:
```
INSERT INTO PERSON(name,message) values("Varun","Hey this is Varun");
INSERT INTO PERSON(name,message) values("Joe","Hey this is Joe");
```
Add the below property to ***application.properties***. This tells spring to look for queries to fire on application startup:
```	
spring.datasource.initialization-mode=always
```
That’s it.

Now let me check if records are inserted.

They are inserted!

We have integrated SQLite database with Spring Boot.

You can exclude Spring Web dependency cited in the first step if you just want to check the database integration .

If you do so , your application won’t create a tomcat server instance and listen for requests. It will immediately shut down once the records are inserted.

To the above code , you can add a Spring Data Repository and then create a REST controller using Spring Web ( spring web dependency should be included for this) and create a REST service. This can then be exposed to UI clients or other API consumers.

You can also use Spring Data REST (https://spring.io/projects/spring-data-rest) to automatically expose your repository as a REST service.

Here is the entire code :
```
https://github.com/vijaysrj/sqlitedemo
```
