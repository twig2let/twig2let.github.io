### Create a MongoDB Container

1. Create a data directory within in your project space
    ```
    $ mkdir -p ./mongo/data/
    ```

2. Download/Start the container and mount the local data volume inside
    ```
    $ docker run --rm --name mongo-wang -v $(pwd)/mongo/data:/data/db -d mongo

    **--rm** - Container is removed when it exits or when the daemon exits
    **--name** - Human friendly container ref
    ```