

=================
NHibernate.Caches
=================

What is NHibernate.Caches?
##########################

NHibernate.Caches namespace contains several second-level cache providers for NHibernate.
=========================================================================================

A cache is place where entities are kept after being loaded from the database; once cached, they can be
retrieved without going to the database. This means that they are faster to (re)load.

An NHibernate session has an internal (first-level) cache where it keeps its entities. There is no sharing
between these caches - a first-level cache belongs to a given session and is destroyed with it. NHibernate
provides a *second-level cache* system; it works at the session factory level.
A second-level cache is shared by all sessions created by the same session factory.

An important point is that the second-level cache *does not* cache instances of the object
type being cached; instead it caches the individual values of the properties of that object. This provides two
benefits. One, NHibernate doesn't have to worry that your client code will manipulate the objects in a way that
will disrupt the cache. Two, the relationships and associations do not become stale, and are easy to keep
up-to-date because they are simply identifiers. The cache is not a tree of objects but rather a map of arrays.

With the *session-per-request* model, a high number of sessions can concurrently access
the same entity without hitting the database each time; hence the performance gain.

Several cache providers have been contributed by NHibernate users:

NHibernate.Caches.Prevalence
    Uses Bamboo.Prevalence as the cache provider. Open the
    file :file:`Bamboo.Prevalence.license.txt` for more information about its license;
    you can also visit its `website <http://bbooprevalence.sourceforge.net/>`_.

NHibernate.Caches.SysCache
    Uses System.Web.Caching.Cache as the cache provider. This means that you can
    rely on ASP.NET caching feature to understand how it works. For more information, read (on the MSDN):
    `Caching Application Data <http://msdn.microsoft.com/library/en-us/cpguide/html/cpconcacheapis.asp>`_.

NHibernate.Caches.SysCache2
    Similar to NHibernate.Caches.SysCache, uses ASP.NET cache. This provider also supports
    SQL dependency-based expiration, meaning that it is possible to configure certain cache regions to automatically
    expire when the relevant data in the database changes.
    SysCache2 requires Microsoft SQL Server 2000 or higher and .NET Framework version 2.0 or higher.

NHibernate.Caches.MemCache
    Uses ``memcached``. See `memcached homepage <http://www.danga.com/memcached/>`_
    for more information.

NCache provider for NHibernate
    Uses ``NCache``,NCache is a commercial distributed caching system with a provider for NHibernate.
    The NCache Express version is free for use, see
    `NCache Express homepage <http://www.alachisoft.com/ncache/ncache_express.html>`_
    for more information.

How to use a cache?
###################

Here are the steps to follow to enable the second-level cache in NHibernate:

- Choose the cache provider you want to use and copy its assembly in your assemblies directory
  (:file:`NHibernate.Caches.Prevalence.dll` or
  :file:`NHibernate.Caches.SysCache.dll`).

- To tell NHibernate which cache provider to use, add in your NHibernate configuration file
  (can be :file:`YourAssembly.exe.config` or :file:`web.config` or a
  :file:`.cfg.xml` file, in the latter case the syntax will be different from what
  is shown below):
  .. code-block:: csharp
    <add key="hibernate.cache.provider_class" value="
  "``XXX``" is the assembly-qualified class name of a class
  implementing ICacheProvider, eg.
  "NHibernate.Caches.SysCache.SysCacheProvider,
  NHibernate.Caches.SysCache".
  The ``expiration`` value is the number of seconds you wish
  to cache each entry (here two minutes). This example applies to SysCache only.

- Add ``<cache usage="read-write|nonstrict-read-write|read-only"/>`` (just
  after ``<class>``) in the mapping of the entities you want to cache. It
  also works for collections (bag, list, map, set, ...).

Be careful
==========

Caches are never aware of changes made to the persistent store by another process (though they may be
configured to regularly expire cached data). As the caches are created at the session factory level,
they are destroyed with the SessionFactory instance; so you must keep them alive as long as you need
them.

Prevalence Cache Configuration
##############################

