# sqoop-commands
Sqoop commands

MySql

username : root
password : root
localhost:3306

--connect to mysql--

mysql -u root -p
root

mysql -u root -h localhost -p
root

CREATE DATABASE mytrainingdb;

use mytrainingdb;

create table temptable ( sno INT, name VARCHAR(180) NOT NULL, gender VARCHAR(20) NOT NULL);
create table address ( id INT PRIMARY KEY, addname VARCHAR(180) NOT NULL, city VARCHAR(20) NOT NULL);
insert into temptable (1,"raj","Male");
alter table temptable add column `id` int(10) unsigned primary KEY AUTO_INCREMENT;

create table address ( addressid INT, addname VARCHAR(180) NOT NULL, city VARCHAR(20) NOT NULL);
alter table address add column `id` int(10) unsigned primary KEY AUTO_INCREMENT;
insert into address (addressid,addname,city,id) values (1,'s4 street','madurai',null);
insert into address (addressid,addname,city,id) values(2,"rain street","chennai",null);
insert into address (addressid,addname,city,id) values(3,"summar street","delhi",null);
insert into address (addressid,addname,city,id) values(4,"wine street","banglore",null);
insert into address (addressid,addname,city,id) values(5,"mobile street","noida",null);
insert into address (addressid,addname,city,id) values(6,"kachara street","some city",null);

if auto increement is set, then to insert give "null" as the column value.
INSERT INTO temptable values (13,'ajith','k',null)

INSERT INTO temptable values (113,'loosu',null,null);
INSERT INTO temptable values (131,'pakki',null,null);
INSERT INTO temptable values (145,'thum',null,null);

#Add index

ALTER TABLE temptable ADD INDEX (sno);

-- TO create NO PK table --

create table temp_nopk as select * from temptable;

Sqoop

/usr/local/sqoop*
user : hduser
password : hduser

mysql-connector placed in /usr/local/sqoop**/lib/*
Exported Hadoop home and Hadoop mapred path in /usr/local/sqoop**/coonf/spark-env.sh/


Sqoop commands

-- To list databases -----

sqoop list-databases \
  --connect "jdbc:mysql://localhost:3306" \
  --username root \
  --password root

--Impport from mysql database to HDFS--

sqoop import \
  --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
  --username=root \
  --password=root \
  --table temptable \
  --target-dir=/sqoop/temptable_1

-- list table ---

sqoop list-tables \
  --connect jdbc:mysql://localhost:3306/mytrainingdb \
  --username root \
  --password root

-- query using sqoop eval--

sqoop-eval \
  --connect "jdbc:mysql://localhost:3306" \
  --username root \
  --password root \
  --query "select * from mytrainingdb.temptable"


sqoop eval --connect jdbc:mysql://localhost:3306/mytrainingdb --username root --password root --query "select * from temptable"

sqoop eval --connect jdbc:mysql://localhost:3306/mytrainingdb --username root --password root --query "INSERT INTO temptable values (13,'ajith','k',null)"

-- create table ----

sqoop-eval \
  --connect "jdbc:mysql://localhost:3306"/mytrainingdb \
  --username root \
  --password root \
  --query "CREATE TABLE dummy (i INT)"

-- Insert values ---

sqoop-eval \
  --connect "jdbc:mysql://localhost:3306"/mytrainingdb \
  --username root \
  --password root \
  --query "INSERT INTO dummy VALUES (1)"

sqoop-eval \
  --connect "jdbc:mysql://localhost:3306"/mytrainingdb \
  --username root \
  --password root \
  --query "SELECT * FROM dummy"

-- Sqoop Import ---

sqoop import \
  --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
  --username root \
  --password root \
  --table temptable \
  --target-dir /sqoop/test_1

--target-dir will create part files in the specified path

--warehouse-dir will creata a folder with name of the table on the specified path

sqoop import   --connect "jdbc:mysql://localhost:3306/mytrainingdb"   --username root   --password root   --table temptable   --warehouse-dir /sqoop/warehouse

