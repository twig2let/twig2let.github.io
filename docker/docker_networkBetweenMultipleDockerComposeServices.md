# Networking Between Multiple Docker Compose Services

Sometimes you may need to communicate between services defined across multiple compositions I.E services that Docker will start on isolated networks. To enable our services to talk one another as if they had all been defined in one docker-compose.yml we can leverage Docker's `user-defined networks`, in this case the [`Bridged`](https://docs.docker.com/engine/userguide/networking/#bridge-networks) network.

## Demonstrating The Problem
___

To illustrate the issue, let's assume we have two docker-compose.yml, **front/docker-compose.yml** and
**back/docker-compose.yml**.

**front/docker-compose.yml**

```yaml
version: '2'

services:
  front:
    image: tutum/curl:alpine
    command: curl back
```

**back/docker-compose.yml**

```yaml
version: '2'

services:
  back:
    image: nginx
    command: /bin/bash nginx
```

Start the services with a `docker-compose up`, **back** service first and then the **front** service; you should see the following output:

```
Recreating front_front_1 ...
Recreating front_front_1 ... done
Attaching to front_front_1
front_1  | curl: (6) Couldn't resolve host 'back'
front_front_1 exited with code 6
```

The problem is quite clear, usually we can rely on Docker to resolve the container names for us, but in this instance our services are running on isloated networks and thus do not support automatic service discovery.

## A Solution
___


[`user-defined networks`](https://docs.docker.com/engine/userguide/networking/#user-defined-networks) to the rescue! As the official Docker documentation says:

```
"It is recommended to use user-defined bridge networks to control which containers can communicate with each other, and also to enable automatic DNS resolution of container names to IP addresses"
```

### Creating a custom bridge network

1. Create the network

      ```bash
      $ docker network create -d bridge my-custom-network
      $ docker network ls
      ```

      You should see `my-custom-network` in the list

2. Now update both docker-compose.yml as follows:

    **front/docker-compose.yml**

    ```yaml
    version: '2'

    services:
      front:
        image: tutum/curl:alpine
        command: curl back
    networks:
      my-custom-network:
        external: true
    ```

    **back/docker-compose.yml**

    ```yaml
    version: '2'

    services:
      back:
        image: nginx
        command: nginx -g 'daemon off;'
    networks:
      my-custom-network:
      external: true
    ```

    We've just specified the network we would like to user, `my-custom-network`.

    *NOTE: `external:true` specifies that the network (`my-custom-network`) has been created outside of compose and therefore docker-compose should not attempt to create it. It will also throw and error if the network does not exist.*

3. Now instruct the services to use this network

    **front/docker-compose.yml**

      ```yaml
    version: '2'

    services:
      front:
        image: tutum/curl:alpine
        command: curl back
        networks:
          - my-custom-network
    networks:
      my-custom-network:
        external: true
      ```

      **back/docker-compose.yml**

      ```yaml
    version: '2'

    services:
      back:
        image: nginx
        command: nginx -g 'daemon off;'
        networks:
          - my-custom-network
    networks:
      my-custom-network:
        external: true
    ```

4. Start the services with a `docker-compose up`, **back** service first and then the **front** service; you should see the following output:

    ```
    Starting front_front_1 ...
    Starting front_front_1 ... done
    Attaching to front_front_1
    front_1  |   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
    front_1  |                                  Dload  Upload   Total   Spent    Left  Speed
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
    front_1  | <html>
    front_1  | <head>
    front_1  | <title>Welcome to nginx!</title>
    front_1  | <style>
    front_1  |     body {
    front_1  |         width: 35em;
    front_1  |         margin: 0 auto;
    front_1  |         font-family: Tahoma, Verdana, Arial, sans-serif;
    front_1  |     }
    front_1  | </style>
    front_1  | </head>
    front_1  | <body>
    front_1  | <h1>Welcome to nginx!</h1>
    front_1  | <p>If you see this page, the nginx web server is successfully installed and
    front_1  | working. Further configuration is required.</p>
    front_1  |
    front_1  | <p>For online documentation and support please refer to
    front_1  | <a href="http://nginx.org/">nginx.org</a>.<br/>
    front_1  | Commercial support is available at
    front_1  | <a href="http://nginx.com/">nginx.com</a>.</p>
    front_1  |
    front_1  | <p><em>Thank you for using nginx.</em></p>
    front_1  | </body>
    front_1  | </html>
    100   612  100   612    0     0   218k      0 --:--:-- --:--:-- --:--:--  597k
    front_front_1 exited with code 0
    ```

  ![alt text](https://m.popkey.co/cc2574/QbLg_f-maxage-0.gif "Happy days!")
