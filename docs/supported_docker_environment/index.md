# General Container runtime requirements

* | run Testcontainers-based tests
  * requirements
    * ⚠️Docker-API compatible container runtime⚠️
      * install Docker locally, OR
      * [Testcontainers Cloud](https://www.testcontainers.cloud/) , OR
      * [Testcontainers Desktop](https://testcontainers.com/desktop/)

## Docker Desktop
* 💡AUTOMATICALLY detected💡
  * == ❌NO require ADDITIONAL configuration❌
* Reason:🧠used | test development Testcontainers🧠

## Colima
* TODO:
In order to run testcontainers against [colima](https://github.com/abiosoft/colima) the env vars below should be set

```bash
colima start --network-address
export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE=/var/run/docker.sock
export TESTCONTAINERS_HOST_OVERRIDE=$(colima ls -j | jq -r '.address')
export DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock"
```

## Podman

In order to run testcontainers against [podman](https://podman.io/) the env vars bellow should be set

MacOS:

```bash
{% raw %}
export DOCKER_HOST=unix://$(podman machine inspect --format '{{.ConnectionInfo.PodmanSocket.Path}}')
export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE=/var/run/docker.sock
{% endraw %}
```

Linux:

```bash
export DOCKER_HOST=unix://${XDG_RUNTIME_DIR}/podman/podman.sock
```

If you're running Podman in rootless mode, ensure to include the following line to disable Ryuk:

```bash
export TESTCONTAINERS_RYUK_DISABLED=true
```

!!! note
    Previous to version 1.19.0, `export TESTCONTAINERS_RYUK_PRIVILEGED=true`
    was required for rootful mode. Starting with 1.19.0, this is no longer required.

## Rancher Desktop

In order to run testcontainers against [Rancher Desktop](https://rancherdesktop.io/) the env vars below should be set.

If you're running Rancher Desktop as an administrator in a MacOS (M1) machine:

Using QEMU emulation

```bash
export TESTCONTAINERS_HOST_OVERRIDE=$(rdctl shell ip a show rd0 | awk '/inet / {sub("/.*",""); print $2}')
```

Using VZ emulation

```bash
export TESTCONTAINERS_HOST_OVERRIDE=$(rdctl shell ip a show vznat | awk '/inet / {sub("/.*",""); print $2}')
```

If you're not running Rancher Desktop as an administrator in a MacOS (M1) machine:

Using VZ emulation

```bash
export DOCKER_HOST=unix://$HOME/.rd/docker.sock
export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE=/var/run/docker.sock
export TESTCONTAINERS_HOST_OVERRIDE=$(rdctl shell ip a show vznat | awk '/inet / {sub("/.*",""); print $2}')
```

## Docker environment discovery

* strategies (in PRIORITY order) / 
  * 👀Testcontainers follow  -- to connect to a -- Docker daemon👀
    * environment variables
      * `DOCKER_HOST`
        * by default, https://localhost:2376
      * `DOCKER_TLS_VERIFY`
        * by default, 1
      * `DOCKER_CERT_PATH`
        * by default, ~/.docker
    * installed FIRST found [Docker Machine](https://docs.docker.com/retired/#docker-machine)
      * requirements
        * Docker Machine needs to be | PATH
      * if you want to disable it -> | ".testcontainers.properties"
        ```properties
        testcontainers.docker.client.strategy=org.testcontainers.dockerclient.UnixSocketClientProviderStrategy
        ```  

* [run your tests | container](continuous_integration/dind_patterns.md)

## Docker registry authentication

Testcontainers will try to authenticate to registries with supplied config using the following strategies in order:

* Environment variables:
    * `DOCKER_AUTH_CONFIG`
* Docker config
	* At location specified in `DOCKER_CONFIG` or at `{HOME}/.docker/config.json`

## [Customizing Docker host detection](../features/configuration.md#customizing-docker-host-detection)
