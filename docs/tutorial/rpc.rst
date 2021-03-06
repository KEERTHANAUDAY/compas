**********************
Remote Procedure Calls
**********************

.. currentmodule:: compas.rpc

.. highlight:: python

Through ``Xfunc``, COMPAS provides a mechanism for calling Python functions through
a separately launched subprocess. This provides the posibility of, for example,
using functionality that relies on CPython-specific packages (such as Numpy) directly
from Rhino.

A drawback of the ``Xfunc`` mechanism is that evey call launches a new Python
(sub)process with all the overhead that that entails. For infrequent calls to
long-running processes this is not an issue. However, for frequent calls to function
that are expected to run quickly, this is not ideal.

The principle of ``RPC`` is to start a server that handles all requests. The advantage
is that once the server is started, no additional processes have to launched and
the server can handle the requests without any overhead. Therefore, the response
time is much faster than with ``XFunc``.


Basic Usage
===========

The basic usage of RPC is simple.
For example, consider a CPython script using the Force Desnity method. ::

    >>> from compas.numerical import fd_numpy
    >>> result = fd_numpy(...)

This script does not work in Rhino or GH because Numpy is not available.
Therefore, instead of importing and calling the function directly,
we call it through a proxy object. ::

    >>> from compas.rpc import Proxy
    >>> numerical = Proxy('compas.numerical')
    >>> result = numerical.fd_numpy(...)

To use functions from more than one package in the same script, simply change the package attribute
of the proxy object, which determines where the proxy will look for the requested function. ::

    >>> from compas.rpc import Proxy
    >>> proxy = Proxy()

    >>> proxy.package = 'compas.numerical'
    >>> proxy.fd_numpy(...)

    >>> proxy.package = 'compas.geometry'
    >>> proxy.bestfit_plane_numpy(...)

The use of :mod:`compas.rpc` is not restricted to COMPAS packages only. ::

    >>> from compas.rpc import Proxy
    >>> linalg = Proxy('scipy.linalg')
    >>> x = linalg.solve(A, b)

Note that Numpy arrays are automatically converted to lists.


Supported data types
====================

:mod:`compas.rpc` uses JSON serialisation to transfer data between the "client" (your script)
and the server runnning the selected CPython environment.

All COMPAS objects (primitives, shapes, data structures, ...) support JSON serialisation through
their ``to_json`` ``from_json`` methods. On a lower level, these methods convert (complex)
internal data to simple dicts, and vice versa, with ``to_data`` and ``from_data``.

In combination with custom JSON encoders and decoders this allows for COMPAS objects to be
serialised and unserialised without loss of information on either side of the RPC communication network.

Therefore the data types supported by :mod:`compas.rpc` include all native Python data types and COMPAS objects.
Numpy arrays are automatically converted to lists.


Starting and Stopping
=====================

Once a server is started it will keep running "as long as possible".
There are many reasons to stop and (re)start the server during its lifetime.
For example, to load functionality from a different conda environment, or to
load changes that were made to the packages in the environment after it was started.
This happens frequently while a package is still under active development.

Stopping and starting the server is easy. ::

    >>> from compas.rpc import Proxy
    >>> proxy = Proxy()
    >>> proxy.stop_server()
    >>> proxy.start_server()

To restart the server after every call, you can use a context manager.
When used in this way, RPC behaves much like its predecessor ``XFunc``. ::

    >>> with Proxy('compas.numerical') as numerical:
    ...     numerical.fd_numpy(...)
    ...


Loading environments
====================

*PLACEHOLDER*


Starting an RPC server manually
===============================

``Proxy`` will try to start an RPC server automatically
if no server is already running, but very often it is recommended
to start it manually from the command-line.

To start a new RPC server use the following command on the terminal
(default port is ``1753``): ::

    $ compas_rpc start <port>

Conversely, to stop an existing RPC server: ::

    $ compas_rpc stop <port>


.. note::

    If COMPAS is installed in a virtual environment, make sure it is activated
    before trying to use this command-line utility.

.. note::

    Currently, the RPC server is launched on the ``localhost``.
    However, it would also be possible to launch it on a remote computer on a
    network, or on a server reachable over the internet.
