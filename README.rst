pymongo-job-queue |travis build| |Codecov| |version|
====================================================

This package (``pymjq``) is a simple MongoDB based job queue for Python.
By using capped collections and tailable cursors, you can queue up data
to be consumed by a service worker in order to process your long running
tasks asynchronously.

This is currently used to send notifications on the Meta sites (a.k.a.
Vinylhub, Bibliogs, Filmogs, Gearogs, Comicogs and the Reference Wiki).

Dependencies
^^^^^^^^^^^^

-  pymongo 2.7.2

Install
~~~~~~~

::

    $ pip install pymjq

Examples
~~~~~~~~

::

    >>> from pymongo import MongoClient
    >>> from pymjq import JobQueue
    >>> client = MongoClient("localhost", 27017)
    >>> db = client.job_queue
    >>> jobqueue = JobQueue(db)
    Creating jobqueue collection.
    >>> jobqueue.valid():
    True
    >>> jobqueue.pub({"message": "hello world!"}) # add a job to queue
    True
    >>> for j in jobqueue:
    ...     print (j)
    ...     print (j["data"]["message"])
    ...
    ---
    Working on job:
    {u'status': u'waiting', u'_id': ObjectId('568d963d2c69a1e3ef34da84'),
      u'data': {u'message': u'hello world!'}...
    hello world!
    waiting!
    waiting!
    waiting!
    waiting!
    waiting!
    waiting!
    ...
    ^C Keyboard Interrupt
    >>> jobqueue.pub({"message": "hello again!"}) # add another job to queue
    True
    >>>  j = jobqueue.next()
    True
    >>> print (j["data"]["message"])
    hello again!
    print (j)
    {u'status': u'waiting', u'_id': ObjectId('568d963d2c69a1e3ef34da84'),
      u'data': {u'message': u'hello again!'}...
    >>>

How It Works
~~~~~~~~~~~~

-  `Capped
   collections <http://docs.mongodb.org/manual/core/capped-collections/>`__
   ensure that documents are accessed in the natural order they are
   inserted into the collection.
-  `Tailable
   cursors <http://docs.mongodb.org/manual/tutorial/create-tailable-cursor/>`__
   give us a cursor which will stay open and wait for new documents to
   process if the job queue is empty.
-  The **JobQueue** class has an iterator that yields a document from
   our queue. The iterator will update a doc's status to 'working' and
   then 'done' once the worker has completed it's task.

Jobs
^^^^

Job document, when added to the queue, has the following structure:

.. code:: python


    {
        'ts': {
            'created': datetime,
            'started': datetime,
            'done': datetime
        },
        'status': 'string',
        'site': 'string',
        'data': 'Your job data goes here! Define whatever structure you want. ''
    }

In the ``data`` field, the ``JobQueue.pub()`` method will add whatever
data you pass as a parameter. The ``ts`` attributes will be updated as
the document is worked on.

License
~~~~~~~

MIT Copyright (c) 2016 Discogs

.. |travis build| image:: https://img.shields.io/travis/discogs/pymongo-job-queue.svg
   :target: https://travis-ci.org/discogs/pymongo-job-queue
.. |Codecov| image:: https://img.shields.io/codecov/c/github/discogs/pymongo-job-queue.svg
   :target: https://codecov.io/github/discogs/pymongo-job-queue
.. |version| image:: https://img.shields.io/pypi/v/pymjq.svg
   :target: https://pypi.python.org/pypi/pymjq
