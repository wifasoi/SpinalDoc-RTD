
Wishbone
========

Introduction
------------
The (Wishbone)[] bus is an open standard for interconnecting IP cores toghether.
The wishbone supports:

* pipelined comunication between IPs
* burst
* optional tags

Configuration and instanciation
-------------------------------

The ``Wishbone`` Bundle has a construction argument ``WishboneConfig``. For more information the Wishbone spec could be find `there <http://cdn.opencores.org/downloads/wbspec_b4.pdf>`_.

.. code-block:: scala

   case class WishboneConfig(
     val addressWidth : Int,
     val dataWidth : Int,
     val selWidth : Int = 0,
     val useSTALL : Boolean = false,
     val useLOCK : Boolean = false,
     val useERR : Boolean = false,
     val useRTY : Boolean = false,
     val useCTI : Boolean = false,
     val tgaWidth : Int = 0,
     val tgcWidth : Int = 0,
     val tgdWidth : Int = 0,
     val useBTE : Boolean = false
   ){
     def useTGA = tgaWidth > 0
     def useTGC = tgcWidth > 0
     def useTGD = tgdWidth > 0
     def useSEL = selWidth > 0

     def isPipelined = useSTALL

     def pipelined : WishboneConfig = this.copy(useSTALL = true)

     def withDataTag(size : Int)    : WishboneConfig = this.copy(tgdWidth = size)
     def withAddressTag(size : Int) : WishboneConfig = this.copy(tgaWidth = size)
     def withCycleTag(size : Int)   : WishboneConfig = this.copy(tgdWidth = size)
     def withCycleTypeIdentifier    : WishboneConfig = this.copy(useCTI = true)
     def withBurstType              : WishboneConfig = this.copy(useCTI = true, useBTE = true)
   }

This configuration object has also some functions to provide some ``WishboneConfig`` templates :

.. list-table::
   :header-rows: 1

   * - Name
     - Return
     - Description
   * - pipelined
     - WishboneConfig
     - Return a wishbone configuration that support pipelined transaction
   * - withDataTag(size : Int)
     - WishboneConfig
     - Return a wishbone configuration with data tag of specidied size
   * - withAddressTag(size : Int)
     - WishboneConfig
     - Return a wishbone configuration with address tag of specidied size
   * - withCycleTag(size : Int)
     - WishboneConfig
     - Return a wishbone configuration with cycle tag of specidied size
   * - withCycleTypeIdentifier
     - WishboneConfig
     - Return a wishbone configuration with type identifier enabled
   * - withBurstType
     - WishboneConfig
     - Return a wishbone configuration with type identifier enabled


You can check the bus configuration with:

.. list-table::
   :header-rows: 1

   * - Name
     - Return
     - Description
   * - useTGA
     - Boolean
     - Return true if the address tag line is used
   * - useTGC
     - Boolean
     - Return true if the cycle tag line is used
   * - useTGD
     - Boolean
     - Return true if the data tag lines are used
   * - useSEL
     - Boolean
     - Return true if the selection line is used
   * - useSTALL
     - Boolean
     - Return true if the stall line is used (same as isPipelined)
   * - useLOCK
     - Boolean
     - Return true if the lock line is used
   * - useERR
     - Boolean
     - Return true if the error line is used
   * - useRTY
     - Boolean
     - Return true if the retry line is used
   * - useCTI
     - Boolean
     - Return true if the Cycle Type Identifie tag line is used
   * - useBTE
     - Boolean
     - Return true if the Burst Type Extension tag line is used
   * - isPipelined
     - Boolean
     - Return true if the bus support pipelined interfacing (same as use STALL)


.. code-block:: scala

   // You can define it in this way
   val myWishboneConfig1 =  WishboneConfig(
                             addressWidth = 8,
                             dataWidth = 16,
                             useSTALL = true
                           )

   // Or you can define it in this way
   val myWishboneConfig2 =  WishboneConfig(8,16).pipelined

   // you can create a wishbone bus in this way
   val wb = Wishbone(myWishboneConfig2)

   // You can check the configuration like this
   wb.config.isPipelined // will return true
   wb.config.dataWidth   //will return 8

Wishbone components
-------------------
The wishbone library has some other componet, like:

- WishboneDecoder
- WishboneArbiter
- WishboneIntercon
- WishboneAdapter

all the componet and utilities are in ``spinal.lib.bus.wishbone``

WishboneDecoder
^^^^^^^^^^^^^^^
This device is usefull when you need to connect multiple wishbone slaves to a master.

You can istantiate it in this way:

.. todo::
   check this code

.. code:: scala

   val salves = Map(
    slave1 -> (0x00000000L,   4 kB),
    slave2 -> (0x40000000L,  64 MB),
    slave3 -> (0xF0000000L,   1 MB)
   )

   val decoder = WishboneDecoder(master, slaves)
   # master and slaves need to have the same config,
   # the only thing allowed to be different is he address size
   # (but master.addressWidth >= slaveX.addressWidth)

WishboneArbiter
^^^^^^^^^^^^^^^
This device is usefull when you need to connect multiple master to a slave.
If two or more masters try to access the slave, The WishboneArbiter will wait the end of the translation,
and hand over the bus next master (only prioritized round-robin is supported).

You can instantiate it in this way:

.. code:: scala

   # The master order in the array matters!
   # master1 have priority over master2 and so on
   val arbiter = WishboneArbiter([master1,master2,master3], slave)

   # masters and slave need to have the same config,
   # the only thing allowed to be different is he address size
   # (but master.addressWidth >= slaveX.addressWidth)

WishboneIntercon
^^^^^^^^^^^^^^^^
This device is usefull if you need to connect multiple masters to multiple slaves.
WishboneArbiter is builded with ``WishboneDecoder`` and ``WishboneArbiter`` connected togheder.

You can istantiate it in this way:

.. todo::
   add code

.. todo::
   spellcheck and proofread