LinuxにおけるStorageにまつわる用語整理
===

# Block devices

Block deviceはある大きさのブロック単位で、ランダムにデータの読み書きを行えるデバイス。ハードディスクやUSBメモリやCD-ROMなどのディスク装置全般がBlock device。

# filesystem

filesystemはBlock deviceにおけるデータ操作(読み書き)を行うためのOSの機能。
Unix系OSでは仮想的なファイルシステムを生成する。
Unix系OSはルートディレクトリをひとつ持ち、すべてのファイルはルートディレクトリ以下に配置される。
["On a UNIX system, everything is a file; if something is not a file, it is a process."](http://www.tldp.org/LDP/intro-linux/html/sect_03_01.html)
ブロックデバイス上のファイルにアクセスするために、それらをディレクトリツリー上のどこに置くかを指示する必要がある。これをマウントと呼ぶ。
マウントポイントの決め方には慣習がある。[File Hierarcy Standard](https://ja.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)。
Linuxでは、現在のkernelでサポートしているfilesystemは

```bash
$ cat /proc/filesystems
```

で確認できる。

# Partitions

Partitionはハードディスクを分割したlogical diskのこと。`fdisk`コマンドを利用する。

# Volumes

Volumeはハードウェア上の記憶領域。

# LVM

LVM(Logical Volume Management)はlogical volumeやfilesystemを管理するシステム。diskをひとつ以上のセグメントにパーティショニングしたり、それらのPartitionをfilesystemを利用してフォーマットするtraditionalな手法よりも高度で柔軟。
キーコンセプトは３つ。Volume Groups、Physical Volumes、Logical Volumes。

Volume GroupsはPhysical VolumesやLogical Volumesの名前のついた集合。

Physical Volumesはdiskに相当する。Logical Volumesを保持する領域を提供するBlock devices。

Logical Volumesはpartitionに相当する。Logical Volumesはfilesystemを持つ。partitionと異なり、番号よりも名前を取得し、複数のdiskにまたがっても良いし、物理的に連続である必要はない。


#### Refs

- [Block devices](http://itpro.nikkeibp.co.jp/article/Keyword/20081023/317625/)
- [ファイルシステム @ Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0)
- [LinuxFilesystemsExplained @ help.ubuntu.com](https://help.ubuntu.com/community/LinuxFilesystemsExplained)
- [Lvm @ wiki.ubuntu.com](https://wiki.ubuntu.com/Lvm)
- [man5 @ manpage.ubuntu.com](http://manpages.ubuntu.com/manpages/xenial/man5/fs.5.html)
- [lvm manpage @ die.com](https://linux.die.net/man/8/lvm)
- [fdisk manpage @ die.com](https://linux.die.net/man/8/fdisk)

