

================
Criteria Queries
================

NHibernate features an intuitive, extensible criteria query API.

Creating an ``ICriteria`` instance
##################################

The interface ``NHibernate.ICriteria`` represents a query against
a particular persistent class. The ``ISession`` is a factory for
``ICriteria`` instances.

.. code-block:: csharp

  ICriteria crit = sess.CreateCriteria(typeof(Cat));
  crit.SetMaxResults(50);
  List cats = crit.List();

Narrowing the result set
########################

An individual query criterion is an instance of the interface
``NHibernate.Expression.ICriterion``. The class
``NHibernate.Expression.Expression`` defines
factory methods for obtaining certain built-in
``ICriterion`` types.

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .Add( Expression.Like("Name", "Fritz%") )
      .Add( Expression.Between("Weight", minWeight, maxWeight) )
      .List();

Expressions may be grouped logically.

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .Add( Expression.Like("Name", "Fritz%") )
      .Add( Expression.Or(
          Expression.Eq( "Age", 0 ),
          Expression.IsNull("Age")
      ) )
      .List();

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .Add( Expression.In( "Name", new String[] { "Fritz", "Izi", "Pk" } ) )
      .Add( Expression.Disjunction()
          .Add( Expression.IsNull("Age") )
      	.Add( Expression.Eq("Age", 0 ) )
      	.Add( Expression.Eq("Age", 1 ) )
      	.Add( Expression.Eq("Age", 2 ) )
      ) )
      .List();

There are quite a range of built-in criterion types (``Expression``
subclasses), but one that is especially useful lets you specify SQL directly.

