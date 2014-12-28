

=============================
ISessionFactory Configuration
=============================

Because NHibernate is designed to operate in many different environments, there
are a large number of configuration parameters. Fortunately, most have sensible
default values and NHibernate is distributed with an example
``App.config`` file (found in ``src\\NHibernate.Test``)
that shows the various options. You usually only have to put that file in your
project and customize it.

Programmatic Configuration
##########################

An instance of ``NHibernate.Cfg.Configuration``
represents an entire set of mappings of an application's .NET types to a
SQL database. The ``Configuration`` is used to build an
(immutable) ``ISessionFactory``. The mappings are compiled
from various XML mapping files.

You may obtain a ``Configuration`` instance by
instantiating it directly. Heres an example of setting up a datastore from
mappings defined in two XML configuration files:

.. code-block:: csharp

  Configuration cfg = new Configuration()
      .AddFile("Item.hbm.xml")
      .AddFile("Bid.hbm.xml");

An alternative (sometimes better) way is to let NHibernate load a mapping file
from an embedded resource:

.. code-block:: csharp

  Configuration cfg = new Configuration()
      .AddClass(typeof(NHibernate.Auction.Item))
      .AddClass(typeof(NHibernate.Auction.Bid));

Then NHibernate will look for mapping files named
``NHibernate.Auction.Item.hbm.xml`` and
``NHibernate.Auction.Bid.hbm.xml`` embedded as resources in the
assembly that the types are contained in. This approach eliminates any hardcoded
filenames.

Another alternative (probably the best) way is to let NHibernate load all of
the mapping files contained in an Assembly:

.. code-block:: csharp

  Configuration cfg = new Configuration()
      .AddAssembly( "NHibernate.Auction" );

Then NHibernate will look through the assembly for any resources that
end with ``.hbm.xml``.  This approach eliminates
any hardcoded filenames and ensures the mapping files in the assembly
get added.

If a tool like Visual Studio .NET or NAnt is used to build the assembly,
then make sure that the ``.hbm.xml`` files are compiled
into the assembly as ``Embedded Resources``.

A ``Configuration`` also specifies various optional properties:

.. code-block:: csharp

  IDictionary props = new Hashtable();
  ...
  Configuration cfg = new Configuration()
      .AddClass(typeof(NHibernate.Auction.Item))
      .AddClass(typeof(NHibernate.Auction.Bind))
      .SetProperties(props);

A ``Configuration`` is intended as a configuration-time object, to be
discarded once an ``ISessionFactory`` is built.

Obtaining an ISessionFactory
############################

When all mappings have been parsed by the ``Configuration``, the application
must obtain a factory for ``ISession`` instances. This factory is intended
to be shared by all application threads:

.. code-block:: csharp

  ISessionFactory sessions = cfg.BuildSessionFactory();

However, NHibernate does allow your application to instantiate more than one
``ISessionFactory``. This is useful if you are using more than one database.

User provided ADO.NET connection
################################

An ``ISessionFactory`` may open an ``ISession`` on
a user-provided ADO.NET connection. This design choice frees the application to
obtain ADO.NET connections wherever it pleases:

.. code-block:: csharp

  IDbConnection conn = myApp.GetOpenConnection();
  ISession session = sessions.OpenSession(conn);

  // do some data access work

The application must be careful not to open two concurrent
``ISession`` on the same ADO.NET connection!

NHibernate provided ADO.NET connection
######################################

Alternatively, you can have the ``ISessionFactory``
open connections for you. The ``ISessionFactory``
must be provided with ADO.NET connection properties in one of the
following ways:

* Pass an instance of ``IDictionary`` mapping
  property names to property values to
  ``Configuration.SetProperties()``.

* Add the properties to a configuration section in the application
  configuration file. The section should be named ``nhibernate``
  and its handler set to ``System.Configuration.NameValueSectionHandler``.

* Include ``<property>`` elements in a configuration
  section in the application configuration file. The section should be named
  ``hibernate-configuration`` and its handler set to
  ``NHibernate.Cfg.ConfigurationSectionHandler``.
  The XML namespace of the section should be set to
  ``urn:nhibernate-configuration-2.2``.

