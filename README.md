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
$ docker-registry catalog centos
+-----------+--------------+---------+
| REGISTRY  | NAME         | TAG     |
+-----------+--------------+---------+
| localhost | centos       | latest  |
| localhost | centos7      | stable  |
| localhost | centos7-base | testing |
+-----------+--------------+---------+
|                          Total: 4  |
+-----------+--------------+---------+
```
