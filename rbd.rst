***
RBD
***

.. include:: rbd-qemu.rst

RBD Features
============

В ceph.conf есть параметр rbd_default_features. Там указываются фичи RBD. В
зависимости от версии Ceph можно указывать только числами, либо ещё можно
строками: https://github.com/ceph/ceph/pull/11175.

.. http://www.tablesgenerator.com

+----------------------------+-----------+----------------+-------------------------------------------------------------+
| #define                    | Численное | Строковое      | Описание                                                    |
|                            | значение  | значение       |                                                             |
|                            |           | в ceph.conf    |                                                             |
+============================+===========+================+=============================================================+
| RBD_FEATURE_LAYERING       | 1         | layering       | Layering support. Layering enables you to use cloning.      |
+----------------------------+-----------+----------------+-------------------------------------------------------------+
| RBD_FEATURE_STRIPINGV2     | 2         | striping       | Striping spreads data across multiple objects.              |
|                            |           |                | Striping helps with parallelism for sequential              |
|                            |           |                | read/write workloads.                                       |
+----------------------------+-----------+----------------+-------------------------------------------------------------+
| RBD_FEATURE_EXCLUSIVE_LOCK | 4         | exclusive-lock | When enabled, it requires a client to get a lock            |
|                            |           |                | on an object before making a write.                         |
+----------------------------+-----------+----------------+-------------------------------------------------------------+
| RBD_FEATURE_OBJECT_MAP     | 8         | object-map     | Block devices are thin provisioned. That mean, they         |
|                            |           |                | only store data that  actually exists. Object map           |
|                            |           |                | support helps track which objects actually  exist           |
|                            |           |                | (have data stored on a drive). Enabling object map          |
|                            |           |                | support speeds  up I/O operations for cloning, or           |
|                            |           |                | importing and exporting a sparsely  populated image.        |
+----------------------------+-----------+----------------+-------------------------------------------------------------+
| RBD_FEATURE_FAST_DIFF      | 16        | fast-diff      | Fast-diff support depends on object map support and         |
|                            |           |                | exclusive lock  support. It adds another property to the    |
|                            |           |                | object map, which makes it much  faster to generate diffs   |
|                            |           |                | between snapshots of an image, and the actual  data usage   |
|                            |           |                | of a snapshot much faster.                                  |
+----------------------------+-----------+----------------+-------------------------------------------------------------+
| RBD_FEATURE_DEEP_FLATTEN   | 32        | deep-flatten   | Deep-flatten makes rbd flatten work on all  the snapshots   |
|                            |           |                | of an image, in addition to the image itself. Without it,   |
|                            |           |                | snapshots of an image will still rely on the parent, so the |
|                            |           |                | parent will  not be delete-able until the snapshots are     |
|                            |           |                | deleted. Deep-flatten makes a  parent independent of its    |
|                            |           |                | clones, even if they have snapshots.                        |
+----------------------------+-----------+----------------+-------------------------------------------------------------+
| RBD_FEATURE_JOURNALING     | 64        | journaling     |                                                             |
+----------------------------+-----------+----------------+-------------------------------------------------------------+
| RBD_FEATURE_DATA_POOL      | 128       | data-pool      |                                                             |
+----------------------------+-----------+----------------+-------------------------------------------------------------+


Недорасписанное
===============

* опции для рбд образов типа фастдифф
* бага с удалением снапшотов созданных ранними версиями
* откат к снапшоту крайне медленный (как он работает) и что без дедупликации по сравнению со старыми
  объектам

* Виды кеширования в квм - дока от сусе где демелиоратор сказал что он не прав.
  И описание что есть потеря данных при вырубания питания.

  * https://www.spinics.net/lists/ceph-users/msg15983.html
  * http://docs.ceph.com/docs/master/rbd/qemu-rbd/#qemu-cache-options
  * https://github.com/ceph/ceph/pull/10797

* скруб еррор -- как понять хотябы какой это образ.
* как бекапить :)
* в рбд сразу после снапшота будут наблюдаться тормоза так как 4-мб объекты будут копироваться целиком даже при записи одного сектора.
* оборванное удаление образа. как доудалить остатки.
* Ядерный драйвер RBD не умеет во много опций. в частности, фастдифф. Варианты -- FUSEmount -- каждый файл это образ. либо NBD.
* iscsi
* qemu-nbd vs rbd-nbd
* преобразование в qcow2 и обратно. сжатие qcow2. Перенос в другой пул средством
  qemu-img. хотя более быстро -- на уровне rados.
* Перенос образов между пулами и копирование образов: рекомендуется qemu-img версии
  более 2.9.

  .. image:: _static/qemu-img-bandwith.jpg
     :alt: График пропускной способности

  https://github.com/qemu/qemu/commit/2d9187bc65727d9dd63e2c410b5500add3db0b0d и описание опций.

  .. code-block:: sh

     $ qemu-img convert -m 16 -W -p -n -f raw -O raw \
     rbd:<from_your_pool>/<you_volume>
     rbd:<to_your_pool>/<you_volume>

  Это если ceph.conf настроен. Можно вообще без него:

  .. code-block:: sh

     $ qemu-img convert -p -n -f raw -O raw \
     rbd:<from_your_pool>/<your_volume>:id=cinder:key=<cinder_key>:mon_host=172.16.16.2,172.16.16.3,172.16.16.4 \
     rbd:<to_your_pool>/<your_volume>:id=cinder:key=<cinder_key>:mon_host=172.16.16.2,172.16.16.3,172.16.16.4


* Сделав снапшот хотябы одного образа, сделать снапшот пула уже не получится. Узнать бы почему.
