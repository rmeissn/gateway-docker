# Docker Mozilla WebThings Gateway

[Docker image](https://github.com/mozilla-iot/gateway-docker) based on Debian Buster for running the [Mozilla WebThings Gateway](https://github.com/mozilla-iot/gateway).

## Compatibility

While the gateway doesn't necessarily require full local network access, some add-ons may. Therefore, it is best to run with the `--network="host"` flag. Currently, this flag will not work when using [Docker for Mac](https://docs.docker.com/docker-for-mac/) or [Docker for Windows](https://docs.docker.com/docker-for-windows/) due to [this](https://github.com/docker/for-mac/issues/68) and [this](https://github.com/docker/for-win/issues/543).

## Usage

### AMD64

* On Linux:

    ```shell
    docker run \
        -d \
        -e TZ=America/Los_Angeles \
        -v /path/to/shared/data:/home/node/.mozilla-iot \
        --network="host" \
        --name webthings-gateway \
        mozillaiot/gateway:latest
    ```

* On Windows or macOS:

    ```shell
    docker run \
        -d \
        -p 8080:8080 \
        -p 4443:4443 \
        -e TZ=America/Los_Angeles \
        -v /path/to/shared/data:/home/node/.mozilla-iot \
        --name webthings-gateway \
        mozillaiot/gateway:latest
    ```

### ARM (e.g. Raspberry Pi)

Tested on Raspberry Pi 3 Model B/B+:

```shell
docker run \
    -d \
    -e TZ=America/Los_Angeles \
    -v /path/to/shared/data:/home/node/.mozilla-iot \
    --network="host" \
    --name webthings-gateway \
    mozillaiot/gateway:arm
```

### Parameters

* `-d` - Run in daemon mode (in the background)
* `-e TZ=America/Los_Angeles` - Set the time zone to `America/Los_Angeles`. The list of names can be found [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List).
* `-v /path/to/shared/data:/home/node/.mozilla-iot` - Change `/path/to/shared/data` to some local path. We are mounting a directory on the host to the container in order to store the persistent "user profile" data, e.g. add-ons, logs, configuration data, etc.
* `--network="host"` - Shares host networking with container (**highly recommended**, needed by some addons, -p is ignored if this option is used)
* `-p 8080:8080` / `-p 4443:4443` - Forward necessary ports to the container (ignored if `--network="host"` is present)
* `--name webthings-gateway` - Name of the container

## Changing ports in `--network="host"` mode
Create a file `local.json` and map it to `/path/to/shared/data/config/local.json`. Contents of the file:

```json
{
  "ports": {
    "https": 8081,
    "http": 8080
  }
}
```
Edit the ports as you like.

## Using docker-compose

***NOTE:*** The present docker-compose config file pulls `mozillaiot/gateway:latest`. If you would like to use the ARM version, then change the image field to `mozillaiot/gateway:arm`.

``` docker-compose up -d```

## Connecting

After running the container, you can connect to it at:
http://&lt;host-ip-address&gt;:8080

## Building

If you'd like to build an image yourself, run the following:

```shell
git clone https://github.com/mozilla-iot/gateway-docker
cd gateway-docker
docker build -t gateway .
docker run \
    -d \
    -e TZ=America/Los_Angeles \
    -v /path/to/shared/data:/home/node/.mozilla-iot \
    --network="host" \
    --name webthings-gateway \
    gateway
```

You can add the following build args:
* `--build-arg "gateway_url=https://github.com/<your-fork>/gateway"`
* `--build-arg "gateway_branch=<your-branch>"`
* `--build-arg "gateway_addon_version=<your-version>"`

## Notes

* If you need to use Zigbee, Z-Wave, or some other add-on which requires physically attached hardware, you will have to share your device into your container, e.g. `--device /dev/ttyACM0:/dev/ttyACM0`.