There is only one configurable parameter: ``prevalenceBase``. This is the directory on the
file system where the Prevalence engine will save data. It can be relative to the current directory or a
full path. If the directory doesn't exist, it will be created.

SysCache Configuration
######################

As SysCache relies on System.Web.Caching.Cache for the underlying implementation,
the configuration is based on the available options that make sense for NHibernate to utilize.

``expiration``
    Number of seconds to wait before expiring each item.

``priority``
    A numeric cost of expiring each item, where 1 is a low cost, 5 is the highest, and 3 is normal.
    Only values 1 through 5 are valid.

SysCache has a config file section handler to allow configuring different expirations and priorities for
different regions. Here's an example:

.. code-block:: xml

  <?xml version="1.0" encoding="utf-8" ?>
  <configuration>
  	<configSections>
  		<section name="syscache" type="NHibernate.Caches.SysCache.SysCacheSectionHandler,NHibernate.Caches.SysCache" />
  	</configSections>
  	<syscache>
  		<cache region="foo" expiration="500" priority="4" />
  		<cache region="bar" expiration="300" priority="3" />
  	</syscache>
  </configuration>

SysCache2 Configuration
#######################

SysCache2 can use SqlCacheDependencies to invalidate cache regions when data in an underlying SQL Server
table or query changes. Query dependencies are only available for SQL Server 2005. To use the cache
provider, the application must be setup and configured to support SQL notifications as described in the
MSDN documentation.

To configure cache regions with SqlCacheDependencies a ``yscache2`` config section must be
defined in the application's configuration file. See the sample below.

.. code-block:: csharp

  <configSections>
  	<section name="syscache2" type="NHibernate.Caches.SysCache2.SysCacheSection, NHibernate.Caches.SysCache2"/>
  </configSections>

Table-based Dependency
======================

A table-based dependency will monitor the data in a database table for changes. Table-based
dependencies are generally used for a SQL Server 2000 database but will work with SQL Server 2005 as
well. Before you can use SQL Server cache invalidation with table based dependencies, you need to
enable notifications for the database. This task is performed with the :command:`aspnet_regsql`
command. With table-based notifications, the application will poll the database for changes at a
predefined interval. A cache region will not be invalidated immediately when data in the table changes.
The cache will be invalidated the next time the application polls the database for changes.

To configure the data in a cache region to be invalidated when data in an underlying table is changed,
a cache region must be configured in the application's configuration file. See the sample below.

.. code-block:: csharp

  <syscache2>
  	<cacheRegion name="Product">
  		<dependencies>
  			<tables>
  				<add name="price"
  					databaseEntryName="Default"
  					tableName="VideoTitle" />
  			</tables>
  		</dependencies>
  	</cacheRegion>
  </syscache2>

Table-based Dependency Configuration Properties
===============================================

``name``
    Unique name for the dependency

``tableName``
    The name of the database table that the dependency is associated with. The table must be enabled
    for notification support with the ``AspNet_SqlCacheRegisterTableStoredProcedure``.

``databaseEntryName``
    The name of a database defined in the ``databases`` element for
    ``qlCacheDependency`` for caching (ASP.NET Settings Schema) element of the
    application's ``Web.config`` file.

Command-Based Dependencies
==========================

A command-based dependency will use a SQL command to identify records to monitor for data changes.
Command-based dependencies work only with SQL Server 2005.

Before you can use SQL Server cache invalidation with command-based dependencies, you need to enable
the Service Broker for query notifications. The application must also start the listener for receiving
change notifications from SQL Server. With command based notifications, SQL Server will notify the
application when the data of a record returned in the results of a SQL query has changed. Note that a
change will be indicated if the data in any of the columns of a record change, not just the columns
returned by a query. The query is a way to limit the number of records monitored for changes, not the
columns.  As soon as data in one of the records is modified, the data in the cache region will be
invalidated immediately.

To configure the data in a cache region to be invalidated based on a SQL command, a cache region must
be configured in the application's configuration file. See the samples below.

Stored Procedure
----------------