BY DEFAULT, sql rows will be delimited by comma by sqoop


Execution Life Cycle
Here is the execution life cycle of Sqoop.

Connect to source database and get metadata
Generate java file with metadata and compile to jar file
Apply boundaryvalsquery to apply split logic, default 4
Use split boundaries to issue queries against source database
Each thread will have different connection to issue the query
Each thread will get mutually exclusive sub set of the data
Data will be written to HDFS in a separate file per thread

--Customize mappers ---

By default the number of input splits is 4.Hence 4 mappers will be running.To customize number of splits or number of mappers,
we can set --num-mappers to 1

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_3/ \
 --num-mappers 1

Managing Directories
By default sqoop import fails if target directory already exists
Directory can be overwritten by using –delete-target-dir
Data can be appended to existing directories by saying –append

--delet target directory for every run--

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_3/ \
 --num-mappers 1 \
 --delete-target-dir


--append to the existing directory---

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_3/ \
 --num-mappers 1 \
 --append


sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_3/ \
 --append


----Customizing split logic ---

By default number of mappers is 4, it can be changed with –num-mappers
Split logic will be applied on primary key, if exists

If primary key does not exists and if we use number of mappers more than 1, then sqoop import will fail

So we are creating a table with no primary key and we will run sqoop import with default splis (which is 4)

TO create no pk table , 
create table temp_nopk as select * from temptable;

-- Without mentioning --num-mappers--

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temp_nopk \
 --warehouse-dir /sqoop/nopk_1/ \
 --delete-target-dir

When running above command ,we will get below error.

Import failed: No primary key could be found for table temp_nopk. Please specify one with --split-by or perform a sequential import with '-m 1'.

-- Now with --num-mappers

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temp_nopk \
 --warehouse-dir /sqoop/nopk_1/ \
 --num-mappers 1 \
 --delete-target-dir


To avoid import failure in this case,we can use --split-by option

#Things to remember for split-by
Column should be indexed.Else whole table will be scanned which is a costlier operation.
Value in the field should be sparse
Value in the field should be sequence generated or evenly increemented
It should not have null values
It need not be unique

For performance reason choose a column which is indexed as part of split-by clause
If there are null values in the column, corresponding records from the table will be ignored
Data in the split-by column need not be unique, but if there are duplicates then there can be skew in the data while importing (which means some files might be relatively bigger compared to other files)

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temp_nopk \
 --warehouse-dir /sqoop/nopk_1/ \
 --split-by id \
 --delete-target-dir


skewed????

How to spilt by used on fied which is string or text.
Let us see now with below command

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temp_nopk \
 --warehouse-dir /sqoop/nopk_1/ \
 --split-by name \
 --delete-target-dir

We will get below error.

Import failed: java.io.IOException: Generating splits for a textual index column allowed only in case of "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" property passed as a parameter

To overcome this error,add "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" to the import command on the top

sqoop import \
 -Dorg.apache.sqoop.splitter.allow_text_splitter=true \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temp_nopk \
 --warehouse-dir /sqoop/nopk_1/ \
 --split-by name \
 --delete-target-dir

*** BUt there will be a side effect of text split,which needs to checked in the internet


Auto reset to one mapper when no primary key is available..This can be added in the import command and it will check if the table has the primary key,if not it will set the mapped to 1.


sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temp_nopk \
 --warehouse-dir /sqoop/nopk_1/ \
 --autoreset-to-one-mapper \
 --delete-target-dir

This will run the command with one mapper since no primary key is available.

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/temp_5/ \
 --autoreset-to-one-mapper \
 --delete-target-dir

This will run 4 mappers because primary key is available.Hence the autoreset property will not be applied.

**** File Formats *********


By Default,sqoop import writes the data in text file.No need to specify text file format in the command.It comes by default

Other file formats,