* Include ``<property>`` elements in
  ``hibernate.cfg.xml`` (discussed later).

If you take this approach, opening an ``ISession`` is as simple as:

.. code-block:: csharp

  ISession session = sessions.OpenSession(); // open a new Session
  // do some data access work, an ADO.NET connection will be used on demand

All NHibernate property names and semantics are defined on the class
``NHibernate.Cfg.Environment``. We will now describe the most
important settings for ADO.NET connection configuration.

NHibernate will obtain (and pool) connections using an ADO.NET data provider
if you set the following properties:

NHibernate ADO.NET Properties

===================================== ================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================
Property name                         Purpose
===================================== ================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================
``connection.provider_class``         The type of a custom ``IConnectionProvider``.    *eg.*  ``full.classname.of.ConnectionProvider`` if the Provider  is built into NHibernate, or ``full.classname.of.ConnectionProvider,  assembly`` if using an implementation of ``IConnectionProvider`` not included in NHibernate.
``connection.driver_class``           The type of a custom ``IDriver``, if using ``DriverConnectionProvider``.    ``full.classname.of.Driver`` if the Driver  is built into NHibernate, or ``full.classname.of.Driver, assembly``  if using an implementation of IDriver not included in NHibernate.     This is usually not needed, most of the time the ``dialect`` will take care of setting the ``IDriver`` using a sensible default.  See the API documentation of the specific dialect for the defaults.
``connection.connection_string``      Connection string to use to obtain the connection.
``connection.connection_string_name`` The name of the connection string (defined in ``<connectionStrings>`` configuration file element) to use to obtain the connection.
``connection.isolation``              Set the ADO.NET transaction isolation level. Check ``System.Data.IsolationLevel`` for meaningful values  and the database's documentation to ensure that level is supported.    *eg.*  ``Chaos, ReadCommitted, ReadUncommitted, RepeatableRead, Serializable, Unspecified``
``connection.release_mode``           Specify when NHibernate should release ADO.NET connections. See :ref:`transactions-connection-release`.    *eg.*  ``auto`` (default) | ``on_close`` | ``after_transaction``     Note that this setting only affects ``ISession`` returned from ``ISessionFactory.OpenSession``.  For ``ISession`` obtained through ``ISessionFactory.GetCurrentSession``, the ``ICurrentSessionContext`` implementation configured for use controls the connection release mode for those ``ISession``. See :ref:`architecture-current-session`.
``command_timeout``                   Specify the default timeout of ``IDbCommands`` generated by NHibernate.
``adonet.batch_size``                 Specify the batch size to use when batching update statements. Setting this to 0 (the default) disables the functionality. See :ref:`performance-batch-updates`.
===================================== ================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================

This is an example of how to specify the database connection properties inside a
``web.config``:

.. code-block:: xml

  <?xml version="1.0" encoding="utf-8" ?>
  <configuration>
  	<configSections>
  		<section name="hibernate-configuration"
  				 type="NHibernate.Cfg.ConfigurationSectionHandler, NHibernate" />
  	</configSections>

  	<hibernate-configuration xmlns="urn:nhibernate-configuration-2.2">
  		<session-factory>
  			<property name="dialect">NHibernate.Dialect.MsSql2005Dialect</property>
  			<property name="connection.connection_string">
  				Server=(local);initial catalog=theDb;Integrated Security=SSPI
  			</property>
  			<property name="connection.isolation">ReadCommitted</property>

  			<property name="proxyfactory.factory_class">
  				NHibernate.ByteCode.LinFu.ProxyFactoryFactory, NHibernate.ByteCode.LinFu
  			</property>
  		</session-factory>
  	</hibernate-configuration>

      <!-- other app specific config follows -->
  </configuration>

NHibernate relies on the ADO.NET data provider implementation of connection pooling.

You may define your own plugin strategy for obtaining ADO.NET connections by implementing the
interface ``NHibernate.Connection.IConnectionProvider``. You may select
a custom implementation by setting ``connection.provider_class``.

Optional configuration properties
#################################

