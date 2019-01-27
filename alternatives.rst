************
Альтернативы
************

Rook.io
=======

Это кластер Ceph в докер-контейнерах.

https://rook.io

Почему не стоит его использовать
--------------------------------

* Сeph потребляет много памяти. А к механизму OOM и работе лимитов kubernetes
  есть вопросы.
* Cложности отладки. В Ceph есть баги, от которых падают OSD. Примерно понятно
  как отлаживать их в стандартном окружении, а в kubernetes? Пересобирать образ?
  А зачем тогда вообще rook.io?
* В Сeph нет контрольных сумм при записи (Filestore), так что крайне рекомендуется
  использовать оперативную память с ECC. Если вы разворачиваете rook.io на десктопных
  компьютерахв (Например, Hetzner), то у вас могут быть проблемы.
* Требование 10G-сети.
* А зачем? Если вы в облаке, то пользуйтесь услугами облака.


https://en.wikipedia.org/wiki/Comparison_of_distributed_file_systems
https://en.wikipedia.org/wiki/List_of_file_systems#Distributed_file_systems
https://en.wikipedia.org/wiki/Clustered_file_system#Distributed_file_systems

Универсальное
=============

* Quobyte https://www.quobyte.com/product
* Skylable SX https://www.openhub.net/p/skylable
* Dell EMC Scaleio

Блочные
=======
* StorageOS https://storageos.com

* Dell EMC VxFlex. Сравнение с Ceph: https://habr.com/en/company/croccloudteam/blog/422905

* MinIO https://www.minio.io

* SheepDog

  https://sheepdog.github.io/sheepdog

  https://habr.com/ru/post/412739/2


* StorPool https://storpool.com/

* DRBD


Как распределённая ФС
=====================

* GPFS
* RedHat GFS2
* MooseFS
* LizardFS
* BeeGFS https://www.beegfs.io/
* GlusterFS
* Lustre
* OrangeFS
* XtreemFS http://www.xtreemfs.org/

