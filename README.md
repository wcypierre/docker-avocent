# Avocent vKVM dockerized

## About

This docker image is based on [zheng1/docker-avocent](https://github.com/zheng1/docker-avocent) and the original [DomiStyle/docker-idrac6](https://github.com/zheng1/docker-avocent) repo for dockerizing Dell idrac KVMs.

This fork has been modified to work for Avocent KVM 4.1.1.54 that ships with **Avocent MergePoint EMS AST2300** on [Gigabyte GA-6PXSV4](https://www.gigabyte.com/Enterprise/Server-Motherboard/GA-6PXSV4-rev-10#Support-Firmware) motherboards, but will probably work for any mobo in that series.

Modifications made to get this working:

* [Original command modifications](https://github.com/DomiStyle/docker-idrac6/commit/96673ba3a02335120952fffaafd75300d964238f) made by [zheng1](https://github.com/zheng1) to get avocent client executing
* Removing TLSv1.1 from disabled algorithms for java since this is what the avocent client uses
* Add `user.home` java arg so certs have somewhere to download
* Downgrading java to java 8 due to client using a deprecated/removed AWT function `java.lang.NoSuchMethodError: 'java.awt.peer.ComponentPeer java.awt.Window.getPeer()` that caused keyboard input to fail to work

# Disclaimer

Due to the above changes (outdated java, TLSv1.1 enabled) this **should only be used on internal, secured networks. Do not expose this container to the internet!** It is highly insecure and should only be used when the container and BMC platform are located an an internal network.

# Usage

`IDRAC_HOST` should be the IP address of the MergePoint EMS interface.

See the docker-compose [here](https://github.com/DomiStyle/docker-idrac6/blob/master/docker-compose.yml) or use this command:
```
docker run -d \
  -p 5800:5800 \
  -p 5900:5900 \
  -e IDRAC_HOST=192.168.0.XXX \
  -e IDRAC_USER=admin \
  -e IDRAC_PASSWORD=password \
  foxxmd/avocent-vkvm
```
The web interface will be available on port 5800 while the VNC server can be accessed on 5900. Startup might take a few seconds while the Java libraries are downloaded. You can add a volume on /app if you would like to cache them.

## Configuration

| Variable       | Description                                  | Required |
|----------------|----------------------------------------------|----------|
|`IDRAC_HOST`| IP address of the MergePoint EMS interface. Make sure your instance is reachable with https://<IDRAC_HOST>. See IDRAC_PORT for using custom ports. HTTPS is always used. | Yes |
|`IDRAC_USER`| Username for your iDRAC instance. | Yes |
|`IDRAC_PASSWORD`| Password for your iDRAC instance. | Yes |
|`IDRAC_PORT`| The optional port for the web interface. (443 by default) | No |
|`IDRAC_KEYCODE_HACK`| If you have issues with keyboard input, try setting this to ``true``. See [here](https://github.com/anchor/idrac-kvm-keyboard-fix) for more infos. | No |
|`VIRTUAL_MEDIA`| Filename of iso located within /vmedia to automount | No |

**For advanced configuration options please take a look [here](https://github.com/jlesage/docker-baseimage-gui#environment-variables).**

## Volumes

| Path       | Description                                  | Required |
|------------|----------------------------------------------|----------|
|`/app`| Libraries downloaded from your iDRAC instance will be stored here. Add a volume to cache those files for a faster container startup. | No |
|`/vmedia`| Can be used to allow virtual media to be mounted. | No |
|`/screenshots`| Screenshots taken from the virtual console can be stored here. | No |

Make sure the container user has read & write permission to these folders on the host. [More info here](https://github.com/jlesage/docker-baseimage-gui#usergroup-ids).

## Issues & limitations

[See original repository](https://github.com/DomiStyle/docker-idrac6#issues--limitations)