.. code-block:: csharp

  <cacheRegion name="Product" priority="High" >
  	<dependencies>
  		<commands>
  			<add name="price"
  				command="ActiveProductsStoredProcedure"
  				isStoredProcedure="true"/>
  		</commands>
  	</dependencies>
  </cacheRegion>

SELECT Statement
----------------

.. code-block:: csharp

  <cacheRegion name="Product" priority="High">
  	<dependencies>
  		<commands>
  			<add name="price"
  				command="Select VideoTitleId from dbo.VideoTitle where Active = 1"
  				connectionName="default"
  				connectionStringProviderType="NHibernate.Caches.SysCache2.ConfigConnectionStringProvider, NHibernate.Caches.SysCache2"/>
  		</commands>
  	</dependencies>
  </cacheRegion>

Command Configuration Properties
--------------------------------

``name``
    Unique name for the dependency

``command`` (required)
    SQL command that returns results which should be monitored for data changes

``isStoredProcedure`` (optional)
    Indicates if command is a stored procedure. The default is ``false``.

``connectionName`` (optional)
    The name of the connection in the applications configuration file to use for registering the
    cache dependency for change notifications. If no value is supplied for ``connectionName`` or ``connectionStringProviderType``, the connection properties from
    the NHibernate configruation will be used.

``connectionStringProviderType`` (optional)
    IConnectionStringProvider to use for retrieving the connection string to
    use for registering the cache dependency for change notifications.  If no value is supplied for
    ``connectionName``, the unnamed connection supplied by the provider will be
    used.

Aggregate Dependencies
======================

Multiple cache dependencies can be specified.  If any of the dependencies triggers a change
notification, the data in the cache region will be invalidated.  See the samples below.

Multiple commands
-----------------

.. code-block:: csharp

  <cacheRegion name="Product">
  	<dependencies>
  		<commands>
  			<add name="price"
  				command="ActiveProductsStoredProcedure"
  				isStoredProcedure="true"/>
  			<add name="quantity"
  				command="Select quantityAvailable from dbo.VideoAvailability"/>
  		</commands>
  	</dependencies>
  </cacheRegion>

Mixed
-----

.. code-block:: csharp

  <cacheRegion name="Product">
  	<dependencies>
  		<commands>
  			<add name="price"
  				command="ActiveProductsStoredProcedure"
  				isStoredProcedure="true"/>
  		</commands>
  		<tables>
  			<add name="quantity"
  				databaseEntryName="Default"
  				tableName=" VideoAvailability" />
  		</tables>
  	</dependencies>
  </cacheRegion>

Additional Settings
===================

In addition to data dependencies for the cache regions, time based expiration policies can be specified
for each item added to the cache.  Time based expiration policies will not invalidate the data
dependencies for the whole cache region, but serve as a way to remove items from the cache after they
have been in the cache for a specified amount of time.  See the samples below.

Relative Expiration
-------------------

.. code-block:: csharp

  <cacheRegion name="Product" relativeExpiration="300" priority="High" />

Time of Day Expiration
----------------------

.. code-block:: csharp

  <cacheRegion name="Product" timeOfDayExpiration="2:00:00" priority="High" />

Additional Configuration Properties
-----------------------------------

``relativeExpiration``
    Number of seconds that an individual item will exist in the cache before being removed.

``timeOfDayExpiration``
    24 hour based time of day that an item will exist in the cache until. 12am is specified as
    00:00:00; 10pm is specified as 22:00:00. Only valid if relativeExpiration is not specified.
    Time of Day Expiration is useful for scenarios where items should be expired from the cache
    after a daily process completes.

``priority``
    System.Web.Caching.CacheItemPriority that identifies the relative
    priority of items stored in the cache.

Patches
=======

There is a known issue where some SQL Server 2005 notifications might not be received when an
application subscribes to query notifications by using ADO.NET 2.0. To fix this problem install
`SQL hotfix for kb 913364 <http://support.microsoft.com/Default.aspx?kbid=913364>`_.

