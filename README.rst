=================
python-denariusrpc
=================

AuthServiceProxy is an improved version of python-jsonrpc.

It includes the following generic improvements:

* HTTP connections persist for the life of the AuthServiceProxy object
* sends protocol 'version', per JSON-RPC 1.1
* sends proper, incrementing 'id'
* uses standard Python json lib
* can optionally log all RPC calls and results
* JSON-2.0 batch support

It also includes the following denarius-specific details:

* sends Basic HTTP authentication headers
* parses all JSON numbers that look like floats as Decimal,
  and serializes Decimal values to JSON-RPC connections.

Installation
============

1. git clone https://github.com/buzzkillb/python-denariusrpc  
2. cd python-denariusrpc  
3. python setup.py install  

Note: This will only install denariusrpc. If you also want to install jsonrpc to preserve 
backwards compatibility, you have to replace 'denariusrpc' with 'jsonrpc' in setup.py and run it again.

Example
=======
.. code:: python

    from denariusrpc.authproxy import AuthServiceProxy, JSONRPCException

    # rpc_user and rpc_password are set in the denarius.conf file
    rpc_connection = AuthServiceProxy("http://%s:%s@127.0.0.1:32369"%(rpc_user, rpc_password))
    best_block_hash = rpc_connection.getbestblockhash()
    print(rpc_connection.getblock(best_block_hash))

    # batch support : print timestamps of blocks 0 to 99 in 2 RPC round-trips:
    commands = [ [ "getblockhash", height] for height in range(100) ]
    block_hashes = rpc_connection.batch_(commands)
    blocks = rpc_connection.batch_([ [ "getblock", h ] for h in block_hashes ])
    block_times = [ block["time"] for block in blocks ]
    print(block_times)

Logging all RPC calls to stderr
===============================
.. code:: python

    from denariusrpc.authproxy import AuthServiceProxy, JSONRPCException
    import logging

    logging.basicConfig()
    logging.getLogger("DenariusRPC").setLevel(logging.DEBUG)

    rpc_connection = AuthServiceProxy("http://%s:%s@127.0.0.1:32369"%(rpc_user, rpc_password))
    print(rpc_connection.getinfo())

Produces output on stderr like

    DEBUG:DenariusRPC:-1-> getinfo []
    DEBUG:DenariusRPC:<-1- {"connections": 8, ...etc }

Socket timeouts under heavy load
================================
Pass the timeout argument to prevent "socket timed out" exceptions:

.. code:: python

    rpc_connection = AuthServiceProxy(
        "http://%s:%s@%s:%s"%(rpc_user, rpc_password, rpc_host, rpc_port),
        timeout=120)