text file (default)
sequence file (binary file format)
avro (binary json format)
parquet (
columnar file format)
orc

-- File format -> --as-sequence

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/temp_6/ \
 --as-sequencefile \
 --delete-target-dir

-- File format -> --as-parquetfile

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/temp_parq/ \
 --as-parquetfile \
 --delete-target-dir

-- File format -> --as-avrodatafile

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/temp_avro/ \
 --as-avrodatafile \
 --delete-target-dir


Avrodatafile is not working...Need to check



*** Compression ***

Buy default ,compression is diabled.

It can be enabled using --compress in the import command.And by default gzip algorithm is the compression-codec
Compression can be applied for any file format.

  --compression-codec org.apache.hadoop.io.compress.GzipCodec

  --compression-codec org.apache.hadoop.io.compress.SnappyCodec

-- Default is Gzip -- 
sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/temp_6/ \
 --as-textfile \
 --compress \
 --delete-target-dir

--GzipCodec--

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/temp_6/ \
 --as-textfile \
 --compress \
--compression-codec org.apache.hadoop.io.compress.GzipCodec \
 --delete-target-dir

--SnappyCodec--

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/temp_6/ \
 --as-textfile \
 --compress \
--compression-codec org.apache.hadoop.io.compress.SnappyCodec \
 --delete-target-dir

Note : Go to /etc/hadoop/conf and check core-site.xml for supported compression codecs


***** Boundary query *****

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --boundary-query "select min(sno),max(sno) from temptable"

--- Primary key in the boundary query ---

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --boundary-query "select min(id),max(id) from temptable where id>=4"

---Giving another column as boundary query ----
** But this is not working
sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --boundary-query "select min(sno),max(sno) from temptable where sno>=4"

--Even if i create an index it is not working.

-- To hardcode range --- 
*** This range is applied only primary key

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --boundary-query "select 4,7"


***** Transformation and filtering *****

** Only table and/or column can be used together -- is mutualy exclusive with query.
** query can be used only

-- IF I want only particular columns to be imported then we can use --column ---

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --columns name,id \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir 


sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --columns name,id \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --num-mappers 1

If I want to join tables or if i want to have many joins,we need use --query because we cannot mention both table names in --table.

Join query : select * from temptable t,address a where t.id = a.addressid;

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --num-mappers 1 \
 --query "select * from temptable t,address a where t.id = a.addressid

** The above command will throw error becuase if we use query we cannot give warehouse-dir ,bcoz we are joining two tables and warehouse dir will have to create a folder with the table name.Since we are using joining it is not possible to create a folder with table name.

Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
18/07/06 07:59:02 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
18/07/06 07:59:02 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
Must specify destination with --target-dir. 
Try --help for usage instructions.


Hence in this case we need to use --target-dir

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --target-dir /sqoop/test_13/ \
 --delete-target-dir \
 --num-mappers 1 \
 --query "select * from temptable t,address a where t.id = a.addressid

** this command will again throw an error as below

8/07/06 08:01:15 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
18/07/06 08:01:15 INFO tool.CodeGenTool: Beginning code generation
18/07/06 08:01:15 ERROR tool.ImportTool: Import failed: java.io.IOException: Query [select * from temptable t,address a where t.id = a.addressid] must contain '$CONDITIONS' in WHERE clause.
	at org.apache.sqoop.manager.ConnManager.getColumnTypes(ConnManager.java:332)
	at org.apache.sqoop.orm.ClassWriter.getColumnTypes(ClassWriter.java:1872)

** So if we are using --query ,we need add $CONDITIONS. This is need for sqoop to run and doesnot mean any logic to us.It is internal stuff

So,

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --target-dir /sqoop/test_13/ \
 --delete-target-dir \
 --num-mappers 1 \
 --query "select t.id,t.name,t.gender,a.addname,a.city from temptable t,address a where t.id = a.addressid and \$CONDITIONS"

The above command works will give a result with a join.Pleas note,we are using --num-mappers as 1.So no issue.