.. code-block:: csharp

  // Create a string parameter for the SqlString below
          IList cats = sess.CreateCriteria(typeof(Cat))
              .Add( Expression.Sql("lower({alias}.Name) like lower(?)", "Fritz%", NHibernateUtil.String )
              .List();

The ``{alias}`` placeholder with be replaced by the row alias
of the queried entity.

Ordering the results
####################

You may order the results using ``NHibernate.Expression.Order``.

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .Add( Expression.Like("Name", "F%")
      .AddOrder( Order.Asc("Name") )
      .AddOrder( Order.Desc("Age") )
      .SetMaxResults(50)
      .List();

Associations
############

You may easily specify constraints upon related entities by navigating
associations using ``CreateCriteria()``.

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .Add( Expression.Like("Name", "F%")
      .CreateCriteria("Kittens")
          .Add( Expression.Like("Name", "F%") )
      .List();

note that the second ``CreateCriteria()`` returns a new
instance of ``ICriteria``, which refers to the elements of
the ``Kittens`` collection.

The following, alternate form is useful in certain circumstances.

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .CreateAlias("Kittens", "kt")
      .CreateAlias("Mate", "mt")
      .Add( Expression.EqProperty("kt.Name", "mt.Name") )
      .List();

(``CreateAlias()`` does not create a new instance of
``ICriteria``.)

Note that the kittens collections held by the ``Cat`` instances
returned by the previous two queries are *not* pre-filtered
by the criteria! If you wish to retrieve just the kittens that match the
criteria, you must use ``SetResultTransformer(Transformers.AliasToEntityMap)``.

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .CreateCriteria("Kittens", "kt")
          .Add( Expression.Eq("Name", "F%") )
      .SetResultTransformer(Transformers.AliasToEntityMap)
      .List();
  foreach ( IDictionary map in cats )
  {
      Cat cat = (Cat) map[CriteriaUtil.RootAlias];
      Cat kitten = (Cat) map["kt"];
  }

Dynamic association fetching
############################

You may specify association fetching semantics at runtime using
``SetFetchMode()``.

.. code-block:: csharp

  IList cats = sess.CreateCriteria(typeof(Cat))
      .Add( Expression.Like("Name", "Fritz%") )
      .SetFetchMode("Mate", FetchMode.Eager)
      .SetFetchMode("Kittens", FetchMode.Eager)
      .List();

This query will fetch both ``Mate`` and ``Kittens``
by outer join. See :ref:`performance-fetching` for more information.

Example queries
###############

The class ``NHibernate.Expression.Example`` allows
you to construct a query criterion from a given instance.

.. code-block:: csharp

  Cat cat = new Cat();
  cat.Sex = 'F';
  cat.Color = Color.Black;
  List results = session.CreateCriteria(typeof(Cat))
      .Add( Example.Create(cat) )
      .List();

Version properties, identifiers and associations are ignored. By default,
null-valued properties and properties which return an empty string from
the call to ``ToString()`` are excluded.

You can adjust how the ``Example`` is applied.

.. code-block:: csharp

  Example example = Example.Create(cat)
      .ExcludeZeroes()           //exclude null- or zero-valued properties
      .ExcludeProperty("Color")  //exclude the property named "color"
      .IgnoreCase()              //perform case insensitive string comparisons
      .EnableLike();             //use like for string comparisons
  IList results = session.CreateCriteria(typeof(Cat))
      .Add(example)
      .List();

You can even use examples to place criteria upon associated objects.

.. code-block:: csharp

  IList results = session.CreateCriteria(typeof(Cat))
      .Add( Example.Create(cat) )
      .CreateCriteria("Mate")
          .Add( Example.Create( cat.Mate ) )
      .List();

Projections, aggregation and grouping
#####################################

The class ``NHibernate.Expression.Projections`` is a
factory for ``IProjection`` instances. We apply a
projection to a query by calling ``SetProjection()``.

.. code-block:: csharp

  IList results = session.CreateCriteria(typeof(Cat))
      .SetProjection( Projections.RowCount() )
      .Add( Expression.Eq("Color", Color.BLACK) )
      .List();

.. code-block:: csharp

  List results = session.CreateCriteria(typeof(Cat))
      .SetProjection( Projections.ProjectionList()
          .Add( Projections.RowCount() )
          .Add( Projections.Avg("Weight") )
          .Add( Projections.Max("Weight") )
          .Add( Projections.GroupProperty("Color") )
      )
      .List();

There is no explicit "group by" necessary in a criteria query. Certain
projection types are defined to be *grouping projections*,
which also appear in the SQL ``group by`` clause.

An alias may optionally be assigned to a projection, so that the projected value
may be referred to in restrictions or orderings. Here are two different ways to
do this:

.. code-block:: csharp

  IList results = session.CreateCriteria(typeof(Cat))
      .SetProjection( Projections.Alias( Projections.GroupProperty("Color"), "colr" ) )
      .AddOrder( Order.Asc("colr") )
      .List();

.. code-block:: csharp

  IList results = session.CreateCriteria(typeof(Cat))
      .SetProjection( Projections.GroupProperty("Color").As("colr") )
      .AddOrder( Order.Asc("colr") )
      .List();

The ``Alias()`` and ``As()`` methods simply wrap a
projection instance in another, aliased, instance of ``IProjection``.
As a shortcut, you can assign an alias when you add the projection to a
projection list:

.. code-block:: csharp

  IList results = session.CreateCriteria(typeof(Cat))
      .SetProjection( Projections.ProjectionList()
          .Add( Projections.RowCount(), "catCountByColor" )
          .Add( Projections.Avg("Weight"), "avgWeight" )
          .Add( Projections.Max("Weight"), "maxWeight" )
          .Add( Projections.GroupProperty("Color"), "color" )
      )
      .AddOrder( Order.Desc("catCountByColor") )
      .AddOrder( Order.Desc("avgWeight") )
      .List();

.. code-block:: csharp

  IList results = session.CreateCriteria(typeof(DomesticCat), "cat")
      .CreateAlias("kittens", "kit")
      .SetProjection( Projections.ProjectionList()
          .Add( Projections.Property("cat.Name"), "catName" )
          .Add( Projections.Property("kit.Name"), "kitName" )
      )
      .AddOrder( Order.Asc("catName") )
      .AddOrder( Order.Asc("kitName") )
      .List();

Detached queries and subqueries
###############################

The ``DetachedCriteria`` class lets you create a query outside the scope
of a session, and then later execute it using some arbitrary ``ISession``.

.. code-block:: csharp

  DetachedCriteria query = DetachedCriteria.For(typeof(Cat))
      .Add( Expression.Eq("sex", 'F') );

  ISession session = ....;
  ITransaction txn = session.BeginTransaction();
  IList results = query.GetExecutableCriteria(session).SetMaxResults(100).List();
  txn.Commit();
  session.Close();

A ``DetachedCriteria`` may also be used to express a subquery. ICriterion
instances involving subqueries may be obtained via ``Subqueries``
.

.. code-block:: csharp

  DetachedCriteria avgWeight = DetachedCriteria.For(typeof(Cat))
      .SetProjection( Projections.Avg("Weight") );
  session.CreateCriteria(typeof(Cat))
      .Add( Subqueries.Gt("Weight", avgWeight) )
      .List();

.. code-block:: csharp

  DetachedCriteria weights = DetachedCriteria.For(typeof(Cat))
      .SetProjection( Projections.Property("Weight") );
  session.CreateCriteria(typeof(Cat))
      .add( Subqueries.GeAll("Weight", weights) )
      .list();

Even correlated subqueries are possible:

.. code-block:: csharp

  DetachedCriteria avgWeightForSex = DetachedCriteria.For(typeof(Cat), "cat2")
      .SetProjection( Projections.Avg("Weight") )
      .Add( Expression.EqProperty("cat2.Sex", "cat.Sex") );
  session.CreateCriteria(typeof(Cat), "cat")
      .Add( Subqueries.Gt("weight", avgWeightForSex) )
      .List();

