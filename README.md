
Create Linked server in Sql server to connect to the MySql Database
===================================================================


(A). Via graphical way
========================
http://www.thebuttonfactory.nl/?p=2204

i.  Create ODBC DSN to connect to MySql

 open odbc Data Source (or Run odbcad32 or 642) --> System DSN --> check following 2 drivers are exists
 MySql ODBC 5.3 ANSI DRIVER
 MySql ODBC 5.3 Unicode DRIVER
 
 Then fine. Otherwise download  & install : http://dev.mysql.com/downloads/connector/odbc/5.1.html
 
 add --> 
  Data Source Name = MySqlDb (Any name)
  Description = any desc
  TCP/IP Server = localhost
  port = 3306
  user = root
  password = mysqlpas (its my common pass with 8 length)
  database = ghtorrent_restore
  
  then press OK
  
ii. Create linked server
  Sql server --> Server Onjects --> Open properties of "MSDASQL"
  then check :
	Dynamic Parameter
	Nested Queries
	Level Zero only
	Allow inprocess
	Supports 'Like' operator
	
	press OK
	
  --> roght mouse clink on Linked serever --> Add new linked server --> 	then give
       Linked server = MySqlDb
	   Other data source:
			Provider = Microsoft OLE DB Providee for ODBC drivers
			Product Name = MySQL
			Data Source = MySqlDb
			Provider String = DRIVER=(MySQL ODBC 5.3 ANSI Driver);SERVER=localhost;PORT=3306;DATABASE=ghtorrent_restore; USER=root;PASSWORD=sala1234;OPTION=3;
			Location=  <== no need to give
			Catalog = ghtorrent_restore

     press OK

Note: 

syntax :
DRIVER=(MySQL ODBC 5.2 ANSI Driver);SERVER=localhost;PORT=3306;DATABASE=mantisbt; USER=user;PASSWORD=password;OPTION=3;

meaning of OPTION=3 in the MySQL connection string:
Option=1 FLAG_FIELD_LENGHT: Do not Optimize Column Width
Option=2 FLAG_FOUND_ROWS: Return matching rows
Option=3 option 1 and 2 together
	 
iii. then run the following user grant cmd in sql server
-- Map local user login to remote MySql user to grant the access 
exec master.dbo.sp_addlinkedsrvlogin
@rmtsrvname=N'MySqlDb',
@locallogin=NULL,
@rmtuser=N'root',
@rmtpassword=N'mypas' <== mysqlpas (its my common pass with 8 length)
 
iv.now test the connection


v. run the sql script in sql server

select * from OPENQUERY(MySqlDb, 'select top 1000 * from ghtorrent_restore.issues')


	 
(B). via Script in Sql server
==============================
i. same as above

ii. run following cmds in sql server

-- to create linked server
Exec master.dbo.sp_addlinkedserver
@server=N'MySqlDb',
@srvproduct=N'MySql',
@provider=N'MSDASQL',
@datasrc=N'MySqlDb',
@provstr=N'DRIVER=(MySQL ODBC 5.3 ANSI Driver);SERVER=localhost;PORT=3306;DATABASE=ghtorrent_restore; USER=root;PASSWORD=mycommonpassword;OPTION=3;';

-- Map local user login to remote MySql user to grant the access 
exec master.dbo.sp_addlinkedsrvlogin
@rmtsrvname=N'MySqlDb',
@locallogin=NULL,
@rmtuser=N'root',
@rmtpassword=N'mypas' <== mysqlpas (its my common pass with 8 length)

ii.now test the connection

iv. run the sql script in sql server

select * from OPENQUERY(MySqlDb, 'select top 1000 * from ghtorrent_restore.issues')

Linked Server Properties:
========================
Options
	Dynamic parameter
		Indicates that the provider allows '?' parameter marker syntax for parameterized queries. Set this option only if the provider supports the ICommandWithParameters interface and supports a '?' as the parameter marker. Setting this option allows SQL Server to execute parameterized queries against the provider. The ability to execute parameterized queries against the provider can result in better performance for certain queries.
		
Nested queries
	Indicates that the provider allows nestedSELECT statements in the FROM clause. Setting this option allows SQL Server to delegate certain queries to the provider that require nesting SELECT statements in the FROM clause.
	
Level zero only
	Only level 0 OLE DB interfaces are invoked against the provider.

Allow inprocess
	SQL Server allows the provider to be instantiated as an in-process server. When this option is not set, the default behavior is to instantiate the provider outside the SQL Server process. Instantiating the provider outside the SQL Server process protects the SQL Server process from errors in the provider. When the provider is instantiated outside the SQL Server process, updates or inserts referencing long columns (text, ntext, or image) are not allowed.
	
Non transacted updates
	SQL Server allows updates, even if ITransactionLocal is not available. If this option is enabled, updates against the provider are not recoverable, because the provider does not support transactions.
	
Index as access path
	SQL Server attempts to use indexes of the provider to fetch data. By default, indexes are used only for metadata and are never opened.
	
Disallow ad hoc access
	SQL Server does not allow ad hoc access through the OPENROWSET and OPENDATASOURCE functions against the OLE DB provider. When this option is not set, SQL Server also does not allow ad hoc access.
	
Supports 'Like' operator
	Indicates that the provider supports queries using the LIKE keyword.


References:
============

best ==>  http://www.ideaexcursion.com/2009/02/25/howto-setup-sql-server-linked-server-to-mysql/
http://www.sqlservercentral.com/Forums/Topic340912-146-1.aspx

http://www.thebuttonfactory.nl/?p=2204
