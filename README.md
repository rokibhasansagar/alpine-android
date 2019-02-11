[![build status][251]][232] [![commit][255]][231] [![version:x86_64][256]][235] [![size:x86_64][257]][235]

## [Alpine-Android][234]
#### Container for Alpine Linux + OpenJDK8 + Android + Cordova
---

This [image][233] containerizes the [Android][136] build
environment, along with [Gradle][137] and [OpenJDK8][135], as well as
[Cordova][138] for building hybrid apps, with its [NPM][139] dependencies.

Based on [Alpine Linux][131] from my [alpine-openjdk8][132] image with
the [s6][133] init system [overlayed][134] in it.

The image is tagged respectively for the following architectures,
* ~~**armhf**~~
* **x86_64** (retagged as the `latest` )

~~**armhf** builds have embedded binfmt_misc support and contain the~~
~~[qemu-user-static][105] binary that allows for running it also inside~~
~~an x64 environment that has it.~~

---
#### Get the Image
---

Pull the image for your architecture it's already available from
Docker Hub.

```
# make pull
docker pull woahbase/alpine-android:x86_64
```

---
#### Run
---

If you want to run images for other architectures, you will need
to have binfmt support configured for your machine. [**multiarch**][104],
has made it easy for us containing that into a docker container.

```
# make regbinfmt
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

Without the above, you can still run the image that is made for your
architecture, e.g for an x86_64 machine..

This images already has a user `circleci` configured to drop
privileges to the passed `PUID`/`PGID` which is ideal if its used
to run in non-root mode. That way you only need to specify the
values at runtime and pass the `-u circleci` if need be. (run `id`
in your terminal to see your own `PUID`/`PGID` values.)

Before you run..

* Mount the project directory (where `build.gradle` or
  `package.json` is) at `/home/circleci/project`. Mounts `PWD` by
  default.

* Builds run under the user `circleci`.

* Optionally, if you want to cache the jars/packages downloaded by
  gradle, so that they're downloaded once, and reused in later
  builds, bind mount the user home directory (`/home/circleci`)
  somewhere in your local. The packages get cached inside the
  `/home/circleci/.gradle` folder.

Running `make` gets a shell.

```
# make
docker run --rm -it \
  --name docker_android --hostname android \
  -e PGID=1000 -e PUID=1000 \
  -c 512 -m 3072m \
  -v $PWD:/home/circleci/project \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/hosts:/etc/hosts:ro \
  rokibhasansagar/alpine-android:x86_64
```

The usual android stuff. e.g dev build a project with

```
docker run --rm -it \
  --name docker_android --hostname android \
  -e PGID=1000 -e PUID=1000 \
  -c 512 -m 1024m \
  -v $PWD:/home/circleci/project \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/hosts:/etc/hosts:ro \
  rokibhasansagar/alpine-android:x86_64 \
  -ec "gradle assembleDebug"
```

also for Cordova projects, you can dev build a project with

```
docker run --rm -it \
  --name docker_android --hostname android \
  -e PGID=1000 -e PUID=1000 \
  -c 512 -m 3072m \
  -v $PWD:/home/circleci/project \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/hosts:/etc/hosts:ro \
  rokibhasansagar/alpine-android:x86_64 \
  -ec "npm install && npm run <your build target>"
