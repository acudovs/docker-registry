# Manage Docker Images

Show help message

```
$ docker-registry -h
usage: docker-registry [-h] [-d] [-f] [-l URL] [-t TIMEOUT] [-u USERNAME]
                       [-p PASSWORD]
                       {catalog,images,delete,manifest,blob} ...

positional arguments:
  {catalog,images,delete,manifest,blob}

optional arguments:
  -h, --help            show this help message and exit
  -d, --debug
  -f, --plain
  -l URL, --url URL
  -t TIMEOUT, --timeout TIMEOUT
  -u USERNAME, --username USERNAME
  -p PASSWORD, --password PASSWORD

usage examples:
    docker-registry catalog [name_regex] [tag_regex]
    docker-registry images [name_regex] [tag_regex]
    docker-registry manifest name tag
    docker-registry blob name layer_digest output_file
    docker-registry delete name image_digest
```

Show catalog

```
$ docker-registry catalog
+----------------------+-----------------------+----------+
| REGISTRY             | NAME                  | TAG      |
+----------------------+-----------------------+----------+
| registry.example.com | centos                | latest   |
| registry.example.com | centos7               | testing  |
| registry.example.com | centos7               | stable   |
| registry.example.com | centos7-base          | testing  |
| registry.example.com | centos7-base          | stable   |
+----------------------+-----------------------+----------+
|                                                Total: 5 |
+----------------------+-----------------------+----------+
```

Show images

```
docker-registry images centos stable
+----------------------+-----------------------+--------+-------------------------------------------------------------------------+-----------+
| REGISTRY             | NAME                  | TAG    | DIGEST                                                                  | SIZE      |
+----------------------+-----------------------+--------+-------------------------------------------------------------------------+-----------+
| registry.example.com | centos7               | stable | sha256:0d121fa7987c60c3f7ecb8d7347d8e86683018625e44f3864e69b388087a4d0b | 69901729  |
| registry.example.com | centos7-base          | stable | sha256:f6cd3c520b44bc3a2263c91a5d02822e6698033f376892ad78fc5c433fb1989e | 143913184 |
+----------------------+-----------------------+--------+-------------------------------------------------------------------------+-----------+
|                                                                                                                                    Total: 2 |
+----------------------+-----------------------+--------+-------------------------------------------------------------------------+-----------+
```

Show image manifest

```
$ docker-registry manifest centos7 stable
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/octet-stream",
      "size": 2574,
      "digest": "sha256:feac5e0dfdb25597c61e19bf3c0a51bb865e7ba5e210eaca66648c02e0ff0d61"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 69901729,
         "digest": "sha256:a2392627bec4b444fc1eeee61f67c569116d85eac8a9ae20600e87d8a2ea9223"
      }
   ]
}
```

Download layer blob

```
$ docker-registry blob centos7 sha256:a2392627bec4b444fc1eeee61f67c569116d85eac8a9ae20600e87d8a2ea9223 /tmp/centos7.tar.gz
```

Delete image

```
$ docker-registry delete centos7 sha256:0d121fa7987c60c3f7ecb8d7347d8e86683018625e44f3864e69b388087a4d0b
```
