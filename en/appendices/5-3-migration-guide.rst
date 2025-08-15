5.3 Migration Guide
###################

The 5.3.0 release is a backwards compatible with 5.0. It adds new functionality
and introduces new deprecations. Any functionality deprecated in 5.x will be
removed in 6.0.0.

Upgrade to PHP 8.2
==================

If you are not running on **PHP 8.2 or higher**, you will need to upgrade PHP before updating CakePHP.

.. note::
    CakePHP 5.3 requires **a minimum of PHP 8.2**.

Behavior Changes
================

Database
--------

- ``Query::with()`` now accepts an array of expressions to align with other query clauses. This also
   allows clearing the expressions with an empty array.

View
----

- The ``format()`` and ``currency()`` methods of ``NumberHelper`` now accept also null as input and can return any default string here.
  This allows for easier templates, in particular baked ones. Make sure to adjust any extending helper (plugin or app level) by adding that type.

Deprecations
============

Form
----

- ``Form::_execute()`` is deprecated. You should rename your ``_execute``
  methods to ``process()`` which accepts the same parameters and has the same
  return type..

Http
----

- Using ``$request->getParam('?')`` to get the query params is deprecated.
  Use ``$request->getQueryParams()`` instead.

ORM
---

- Calling behavior methods on table instances is now deprecated. To call
  a method of an attached behavior you need to use
  ``$table->getBehavior('Sluggable')->slugify()`` instead of ``$table->slugify()``.
- ``EntityTrait::isEmpty()`` is deprecated. Use ``hasValue()`` instead.

New Features
============

Command
-------

- ``cake plugin assets symlink`` command now supports a ``--relative`` option to
  create relative path symlinks. This is useful when creating symlinks within
  containers that use volume mounts.
- ``cake server`` now supports a ``--frankenphp`` option that will start the
  development server with `FrankenPHP <https://frankenphp.dev/>`__.

Http
----

- The new ``RateLimitMiddleware`` provides configurable rate limiting for your
  application to protect against abuse and ensure fair usage of resources. It
  supports multiple identification strategies (IP, user, route, API key),
  different rate limiting algorithms (sliding window, fixed window, token bucket),
  and advanced features like custom identifiers, request costs, and dynamic limits.

Cache
-----

- Added Redis Cluster support to ``RedisEngine``. Configure the ``cluster`` option
  with an array of server addresses to enable cluster mode.

Console
-------

- Added ``TreeHelper`` which outputs an array as a tree such as an array of filesystem
  directories as array keys and files as lists under each directory.

Database
--------

- Added support for Entra authentication to SqlServer driver.
- Added ``Query::optimizerHint()`` which accepts engine-specific optimizer hints.
- Added ``Query::getDriver()`` helper which returns the ``Driver`` for the current connection
  role by default.
- Added support for ``year`` column types in MySQL.
- Added support for ``inet``, ``cidr`` and ``macaddr`` network column types to
  postgres driver.
- Added ``TypeFactory::getMapped()`` to retrieve the mapped class name for a specific type.
  This provides a cleaner API compared to using ``TypeFactory::getMap()`` with a type argument.

I18n
----

- Added ``DateTimePeriod`` which wraps a php ``DatePeriod`` and returns ``DateTime``
  instances when iterating.
- Added ``DatePeriod`` which wraps a php ``DatePeriod`` and returns ``Date`` instances
  when iterating.
- Added ``toQuarterRange()`` method to ``DateTime`` and ``FrozenTime`` classes which returns
  an array containing the start and end dates of the quarter for the given date.
- Added ``Date::getTimestamp()``. This method returns an int of the date's
  timestamp.

Mailer
------

- Added ``Message::addAttachment()`` for adding attachments to a message. Like
  other message methods, it can be accessed via the ``Mailer`` instance as ``$mailer->addAttachment()``.

Routing
-------

- ``EntityRoute`` now handles enum value conversions. This enables you to use
  enum backed properties as route parameters. When an enum backed property is
  used in routing, the enum's ``value`` or ``name`` will be used.

Validation
----------

- ``ipOrRange()`` validation has has been added to check for an IP or a range (subnet).

TestSuite
---------

- ``assertRedirectBack()`` added to assert a successful redirect has been made to the same previous URL.
- ``assertRedirectBackToReferer()`` added to assert a successful redirect has been made to the referer URL.
