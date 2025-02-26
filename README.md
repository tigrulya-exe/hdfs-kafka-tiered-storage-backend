# HDFS Storage Backend for Kafka Tiered Storage

Current project is an implementation of Storage Backend for Apache Kafka [RemoteStorageManager implementation](https://github.com/Aiven-Open/tiered-storage-for-apache-kafka) by Aiven.

Currently, `3.2.4` and `3.3.6` HDFS versions are supported. 
The version of HDFS can be specified in the `hadoopVersion` property during project build, e.g.

```shell
./gradlew clean build testClasses -x test -x integrationTest -PhadoopVersion=3.3.6
```

### Local demo

To run the Kafka instance with HDFS storage backend locally, follow these steps:
1. Run `make docker_image` to build the docker image of Kafka with the necessary dependencies. 
2. Follow the instruction from [demo/README.md](demo/README.md)

### Configuration of HDFS Storage Backend

| Name                                              | Type    | Default value | Description                                                                                                                                                                                                                                        |
|---------------------------------------------------|---------|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `rsm.config.storage.hdfs.root`                    | String  | /             | The base directory path in HDFS relative to which all uploaded file paths will be resolved                                                                                                                                                         |
| `rsm.config.storage.hdfs.core-site.path`          | String  | -             | Absolute path of core-site.xml                                                                                                                                                                                                                     |
| `rsm.config.storage.hdfs.hdfs.hdfs-site.path`     | String  | -             | Absolute path of hdfs-site.xml                                                                                                                                                                                                                     |
| `rsm.config.storage.hdfs.upload.buffer.size`      | int     | 8192          | Size of the buffer used during file upload                                                                                                                                                                                                         |
| `rsm.config.storage.hdfs.auth.enabled`            | boolean | false         | Whether to enable Kerberos HDFS authentication. If enabled, corresponding value should be provided in `hadoop.security.authentication` in hadoop XML config files or in `rsm.config.storage.hdfs.conf.hadoop.security.authentication` kafka option |
| `rsm.config.storage.hdfs.auth.kerberos.principal` | String  | -             | Kerberos principal to be used in HDFS client                                                                                                                                                                                                       |
| `rsm.config.storage.hdfs.auth.kerberos.keytab`    | String  | -             | Absolute path of the keytab file with the credentials for the principal                                                                                                                                                                            |
| `rsm.config.storage.hdfs.conf.*`                  |         |               | HDFS configuration options. `rsm.config.storage.hdfs.conf.hdfs-option.key` will be trasnformed to HDFS `hdfs-option.key` option.                                                                                                                   |

### Example configuration of Kafka brokers with Aiven RemoteStorageManager support

```properties
# ----- Enable tiered storage -----

remote.log.storage.system.enable=true

# ----- Configure the remote log manager -----

# This is the default, but adding it for explicitness:
remote.log.metadata.manager.class.name=org.apache.kafka.server.log.remote.metadata.storage.TopicBasedRemoteLogMetadataManager

# Put the real listener name you'd like to use here:
remote.log.metadata.manager.listener.name=PLAINTEXT

# You may need to set this if you're experimenting with a single broker setup:
#rlmm.config.remote.log.metadata.topic.replication.factor=1

# ----- Configure the remote storage manager -----

# Here you need either one or two directories depending on what you did in Step 1:
remote.log.storage.manager.class.path=/path/to/core/*:/path/to/storage/*
remote.log.storage.manager.class.name=io.aiven.kafka.tieredstorage.RemoteStorageManager

# 4 MiB is the current recommended chunk size:
rsm.config.chunk.size=4194304

# ----- Configure the storage backend -----

# Using HDFS backend as an example:
rsm.config.storage.backend.class=io.arenadata.kafka.tieredstorage.storage.hdfs.HdfsStorage
rsm.config.storage.hdfs.root=/tmp/kafka/
rsm.config.storage.hdfs.hdfs.core-site.path=/etc/conf/core-site.xml
rsm.config.storage.hdfs.hdfs.hdfs-site.path=/etc/conf/hdfs-site.xml
# The prefix can be skipped:
#rsm.config.storage.key.prefix: "some/prefix/"

# ----- Configure the fetch chunk cache -----

rsm.config.fetch.chunk.cache.class=io.aiven.kafka.tieredstorage.fetch.cache.DiskChunkCache
rsm.config.fetch.chunk.cache.path=/cache/root/directory
# Pick some cache size, 16 GiB here:
rsm.config.fetch.chunk.cache.size=17179869184
# Prefetching size, 16 MiB here:
rsm.config.fetch.chunk.cache.prefetch.max.size=16777216
# Cache retention time ms, where -1 represents infinite retention
rsm.config.fetch.chunk.cache.retention.ms=600000
```