However if we remove and run the command.

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --target-dir /sqoop/test_13/ \
 --delete-target-dir \
 --query "select t.id,t.name,t.gender,a.addname,a.city from temptable t,address a where t.id = a.addressid and \$CONDITIONS"

We will get below error,

Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
18/07/06 08:16:42 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
18/07/06 08:16:42 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
When importing query results in parallel, you must specify --split-by.
Try --help for usage instructions.

So,add --split-by to tell sqoop to know the primary key to be used for splits

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --target-dir /sqoop/test_13/ \
 --delete-target-dir \
 --query "select t.id,t.name,t.gender,a.addname,a.city from temptable t,address a where t.id = a.addressid and \$CONDITIONS" \
 --split-by t.id

Please note in the above command, we cannot give the column name as "id" in --split-by instaed we need to give t.id.

Else let us try with allias as below,

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --target-dir /sqoop/test_13/ \
 --delete-target-dir \
 --query "select t.id as id,t.name,t.gender,a.addname,a.city from temptable t,address a where t.id = a.addressid and \$CONDITIONS" \
 --split-by id

It is still throwin error with even alias.So we need to give t.id

#table and/or columns are mutualy exclusive
Either you can only table
Either you can have both table and column
Either you can only have query

For --query, --split-by is mandatory if num-mappers is greated than 1.
#quert should have have a placeholder for \$CONDITIONS with backward slash


**** Delimiters and handling nulls ******


Added few null values to gender column in temptable

Run a simple sqoop command

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir

Out put is,

1,raj,Male,1
1,sekar,Male,2
1,siva,female,3
1,trisha,female,4
10,nayan,female,5
13,ajith,k,6
13,ajith,k,7
19,jason,st,8
113,loosu,null,9
131,pakki,null,10
145,thum,null,11


By default,delimiter is comma(,) and new line character at end.

Suppose we have null values in our mysql table and if we import the same through sqoop to hdfs.We will have null values in the hdfs as above.

Sometimes we need to set null values to some default data.In this case we can use below command.
--null-string <null-string>	The string to be written for a null value for string columns
--null-non-string <null-string>	The string to be written for a null value for non-string columns


sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --null-string unknown


On running the above command we will get below output.

1,raj,Male,1
1,sekar,Male,2
1,siva,female,3
1,trisha,female,4
10,nayan,female,5
13,ajith,k,6
13,ajith,k,7
19,jason,st,8
113,loosu,unknown,9
131,pakki,unknown,10
145,thum,unknown,11


null values are replaced by unknown

Below are 6 different delimiter we can use.

Argument	Description
--enclosed-by <char>	Sets a required field enclosing character
--escaped-by <char>	Sets the escape character
--fields-terminated-by <char>	Sets the field separator character
--lines-terminated-by <char>	Sets the end-of-line character
--mysql-delimiters	Uses MySQL’s default delimiter set: fields: , lines: \n escaped-by: \ optionally-enclosed-by: '
--optionally-enclosed-by <char>	Sets a field enclosing character


Suppose if I need to change the field delimiter to "tab" .BY default is "comma"

Suppose if I need to change the lines terminated to ":" .BY default is "new line"

sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --null-string unknown \
 --fields-terminated-by "\t" \
 --lines-terminated-by  ":"

So the output is

1	raj	Male	1:1	sekar	Male	2:1	siva	female	3:1	trisha	female	4:10	nayan	female	5:13	ajith	k	6:13	ajith	k	7:19	jason	st	8:113	loosu	unknown	9:131	pakki	unknown	10:145	thum	unknown	11:


sqoop import \
 --connect "jdbc:mysql://localhost:3306/mytrainingdb" \
 --username root \
 --password root \
 --table temptable \
 --warehouse-dir /sqoop/test_13/ \
 --delete-target-dir \
 --null-string unknown \
 --fields-terminated-by "\000" \
 --lines-terminated-by  ":"
