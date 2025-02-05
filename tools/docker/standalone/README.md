# How to deploy OceanBase with docker

OceanBase provide a standalone test image named [oceanbase-ce](https://hub.docker.com/r/oceanbase/oceanbase-ce) for OceanBase Database. By default, this image deploys a MINI_MODE OceanBase instance.

**WARNING**

- The oceanbase-ce docker image is just used for study or test;
- Please use [oceanbase-operator](https://github.com/oceanbase/ob-operator) instead if you want to deploy oceanbase in k8s;
- You should not deploy it with important data as it is not used in production environment.

Reasons:

1. The cluster contains only one instance, so there is no disaster tolerant ability;
2. It is very difficult to recover after failure because docker container cannot started while the oceanbase instance cannot start success, which means you lost the container and the data with it;
3. K8s can not restart a new pod because the container still exists after the observer process quit.

## Prerequisite

Before you deploy oceanbase-ce docker, do the following checks:

- Make sure that your machine has enough resource that can execute at least 2 phycical core and 8GB memory.
- Your machine has installed these applications:

    | Application | Recommended version | Documentation                                               |
    | ----------- | ------------------- | ----------------------------------------------------------- |
    | Docker      | Latest              | [Docker Documentation](https://docs.docker.com/get-docker/) |
- You have started the Docker service on your machine.

## Start an OceanBase instance

To start an OceanBase instance, run this command:

```bash
# deploy mini instance
docker run -p 2881:2881 --name oceanbase-ce -d oceanbase/oceanbase-ce

# deploy an instance of the largest size according to the current container
docker run -p 2881:2881 --name oceanbase-ce -e MINI_MODE=0 -d oceanbase/oceanbase-ce
```

Two to five minutes are necessary for the boot procedure. To make sure that the boot procedure is successful, run this command:

```bash
$ docker logs oceanbase-ce | tail -1
boot success!
```

## Connect to an OceanBase instance

oceanbase-ce image contains obclient (OceanBase Database client) and the default connection script `ob-mysql`.

```bash
docker exec -it oceanbase-ce ob-mysql sys # Connect to sys tenant
docker exec -it oceanbase-ce ob-mysql root # Connect to the root account of a general tenant
docker exec -it oceanbase-ce ob-mysql test # Connect to the test account of a general tenant
```

Or you can run this command to connect to an OceanBase instance with your local obclient or MySQL client.

```bash
mysql -uroot -h127.1 -P2881
```

## Supported environment variables

This table shows the supported environment variables of the current oceanbase-ce mirror version:

| Variable name    | Default value | Description                                                  |
| ---------------- | ------------- | ------------------------------------------------------------ |
| MINI_MODE        | false         | If ture, will use mini mode to deploy OceanBase Database instance, it should be used only for research/study/evaluation.  DO NOT use it for production or performance testing. |
| EXIT_WHILE_ERROR | true          | Whether quit the container while start observer failed. If start observer failed, you can not explore the logs as the container will exit. But if you set the EXIT_WHILE_ERROR=false, the container will not exit while observer starting fail and you can use docker exec to debug. |

## Run the Sysbench script

oceanbase-ce image installs the Sysbench tool by default. And the Sysbench tool is configured. You can run these commands in sequence to run the Sysbench script with the default configurations.

```bash
docker exec -it oceanbase-ce obd test sysbench obcluster
```

## Mount Volumn
You can use `-v /your/host/path:/container/path` parameter in docker `run` command to save data in host os if you want to persistence the data of a container.

Below is an example.

```bash
docker run -d -p 2881:2881 -v $PWD/ob:/root/ob -v $PWD/obd:/root/.obd --name oceanbase oceanbase/oceanbase-ce
```

Note that you should use your own path.

The docker image `oceanbase-ce` saves the data to /root/ob directory default. You should bind both the /root/ob and /root/.obd. You can not start new docker image if you only bind the /root/ob directory, because the docker image oceanbase-ce uses the [obd](https://github.com/oceanbase/obdeploy) to manage database clusters and there is no information about the database cluster in a new docker container.

You can view more information about `docker -v` at [docker volumn](https://docs.docker.com/storage/volumes/).