There are a number of other properties that control the behaviour of NHibernate
at runtime. All are optional and have reasonable default values.

System-level properties can only be set manually by setting static properties of
``NHibernate.Cfg.Environment`` class or be defined in the
``<nhibernate>`` section of the application
configuration file. These properties cannot be set using
``Configuration.SetProperties`` or be defined in the
``<hibernate-configuration>`` section of the application configuration
file.

NHibernate Configuration Properties

============================= ============================================================================================================================================================================================================================================================================================================================================================================================================================================================================
Property name                 Purpose
============================= ============================================================================================================================================================================================================================================================================================================================================================================================================================================================================
``dialect``                   The classname of a NHibernate ``Dialect`` - enables certain platform dependent features.    *eg.*  ``full.classname.of.Dialect, assembly``
``default_schema``            Qualify unqualified tablenames with the given schema/tablespace in generated SQL.    *eg.*  ``SCHEMA_NAME``
``max_fetch_depth``           Set a maximum "depth" for the outer join fetch tree for single-ended associations (one-to-one, many-to-one). A ``0`` disables default outer join fetching.    *eg.*  recommended values between ``0`` and ``3``
``use_reflection_optimizer``  Enables use of a runtime-generated class to set or get properties of an entity or component instead of using runtime reflection (System-level property). The use of the reflection optimizer inflicts a certain startup cost on the application but should lead to better performance in the long run. You can not set this property in ``hibernate.cfg.xml`` or ``<hibernate-configuration>`` section of the application configuration file.    *eg.*  ``true`` | ``false``
``bytecode.provider``         Specifies the bytecode provider to use to optimize the use of reflection in NHibernate. Use ``null`` to disable the optimization completely, ``lcg`` to use lightweight code generation.     *eg.* ``null`` | ``lcg``
``cache.provider_class``      The classname of a custom ``ICacheProvider``.    *eg.*  ``classname.of.CacheProvider, assembly``
``cache.use_minimal_puts``    Optimize second-level cache operation to minimize writes, at the cost of more frequent reads (useful for clustered caches).    *eg.*  ``true`` | ``false``
``cache.use_query_cache``     Enable the query cache, individual queries still have to be set cacheable.    *eg.*  ``true`` | ``false``
``cache.query_cache_factory`` The classname of a custom ``IQueryCacheFactory`` interface, defaults to the built-in ``StandardQueryCacheFactory``.    *eg.* ``classname.of.QueryCacheFactory, assembly``
``cache.region_prefix``       A prefix to use for second-level cache region names.    *eg.*  ``prefix``
``query.substitutions``       Mapping from tokens in NHibernate queries to SQL tokens (tokens might be function or literal names, for example).    *eg.*  ``hqlLiteral=SQL_LITERAL, hqlFunction=SQLFUNC``
``show_sql``                  Write all SQL statements to console.    *eg.*  ``true`` | ``false``
``hbm2ddl.auto``              Automatically export schema DDL to the database when the ``ISessionFactory`` is created. With ``create-drop``, the database schema will be dropped when the ``ISessionFactory`` is closed explicitly.    *eg.*  ``create`` | ``create-drop``
``hbm2ddl.keywords``          Automatically import ``reserved/keywords`` from the database when the ``ISessionFactory`` is created.    *none :* disable any operation regarding RDBMS KeyWords     *keywords :* imports all RDBMS KeyWords where the ``Dialect`` can provide the implementation of ``IDataBaseSchema``.     *auto-quote :* imports all RDBMS KeyWords and auto-quote all table-names/column-names .     *eg.* ``none`` | ``keywords`` | ``auto-quote``
``use_proxy_validator``       Enables or disables validation of interfaces or classes specified as proxies. Enabled by default.    *eg.*  ``true`` | ``false``
``transaction.factory_class`` The classname of a custom ``ITransactionFactory`` implementation, defaults to the built-in ``AdoNetWithDistributedTransactionFactory``.    *eg.*  ``classname.of.TransactionFactory, assembly``
============================= ============================================================================================================================================================================================================================================================================================================================================================================================================================================================================

SQL Dialects
============

