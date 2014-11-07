.. -*- rst -*-

National Stock Exchange of India Limit Order Book Simulation 
============================================================
This package implements a limit order book that simulates the 
processing of limit/market orders on India's National Stock Exchange.

Requirements
------------
The code requires the following software for installation (older versions may
work, but haven't been tested):

* Python 2.7 or later.
* `cython <http://www.cython.org/>`_ 0.19.1 or later.
* `numpy <http://www.numpy.org/>`_ 1.7.0 or later.
* `pandas <http://pandas.pydata.org/>`_ 0.10 or later.
* `odict <https://github.com/bluedynamics/odict/>`_ 1.5.0 or later.
* `rbtree <https://bitbucket.org/bcsaller/rbtree/>`_ 0.9.0 or later.

Installation
------------
Build the extension by running: ::

    python setup.py build_ext --inplace

Running the Simulation
----------------------
To run the simulation, invoke the simulation script with a specified firm name,
output directory, and list of input files. For example: ::

     python lob.py INCI ./output INCI-orders-03092013.csv.gz INCI-orders-03102013.csv.gz
     
A sample data file (``EXAMPLE-orders.csv``) is included. A script for launching
the code on a Sun Grid Engine cluster is also included; the script requires the
`drmaa-python <http://drmaa-python.github.io/>`_ package. To use the script, replace
the listed security names accordingly.

Input File Format
-----------------
The simulation requires input files in CSV format comprising the following
columns with the indicated byte lengths. The input file may be compressed with
gzip.

record indicator (2)
  Ignored.
segment (4)
  Ignored.
order number (16)
  8 left-most digits are date YYYYMMDD, followed by 00000001-99999999.
transaction date (10)
  MM/DD/YYYY
transaction time (14)
  HH:MM:SS.XXXXXX, where XXXXXX is microseconds.
buy/sell indicator (1)
  Must be either 'B' or 'S'.
activity type (1)
  Must be 1 (order add), 3, (order cancel), or 4 (order modify).
symbol (10)
  Firm identifier.
instrument (6)
  Ignored.
expiry date (10)
  MM/DD/YYYY
strike price (variable)
  Integer.
option type (2)
  Ignored.
volume disclosed (variable)
  Integer.
volume original (variable)
  Integer.
limit price (variable)
  Float.
trigger price (variable)
  Float.
market flag (1)
  'Y' for market order, 'N' for limit order.
stop loss flag (1)
  'Y' for stop loss order, 'N' for regular lot order.
Immediate-or-Cancel (IOC) flag (1)
  'Y' for IOC, 'N' for non-IOC.
spread/combination type (1)
  Ignored.
algo indicator (1)
  Ignored.
client identity flag (1)
  Ignored.

Methodology and Implementation
------------------------------
The limit order book is implemented as two red-black trees of queues
corresponding to different buy and sell price levels. The use of red-black trees
accelerates determination of the bid and ask prices at any step of the
simulation. Further acceleration is achieved by compiling the simulation with Cython.

Order processing is restricted to the orders with the first futures expiration date
observed during processing; all other orders are ignored.

Submitted orders may be add requests, modification requests, or cancellation
requests. Both market and limit orders are supported; during processing, the
former are discarded if they fail to match any orders already in the
book. Residual volume for IOC orders is discarded.  Orders that have explicitly
disclosed (i.e., non-zero) volumes are assumed to be hidden; if any such orders
in a price level queue, they are matched against a new incoming order AFTER
orders with zero disclosed volume.

Daily stats are accumulated during the simulation and are reset when the date
associated with the processed orders changes. 

Author
------
The code was written by Lev Givon in 2012-2013 for Prof.
Costis Maglaras at Columbia Unversity's Business School.

License
-------
See included LICENSE file.
