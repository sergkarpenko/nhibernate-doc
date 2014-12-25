

=======
Preface
=======

Working with object-oriented software and a relational database can be cumbersome
and time consuming in today's enterprise environments. NHibernate is an object/relational
mapping tool for .NET environments. The term object/relational mapping (ORM) refers to
the technique of mapping a data representation from an object model to a relational
data model with a SQL-based schema.

NHibernate not only takes care of the mapping from .NET classes to
database tables (and from .NET data types to SQL data types), but also provides data
query and retrieval facilities and can significantly reduce development time otherwise
spent with manual data handling in SQL and ADO.NET.

NHibernate's goal is to relieve the developer from 95 percent of common data persistence
related programming tasks. NHibernate may not be the best solution for data-centric
applications that only use stored-procedures to implement the business logic in the
database, it is most useful with object-oriented domain models and business logic in
the .NET-based middle-tier. However, NHibernate can certainly help you to remove or
encapsulate vendor-specific SQL code and will help with the common task of result set
translation from a tabular representation to a graph of objects.

If you are new to NHibernate and Object/Relational Mapping or even .NET Framework,
please follow these steps:

* Read :ref:`quickstart` for a 30 minute tutorial,
  using Internet Information Services (IIS) web server.

* Read :ref:`architecture` to understand the environments where
  NHibernate can be used.

* Use this reference documentation as your primary source of information.
  Consider reading *Hibernate in Action*
  (java `http://www.manning.com/bauer/ <http://www.manning.com/bauer/>`_)
  or *NHibernate in Action*
  (`http://www.manning.com/kuate/ <http://www.manning.com/kuate/>`_)
  or *NHibernate 3.0 Cookbook*
  (`https://www.packtpub.com/nhibernate-3-0-cookbook/book <https://www.packtpub.com/nhibernate-3-0-cookbook/book>`_)
  or *NHibernate 2 Beginner's Guide*
  (`http://www.packtpub.com/nhibernate-2-x-beginners-guide/book <http://www.packtpub.com/nhibernate-2-x-beginners-guide/book>`_)if you need more help
  with application design or if you prefer a step-by-step tutorial. Also visit
  `http://nhibernate.sourceforge.net/NHibernateEg/ <http://nhibernate.sourceforge.net/NHibernateEg/>`_ for NHibernate
  tutorial with examples.

* FAQs are answered on the `NHibernate users group <http://groups.google.com/group/nhusers>`_.

* The Community Area on the `NHibernate website <http://www.nhforge.org/>`_ is a good source for
  design patterns and various integration solutions (ASP.NET, Windows	Forms).

If you have questions, use the
`NHibernate user forum <http://groups.google.com/group/nhusers>`_.
We also provide a `JIRA issue trackings system <http://jira.nhforge.org/>`_
for bug reports and feature requests.
If you are interested in the development of NHibernate, join the developer mailing list.
If you are interested in translating this documentation into your language, contact us
on the `developer mailing list <http://groups.google.com/group/nhibernate-development>`_.