You should always set the ``dialect`` property to the correct
``NHibernate.Dialect.Dialect`` subclass for your database. This is not
strictly essential unless you wish to use ``native`` or
``sequence`` primary key generation or pessimistic locking (with, eg.
``ISession.Lock()`` or ``IQuery.SetLockMode()``).
However, if you specify a dialect, NHibernate will use sensible defaults for some of the
other properties listed above, saving you the effort of specifying them manually.

NHibernate SQL Dialects (

==================================== ================================================= ================================================================================================================
RDBMS                                Dialect                                           Remarks
==================================== ================================================= ================================================================================================================
DB2                                  ``NHibernate.Dialect.DB2Dialect``
DB2 for iSeries (OS/400)             ``NHibernate.Dialect.DB2400Dialect``
Ingres                               ``NHibernate.Dialect.IngresDialect``
PostgreSQL                           ``NHibernate.Dialect.PostgreSQLDialect``
PostgreSQL 8.1                       ``NHibernate.Dialect.PostgreSQL81Dialect``        This dialect supports ``FOR UPDATE NOWAIT`` available in PostgreSQL 8.1.
PostgreSQL 8.2                       ``NHibernate.Dialect.PostgreSQL82Dialect``        This dialect supports ``IF EXISTS`` keyword in ``DROP TABLE`` and ``DROP SEQUENCE`` available in PostgreSQL 8.2.
MySQL 3 or 4                         ``NHibernate.Dialect.MySQLDialect``
MySQL 5                              ``NHibernate.Dialect.MySQL5Dialect``
Oracle                               ``NHibernate.Dialect.Oracle8iDialect``
Oracle 9                             ``NHibernate.Dialect.Oracle9iDialect``
Oracle 10g                           ``NHibernate.Dialect.Oracle10gDialect``
Sybase Adaptive Server Enterprise 15 ``NHibernate.Dialect.SybaseASE15Dialect``
Sybase Adaptive Server Anywhere 9    ``NHibernate.Dialect.SybaseASA9Dialect``
Sybase SQL Anywhere 10               ``NHibernate.Dialect.SybaseSQLAnywhere10Dialect``
Sybase SQL Anywhere 11               ``NHibernate.Dialect.SybaseSQLAnywhere11Dialect``
Microsoft SQL Server 7               ``NHibernate.Dialect.MsSql7Dialect``
Microsoft SQL Server 2000            ``NHibernate.Dialect.MsSql2000Dialect``
Microsoft SQL Server 2005            ``NHibernate.Dialect.MsSql2005Dialect``
Microsoft SQL Server 2008            ``NHibernate.Dialect.MsSql2008Dialect``
Microsoft SQL Server Compact Edition ``NHibernate.Dialect.MsSqlCeDialect``
Firebird                             ``NHibernate.Dialect.FirebirdDialect``            Set ``driver_class`` to ``NHibernate.Driver.FirebirdClientDriver`` for Firebird ADO.NET provider 2.0.
SQLite                               ``NHibernate.Dialect.SQLiteDialect``              Set ``driver_class`` to ``NHibernate.Driver.SQLite20Driver`` for System.Data.SQLite provider for .NET 2.0.
==================================== ================================================= ================================================================================================================

Additional dialects may be available in the NHibernate.Dialect namespace.

Outer Join Fetching
===================

If your database supports ANSI or Oracle style outer joins, *outer join
fetching* might increase performance by limiting the number of round
trips to and from the database (at the cost of possibly more work performed by
the database itself). Outer join fetching allows a graph of objects connected
by many-to-one, one-to-many or one-to-one associations to be retrieved in a single
SQL ``SELECT``.

By default, the fetched graph when loading an objects ends at leaf objects,
collections, objects with proxies, or where circularities occur.

For a *particular  association*, fetching may be configured
(and the default behaviour overridden) by setting the ``fetch``
attribute in the XML mapping.

Outer join fetching may be disabled *globally* by setting
the property ``max_fetch_depth`` to ``0``.
A setting of ``1`` or higher enables outer join fetching for
one-to-one and many-to-one associations which have been mapped with
``fetch="join"``.

See :ref:`performance-fetching` for more information.

In NHibernate 1.0, ``outer-join`` attribute could be used to achieve
a similar effect. This attribute is now deprecated in favor of ``fetch``.

Custom ``ICacheProvider``
=========================

You may integrate a process-level (or clustered) second-level cache system by
implementing the interface ``NHibernate.Cache.ICacheProvider``.
You may select the custom implementation by setting
``cache.provider_class``. See the
:ref:`performance-cache` for more details.

Query Language Substitution
===========================

You may define new NHibernate query tokens using ``query.substitutions``.
For example:

.. code-block:: csharp

  query.substitutions true=1, false=0

would cause the tokens ``true`` and ``false`` to be translated to
integer literals in the generated SQL.

.. code-block:: csharp

  query.substitutions toLowercase=LOWER

would allow you to rename the SQL ``LOWER`` function.

Logging
#######

NHibernate logs various events using Apache log4net.

You may download log4net from ``http://logging.apache.org/log4net/``.
To use log4net you will need a ``log4net`` configuration section in
the application configuration file.  An example of the configuration section is
distributed with NHibernate in the ``src/NHibernate.Test`` project.

We strongly recommend that you familiarize yourself with NHibernate's log
messages. A lot of work has been put into making the NHibernate log as
detailed as possible, without making it unreadable. It is an essential
troubleshooting device. Also don't forget to enable SQL logging as
described above (``show_sql``), it is your first
step when looking for performance problems.

Implementing an ``INamingStrategy``
###################################

The interface ``NHibernate.Cfg.INamingStrategy`` allows you
to specify a "naming standard" for database objects and schema elements.

You may provide rules for automatically generating database identifiers from
.NET identifiers or for processing "logical" column and table names given in
the mapping file into  "physical" table and column names. This feature helps
reduce the verbosity of the mapping document, eliminating repetitive noise
(``TBL_`` prefixes, for example). The default strategy used by
NHibernate is quite minimal.

You may specify a different strategy by calling
``Configuration.SetNamingStrategy()`` before adding mappings:

.. code-block:: csharp

  ISessionFactory sf = new Configuration()
      .SetNamingStrategy(ImprovedNamingStrategy.Instance)
      .AddFile("Item.hbm.xml")
      .AddFile("Bid.hbm.xml")
      .BuildSessionFactory();

``NHibernate.Cfg.ImprovedNamingStrategy`` is a built-in
strategy that might be a useful starting point for some applications.

XML Configuration File
######################

An alternative approach is to specify a full configuration in a file named
``hibernate.cfg.xml``. This file can be used as a replacement
for the ``<nhibernate;>`` or ``<hibernate-configuration>``
sections of the application configuration file.

The XML configuration file is by default expected to be in your application directory.
Here is an example:

.. code-block:: xml

  <?xml version='1.0' encoding='utf-8'?>
  <hibernate-configuration xmlns="urn:nhibernate-configuration-2.2">

      <!-- an ISessionFactory instance -->
      <session-factory>

          <!-- properties -->
          <property name="connection.provider">NHibernate.Connection.DriverConnectionProvider</property>
          <property name="connection.driver_class">NHibernate.Driver.SqlClientDriver</property>
          <property name="connection.connection_string">Server=localhost;initial catalog=nhibernate;User Id=;Password=</property>
          <property name="show_sql">false</property>
          <property name="dialect">NHibernate.Dialect.MsSql2000Dialect</property>

          <!-- mapping files -->
          <mapping resource="NHibernate.Auction.Item.hbm.xml" assembly="NHibernate.Auction" />
          <mapping resource="NHibernate.Auction.Bid.hbm.xml" assembly="NHibernate.Auction" />

      </session-factory>

  </hibernate-configuration>

Configuring NHibernate is then as simple as

.. code-block:: csharp

  ISessionFactory sf = new Configuration().Configure().BuildSessionFactory();

You can pick a different XML configuration file using

.. code-block:: csharp

  ISessionFactory sf = new Configuration()
      .Configure("/path/to/config.cfg.xml")
      .BuildSessionFactory();

