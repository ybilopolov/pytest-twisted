.. -*- mode: rst; coding: utf-8 -*-

==============================================================================
pytest-twisted - test twisted code with pytest
==============================================================================

|Travis|_ |AppVeyor|_ |Pythons|

:Authors: Ralf Schmitt, Kyle Altendorf, Victor Titor
:Version: 1.7.1
:Date:    2018-03-08
:Download: https://pypi.python.org/pypi/pytest-twisted#downloads
:Code: https://github.com/pytest-dev/pytest-twisted


pytest-twisted is a plugin for pytest, which allows to test code,
which uses the twisted framework. test functions can return Deferred
objects and pytest will wait for their completion with this plugin.

Installation
==================
Install the plugin with::

    pip install pytest-twisted


Using the plugin
==================

The plugin is available after installation and can be disabled using
``-p no:twisted``.

By default ``twisted.internet.default`` is used to install the reactor.
This creates the same reactor that ``import twisted.internet.reactor``
would.  Alternative reactors can be specified using the ``--reactor``
option.

Presently only ``qt5reactor`` is supported for use with ``pyqt5``
and ``pytest-qt``. This `guide`_ describes how to add support for
a new reactor.

The reactor is automatically created prior to the first test but can
be explicitly installed earlier by calling
``pytest_twisted.init_default_reactor()`` or the corresponding function
for the desired alternate reactor.

Beware that in situations such as
a ``conftest.py`` file that the name ``pytest_twisted`` may be
undesirably detected by ``pytest`` as an unknown hook.  One alternative
is to ``import pytest_twisted as pt``.


inlineCallbacks
=================
Using `twisted.internet.defer.inlineCallbacks` as a decorator for test
functions, which take funcargs, does not work. Please use
`pytest_twisted.inlineCallbacks` instead::

  @pytest_twisted.inlineCallbacks
  def test_some_stuff(tmpdir):
      res = yield threads.deferToThread(os.listdir, tmpdir.strpath)
      assert res == []

Waiting for deferreds in fixtures
=================================
`pytest_twisted.blockon` allows fixtures to wait for deferreds::

  @pytest.fixture
  def val():
      d = defer.Deferred()
      reactor.callLater(1.0, d.callback, 10)
      return pytest_twisted.blockon(d)


The twisted greenlet
====================
Some libraries (e.g. corotwine) need to know the greenlet, which is
running the twisted reactor. It's available from the
`twisted_greenlet` funcarg. The following code can be used to make
corotwine work with pytest-twisted::

  @pytest.fixture(scope="session", autouse=True)
  def set_MAIN(request, twisted_greenlet):
      from corotwine import protocol
      protocol.MAIN = twisted_greenlet


That's all.


.. |Travis| image:: https://travis-ci.org/pytest-dev/pytest-twisted.svg?branch=master
   :alt: Travis build status
.. _Travis: https://travis-ci.org/pytest-dev/pytest-twisted

.. |AppVeyor| image:: https://ci.appveyor.com/api/projects/status/us5l0l9p7hyp2k6x/branch/master?svg=true
   :alt: AppVeyor build status
.. _AppVeyor: https://ci.appveyor.com/project/vtitor/pytest-twisted

.. |Pythons| image:: https://img.shields.io/pypi/pyversions/pytest-twisted.svg
   :alt: supported Python versions

.. _guide: CONTRIBUTING.rst
