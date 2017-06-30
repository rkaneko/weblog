Docker containerにおけるデータの取り扱い
===

#### materials

- [Manage data in containers](https://docs.docker.com/engine/tutorials/dockervolumes/)

Docker engineを利用したcontainer間、container内のデータを取り扱う方法は主に２つある。

- Data volumes
- Data volumes containers

## Data volumes

Data volumesは**Union File System**をバイパスする一つ、または複数のcontainerにおける特別にデザインされたディレクトリ。データの永続化や共有のためにいくつかの特徴を提供する。

- containerが作成されたときにVolumesは初期化される。特定のマウントポイントにおいてcontainerのベースイメージがデータを所持していれば、Volumeの初期化において新しいVolumeに既存データはコピーされる。ただし、ホストディレクトリにマウントするときはこれは適用されない。
- Data volumesはcontainer間で共有・再利用ができる。
- Data volumesへの変更は直接行われる。
- Data volumesへの変更はイメージへの更新をする際には含まれない。
- container自体が削除されてもData volumesは永続化されている。

## Add a data volume

`docker create`や`docker run`コマンドにオプション`-v`をつけて複数のdata volumesを利用することができる。

```bash
$ docker run -d -P --name web -v /webapp training/webapp python app.py
```

Dockerfileで`VOLUME` instructionを利用して1つ以上の新しいvolumesをcontainer作成時に指定することができる。

## Mount a host directory as a data volume

Docker engineのホストのディレクトリをcontainerのディレクトリにマウントすることもできる。

```bash
$ docker run -d -P --name web -v /src/webapp:/webapp training/webapp python app.py
```

上のコマンドでは、ホストの`/src/webapp`ディレクトリをcontainerの`/webapp`ディレクトリにマウントしている。
すでにcontainer内に`/webapp`ディレクトリがあれば、そのまま利用して既存のデータを削除しない。

containerのディレクトリ指定は常に`/path/to/dir`のような絶対パスでなければならない。

ホストのディレクトリは絶対パスや一意に決まるnameを指定する。nameを指定するとDockerが作成する。nameはアルファヌメリックで始める必要がある。

次のようにDocker containerのディレクトリに対しread-onlyを指定することも可能。

```bash
$ docker run -d -P --name web -v /src/webapp:/webapp:ro training/webapp python app.py
```

また、`cached`を指定すると読み込みが重たくなるような処理でパフォーマンス改善につながりうる。ただし、ホストとの一時的な一貫性が失われる。

```bash
$ docker run -d -P --name web -v /src/webapp:/webapp:cached training/webapp python app.py
```

## Volume labels

SELinuxのようなlabeling systemでは、containerに適切なlabelを配置する必要がある。

## Mount a host file as a data volume

`-v`オプションではディレクトリの他にファイルを指定することもできる。
ユースケースとしてはホストの`.bash_history`をcontainer内で利用するなどが挙げられる。

## Creating and mounting a data volume container

container間でデータ共有をしたり、non-persistent containerからデータを利用したい場合、**named Data Volume Containerを作成して、そこからデータをマウントする**のが最もよい方法。

dockerのstorage driver等は以下を前提とする。

```bash
$ sudo docker info

Server Version: 17.03.1-ce
Storage Driver: aufs

Plugins:
 Volume: local

```

まずはVolumeを作成する。

```bash
$ sudo docker volume create --opt o=size=50G elasticsearch-data
```

volumeをリストする。

```bash
$ sudo docker volume ls
```

生成されたvolumeの構成を見る。

```bash
$ sudo docker volume inspect elasticsearch-data
[
{
    "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/elasticsearch-data/_data",
        "Name": "elasticsearch-data",
        "Options": {
            "o": "size=50G"
        },
        "Scope": "local"
}
]
```


生成したvolumeは以下のコマンドで削除できる。

```bash
$ sudo docker volume rm elasticsearch-data
```


