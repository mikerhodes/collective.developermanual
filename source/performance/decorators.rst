=================
Cache decorators
=================

.. admonition:: Description

    How to use the Python decorator pattern to cache the result values of
    your computationally expensive method calls.

.. contents :: :local:

Introduction
============

Cache decorators are convenient methods caching of function return values.

They work like this::

   @cache_this_function
   def my_slow_function():
        # This is run only once and all subsequent calls get value from the cache
        return  

.. warning ::

    Cache decorators do not work with methods or functions using generators
    (``yield``). The cache stores empty value.

The `plone.memoize <http://pypi.python.org/pypi/plone.memoize>`_ package
offers helpful function decorators to cache return values.

See also :doc:`using memcached backend for memoizers </performance/ramcache>`. 

Cache result for process lifecycle
==================================

Example::

    from plone.memoize import forever

    @forever.memoize
    def getFields(area, subject):
        """ Get all fields inside area / subject.

        Results is cached for process lifetime.

        @return: List of internal fields
        """
        schema = getSchema(area)
        return [field for field in schema if field["subject"] == subject]


Timeout caches
==============

Please read `Timed caching decorator with plone.memoize <http://danielnouri.org/blog/devel/plone-memoize-timeout.html?showcomments=yes>`_.

* `Another example <https://svn.plone.org/svn/collective/collective.externalcontent/trunk/collective/externalcontent/tests/test_vocabulary.py>`_.

Caching per request
===================

This pattern shows how to avoid recalculating the same value repeatedly
during the lifecycle of an HTTP request object.

Caching on BrowserViews
------------------------

This is useful if the same view/utility is going to be called many times
from different places for the same HTTP request.

The `plone.memoize.view <https://github.com/plone/plone.memoize/tree/master/plone/memoize/view.txt>`_
package provides necessary decorators for BrowserView based classes.


.. code-block:: python

    from plone.memoize.view import memoize, memoize_contextless

    class MyView(BrowserView):

        @memoize
        def getValue():
            """ This value is recalculated for every new BrowserView context
                per request 
            """
            return "something"

        @memoize_contextless
        def getValueNoContext():
            """ This value is recalculated for all context objects once per
                request
            """
            return "something"

Caching on Archetypes accessors
---------------------------------

If you have a custom 
:doc:`Archetypes accessor method </content/archetypes/fields>`,
you can avoid recalculating it during the request processing.

Example::

    def getParsedORADataCached(self):
        """ Same as above, but does not run through JSON reader every time.
        """

        # Manually store the result on HTTP request
        # object annotations 

        # Use function name + Archetypes unique identified as the key
        key = "parsed-ora-data-" + self.UID()

        cache = IAnnotations(self.REQUEST)
        data = cache.get(key, None)
        if data is not None:
            data = self.getParsedORAData()
            cache[key] = data 

        return data

Caching using global HTTP request
----------------------------------

This example uses `five.globalrequest package <http://pypi.python.org/pypi/five.globalrequest>´_ 
for caching. Values are stored on thread-local HTTPRequest object which lasts
the transaction lifecycle.

::


     from zope.globalrequest import getRequest
     from zope.annotation.interfaces import IAnnotations


        def _getProductList(self, type, language):
            """

            """

            logger.info("Getting product list %s %s" % (type, language))
            ...
            return result


        def getProductListCached(self, type, language):
            """

            """

            request = getRequest()

            key = "cache-%s-%s" % (type, language)

            cache = IAnnotations(request)
            data = cache.get(key, None)
            if not data:
                data = self._getProductList(type, language)
                cache[key] = data

            return data



Other resources
===============

* `plone.memoize source code <https://github.com/plone/plone.memoize/tree/master/plone/memoize/>`_

* `zope.app.cache source code <http://svn.zope.org/zope.app.cache/trunk/src/zope/app/cache/>`_


