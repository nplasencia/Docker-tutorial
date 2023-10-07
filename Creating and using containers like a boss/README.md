<p align="center">
  <img src="https://cdn.worldvectorlogo.com/logos/docker.svg" width="250px"/>
</p>

# Creating and using containers like a boss

## Docker networks: concepts for private and public communication in containers

By default, when we start a container, we are really in the background connecting to a particular Docker network (private virtual). By default,
this is the "bridge" network.

Then each one of those networks that you would connect to actually route out through a NAT firewall, which is actually the Docker
daemon configuring the host IP address on its default interface so that our containers can get out to the Internet or to the rest
of our network and then get back.

All the containers on a virtual network can talk to each other without the `-p` option.

It is a best practice to create a new virtual network for each app to isolate them from each other when they are unrelated.

All default Docker configuration works well in many cases, but we can customize many of those:
- Make new virtual networks.
- Attach containers to more than one virtual network or none.
- Skip virtual networks and use host IP using `--net=host`.
- Use different Docker network drivers to gain new abilities.
- And much more...

### Commands

We usually use the `-p` option to indicate which ports to publish. It follows the notation `-p <host-port>:<container-port>`:

```shell
docker container run -p 81:80 --name webhost -d nginx
```

If we want to check what is the port configuration for a running container we can run:

```shell
docker container port webhost
80/tcp -> 0.0.0.0:81
```

This shows which ports are forwarding traffic to that container from the host into the container itself.

#### How can we know the IP of a container?

```shell
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' webhost
```

The `--format` option is a very common option used for formatting the output of commands using "Go templates".

## Docker networks: CLI management of Virtual Networks

### Show networks 

```shell
docker network ls
NETWORK ID     NAME                              DRIVER    SCOPE
3ea12b6d79a7   bridge                            bridge    local
d255090d61f2   host                              host      local
cd17e5a00342   none                              null      local
```

- `Host` network: it is a special network that skips the virtual networking of Docker but sacrifices security: it prevents the
security boundaries of the containerization from protecting the interface of that container. But also, in certain situations,
can improve the performance of high throughput networking.
- `None` network: it removes eth0 and only leaves you with localhost interface in the container.It is kind of the equivalent 
of having an interface on our computer that's not attached to anything. 

### Inspect a network

```shell
docker network inspect
```

```shell
docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "3ea12b6d79a74def2b70ba11542bb26ec2f35267bde3565415768b8a1e4d002d",
        "Created": "2023-09-13T08:30:25.797165824Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "65535"
        },
        "Labels": {}
    }
]
```

We can see a list of Containers which are attached to that specific network. In this example case, there are no containers
attached: `"Containers": {}`.

### Create a network

```shell
docker network create <name_of_the_network> --driver
```

```shell
docker network create my_app_net
35bb5c85623f80725ba95773cf9d8651cad80893ed3267d941525664917bf355

docker network ls
NETWORK ID     NAME                     DRIVER    SCOPE
3ea12b6d79a7   bridge                   bridge    local
d255090d61f2   host                     host      local
35bb5c85623f   my_app_net               bridge    local
cd17e5a00342   none                     null      local
```

*Network driver*: Built-in or third party extensions that give us virtual network features.

```shell
docker container run -d --name new_nginx --network my_app_net nginx
cd9d7af5bca344e1ad476683b78e4cf58c5ff705e3620c8181916580ddf019ac

docker network inspect my_app_net                                  
[
    {
        "Name": "my_app_net",
        "Id": "35bb5c85623f80725ba95773cf9d8651cad80893ed3267d941525664917bf355",
        "Created": "2023-09-13T12:40:00.163697967Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "cd9d7af5bca344e1ad476683b78e4cf58c5ff705e3620c8181916580ddf019ac": {
                "Name": "new_nginx",
                "EndpointID": "b7557a9052a3ff4c134035eddd63f7756e03b78d7afdebf4bda4eca39cac43d1",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

### Attach a network to a container

```shell
docker network connect <name/id_container>
```

### Detach a network to a container

```shell
docker network disconnect <name/id_container>
```

## Docker networks: DNS and how containers find each other

No use IP's. It is an anti-pattern. Use Docker names and the Docker DNS. Docker daemon has a built-in DNS server that containers
use by default.

It is important to mention here that the default `bridge` network foes not have the DNS server built into it by default.
So, we can use the `--link` option when creating the containers. This option allows us to add a link to another container.

But, it is much easier to just create a new network for our apps so we won't need to do all of this.
