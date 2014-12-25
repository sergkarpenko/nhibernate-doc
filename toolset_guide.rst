.. _toolset_guide:

=============
Toolset Guide
=============

Roundtrip engineering with NHibernate is possible using a set of commandline tools
maintained as part of the NHibernate project, along with NHibernate support built into
various code generation tools (MyGeneration, CodeSmith, ObjectMapper, AndroMDA).

The NHibernate main package comes bundled with the most important tool (it can even
be used from "inside" NHibernate on-the-fly):

- DDL schema generation from a mapping file
  (aka SchemaExport, hbm2ddl)

Other tools directly provided by the NHibernate project are delivered with a separate
package, *NHibernateContrib*. This package includes tools for
the following tasks:

- C# source generation from a mapping file (aka hbm2net)

- mapping file generation from .NET classes marked with attributes
  (NHibernate.Mapping.Attributes, or NHMA for short)

Third party tools with NHibernate support are:

- CodeSmith, MyGeneration, and ObjectMapper (mapping file generation from an existing
  database schema)

- AndroMDA (MDA (Model-Driven Architecture) approach generating code for
  persistent classes from UML diagrams and their XML/XMI representation)

These 3rd party tools are not documented in this reference. Please refer to the NHibernate
website for up-to-date information.

Schema Generation
#################

The generated schema includes referential integrity constraints (primary and foreign keys) for entity
and collection tables. Tables and sequences are also created for mapped identifier generators.

You *must* specify a SQL Dialect via the
hibernate.dialect property when using this tool.

Customizing the schema
======================

Many NHibernate mapping elements define an optional attribute named length. You may set
the length of a column with this attribute. (Or, for numeric/decimal data types, the precision.)

Some tags also accept a not-null attribute (for generating a NOT NULL
constraint on table columns) and a unique attribute (for generating UNIQUE
constraint on table columns).

Some tags accept an index attribute for specifying the
name of an index for that column. A unique-key attribute
can be used to group columns in a single unit key constraint. Currently, the
specified value of the unique-key attribute is
*not* used to name the constraint, only to group the
columns in the mapping file.

Examples:

.. code-block:: xml

    <property name="Foo" type="String" length="64" not-null="true"/>
    <many-to-one name="Bar" foreign-key="fk_foo_bar" not-null="true"/>
    <element column="serial_number" type="Int64" not-null="true" unique="true"/>

Alternatively, these elements also accept a child <column> element. This is
particularly useful for multi-column types:

.. code-block:: xml

    <property name="Foo" type="String">
    <column name="foo" length="64" not-null="true" sql-type="text"/>
    </property>
    <property name="Bar" type="My.CustomTypes.MultiColumnType, My.CustomTypes"/>
    <column name="fee" not-null="true" index="bar_idx"/>
    <column name="fi" not-null="true" index="bar_idx"/>
    <column name="fo" not-null="true" index="bar_idx"/>
    </property>

The sql-type attribute allows the user to override the default mapping
of NHibernate type to SQL datatype.

The check attribute allows you to specify a check constraint.

.. code-block:: xml

    <property name="Foo" type="Int32">
    <column name="foo" check="foo > 10"/>
    </property>
    <class name="Foo" table="foos" check="bar < 100.0">
    ...
    <property name="Bar" type="Single"/>
    </class>

Summary

=========== ================ ========================================================================================================================================================================================================================================
Attribute   Values           Interpretation
=========== ================ ========================================================================================================================================================================================================================================
length      number           column length/decimal precision
not-null    true|false       specfies that the column should be non-nullable
unique      true|false       specifies that the column should have a unique constraint
index       index_name       specifies the name of a (multi-column) index
unique-key  unique_key_name  specifies the name of a multi-column unique constraint
foreign-key foreign_key_name specifies the name of the foreign key constraint generated for an association, use it on <one-to-one>, <many-to-one>, <key>, and <many-to-many> mapping elements. Note that inverse="true" sides will not be considered by SchemaExport.
sql-type    column_type      overrides the default column type (attribute of <column> element only)
check       SQL expression   create an SQL check constraint on either column or table
=========== ================ ========================================================================================================================================================================================================================================

Running the tool
================

The SchemaExport tool writes a DDL script to standard out and/or
executes the DDL statements.

You may embed SchemaExport in your application:

.. code-block:: csharp

    Configuration cfg = ....;
    new SchemaExport(cfg).Create(false, true);

Properties
==========

Database properties may be specified

- as system properties with -D*<property>*

- in hibernate.properties

- in a named properties file with --properties

The needed properties are:

SchemaExport Connection Properties

================================= =================
Property Name                     Description
================================= =================
hibernate.connection.driver_class jdbc driver class
hibernate.connection.url          jdbc url
hibernate.connection.username     database user
hibernate.connection.password     user password
hibernate.dialect                 dialect
================================= =================

Using Ant
=========

You can call SchemaExport from your Ant build script:

.. code-block:: xml

    <target name="schemaexport">
        <taskdef name="schemaexport"
            classname="net.sf.hibernate.tool.hbm2ddl.SchemaExportTask"
            classpathref="class.path"/>
        <schemaexport
            properties="hibernate.properties"
            quiet="no"
            text="no"
            drop="no"
            delimiter=";"
            output="schema-export.sql"
        >
            <fileset dir="src">
                <include name="\**/\*.hbm.xml"/>
            </fileset>
        </schemaexport>
    </target>

If you don't specify properties or a config file,
the SchemaExportTask will try to use normal Ant project properties instead.
In other words, if you don't want or need an external configuration or properties file, you
may put hibernate.* configuration properties in your build.xml or
build.properties.

Code Generation
###############

The NHibernate code generator may be used to generate skeletal C# implementation classes
from a NHibernate mapping file. This tool is included in the NHibernate Contrib package
(a seperate download in http://sourceforge.net/projects/nhcontrib/).

hbm2net parses the mapping files and generates fully working C#
source files from these. Thus with hbm2net one could "just" provide the
.hbm files, and then don't worry about hand-writing/coding the C# files.

hbm2net *options
mapping_files*

Code Generator Command Line Options

===================== =====================================
Option                Description
===================== =====================================
-output:*output_dir*  root directory for generated code
-config:*config_file* optional file for configuring hbm2net
===================== =====================================

A more detailed guide of hbm2net is available in
http://nhforge.org/blogs/nhibernate/archive/2009/12/12/t4-hbm2net-alpha-2.aspx