```

Stop the container with a timeout, (defaults to 2 seconds)

```
# make stop
docker stop -t 2 docker_android
```

Removes the container, (always better to stop it first and `-f`
only when needed most)

```
# make rm
docker rm -f docker_android
```

Restart the container with

```
# make restart
docker restart docker_android
```

---
#### Shell access
---

Get a shell inside a already running container,

```
# make shell
docker exec -it docker_android /bin/bash
```

set user or login as root,

```
# make rshell
docker exec -u root -it docker_android /bin/bash
```

To check logs of a running container in real time

```
# make logs
docker logs -f docker_android
```

---
### Development
---

If you have the repository access, you can clone and
build the image yourself for your own system, and can push after.

---
#### Setup
---

Before you clone the [repo][231], you must have [Git][101], [GNU make][102],
and [Docker][103] setup on the machine.

```
git clone https://github.com/woahbase/alpine-android
cd alpine-android
```
You can always skip installing **make** but you will have to
type the whole docker commands then instead of using the sweet
make targets.

---
#### Build
---

You need to have binfmt_misc configured in your system to be able
to build images for other architectures.

Otherwise to locally build the image for your system.
[`ARCH` defaults to `x86_64`, need to be explicit when building
for other architectures.]

```
# make ARCH=x86_64 build
# sets up binfmt if not x86_64
docker build --rm --compress --force-rm \
  --no-cache=true --pull \
  -f ./Dockerfile_x86_64 \
  --build-arg ARCH=x86_64 \
  --build-arg DOCKERSRC=alpine-openjdk8 \
  --build-arg PGID=1000 \
  --build-arg PUID=1000 \
  --build-arg USERNAME=rokibhasansagar \
  --build-arg GRADLE_VERSION=$(GRADLE_VERSION) \
  --build-arg NPM_VERSION=$(NPM_VERSION) \
  --build-arg SDK_TARGET=$(SDK_TARGET) \
  --build-arg SDK_TOOLS_VERSION=$(SDK_TOOLS_VERSION) \
  --build-arg SDK_API_VERSION=$(SDK_API_VERSION) \
  -t rokibhasansagar/alpine-android:x86_64 \
  .
```

To check if its working..

```
# make ARCH=x86_64 test
docker run --rm -it \
  --name docker_android --hostname android \
  -e PGID=1000 -e PUID=1000 \
  rokibhasansagar/alpine-android:x86_64 \
  -ec 'sdkmanager --version; \
    sdkmanager --list | sed -e "/Available Packages/q" ; \
    gradle -version; \
    adb version; \
    npm --version; \
    cordova --version --no-telemetry';
```

And finally, if you have push access,

```
# make ARCH=x86_64 push
docker push rokibhasansagar/alpine-android:x86_64
```

---
### Maintenance
---

Sources at [Github][106]. Built at [Travis-CI.org][107] (armhf / x64 builds). Images at [Docker hub][108]. Metadata at [Microbadger][109].

Maintained by [WOAHBase][204].

[101]: https://git-scm.com
[102]: https://www.gnu.org/software/make/
[103]: https://www.docker.com
[104]: https://hub.docker.com/r/multiarch/qemu-user-static/
[105]: https://github.com/multiarch/qemu-user-static/releases/
[106]: https://github.com/
[107]: https://travis-ci.org/
[108]: https://hub.docker.com/
[109]: https://microbadger.com/

[131]: https://alpinelinux.org/
[132]: https://hub.docker.com/r/woahbase/alpine-openjdk8
[133]: https://skarnet.org/software/s6/
[134]: https://github.com/just-containers/s6-overlay
[135]: http://openjdk.java.net/
[136]: https://developer.android.com/studio/#command-tools
[137]: https://gradle.org/
[138]: https://cordova.apache.org/
[139]: https://www.npmjs.com/

[201]: https://github.com/woahbase
[202]: https://travis-ci.org/woahbase/
[203]: https://hub.docker.com/u/woahbase
[204]: https://woahbase.online/

[231]: https://github.com/woahbase/alpine-android
[232]: https://travis-ci.org/woahbase/alpine-android
[233]: https://hub.docker.com/r/woahbase/alpine-android
[234]: https://woahbase.online/#/images/alpine-android
[235]: https://microbadger.com/images/woahbase/alpine-android:x86_64

[251]: https://travis-ci.org/woahbase/alpine-android.svg?branch=master

[255]: https://images.microbadger.com/badges/commit/woahbase/alpine-android.svg

[256]: https://images.microbadger.com/badges/version/woahbase/alpine-android:x86_64.svg
[257]: https://images.microbadger.com/badges/image/woahbase/alpine-android:x86_64.svg
