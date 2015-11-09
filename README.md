Kubernetes Machine (`kmachine`)
===============================

Kmachine lets you create Docker hosts on your computer, on cloud providers, and
inside your own data center. It creates servers, installs Docker on them, then
configures the Docker client to talk to them.

Kmachine differs from Docker machine by also setting up a Kubernetes standalone system.
Each component of Kubernetes are started as Docker containers. Kmachine returns the configuration
information necessary for `kubectl` to communicate to this remote k8s endpoint.

Kmachine is a work in progress but already does a lot.

Kmachine can be used to create your Docker hosts, the functionalities of `docker-machine` are preserved.

It works a bit like this:

Digital Ocean
-------------

```console
$ docker-machine create -d digitalocean skippbox
Running pre-create checks...
Creating machine...
Waiting for machine to be running, this may take a few minutes...
Machine is running, waiting for SSH to be available...
Detecting operating system of created instance...
Provisioning created instance...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
To see how to connect Docker to this machine, run: docker-machine env skippbox
```

Once the machine is created, just like with `docker-machine` you can get some environment variables that will allow you to use it easily.
Note that with `kmachine`, we return some instructions that `kubectl` can use to define a new k8s context.

```console
$ docker-machine env skippbox
kubectl config set-cluster skippbox --server=https://159.203.140.251:6443 --insecure-skip-tls-verify=false
kubectl config set-cluster skippbox --server=https://159.203.140.251:6443 --certificate-authority=/Users/sebgoa/.docker/machine/machines/skippbox/ca.pem
kubectl config set-credentials kuser --token=IHqC9JMhWOHnFFlr2cO3tBpGGAXzDqYx
kubectl config set-context skippbox --user=kuser --cluster=skippbox
kubectl config use-context skippbox
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://159.203.140.251:2376"
export DOCKER_CERT_PATH="/Users/sebgoa/.docker/machine/machines/skippbox"
export DOCKER_MACHINE_NAME="skippbox"
# Run this command to configure your shell: 
# eval "$(docker-machine env skippbox)"
```

The authentication token is auto-generated, and the certificates are put in place for proper TLS communication with the k8s API server.
Once this new context is set you see it with `kubectl config view`

```console
$ eval "$(docker-machine env skippbox)"
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/sebgoa/.docker/machine/machines/skippbox/ca.pem
    server: https://159.203.140.251:6443
  name: skippbox
contexts:
- context:
    cluster: skippbox
    user: kuser
  name: skippbox
current-context: skippbox
kind: Config
preferences: {}
users:
- name: kuser
  user:
    token: IHqC9JMhWOHnFFlr2cO3tBpGGAXzDqYx
```

Note that since the functionalities of `docker-machine` are preserved you will have an easy into your kmachine via SSH:

```console
$ docker-machine ssh skippbox
Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.13.0-57-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Sat Nov  7 11:08:54 EST 2015

  System load:  0.86              Processes:              72
  Usage of /:   9.2% of 19.56GB   Users logged in:        0
  Memory usage: 18%               IP address for eth0:    159.203.140.251
  Swap usage:   0%                IP address for docker0: 172.17.0.1

  Graph this data and manage this system at:
    https://landscape.canonical.com/

root@skippbox:~# docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS              PORTS               NAMES
3ed51c981f54        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube scheduler"   22 minutes ago      Up 22 minutes                           k8s_scheduler.6346e99c_kubernetes123-127.0.0.1_default_6fde80142812f40cf848367ebaeef544_35e95afb
305cb84717c8        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube proxy --m"   22 minutes ago      Up 22 minutes                           k8s_proxy.7d0a1297_kubernetes123-127.0.0.1_default_6fde80142812f40cf848367ebaeef544_0d5cb791
6b23bfaee4b8        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube apiserver"   22 minutes ago      Up 22 minutes                           k8s_apiserver.f4a937b5_kubernetes123-127.0.0.1_default_6fde80142812f40cf848367ebaeef544_71cab2d1
f45185c25100        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube controlle"   22 minutes ago      Up 22 minutes                           k8s_controller-manager.7a35f0b6_kubernetes123-127.0.0.1_default_6fde80142812f40cf848367ebaeef544_40b06c2e
94c9bff59658        b.gcr.io/kuar/etcd:2.1.1                    "/etcd --data-dir=/va"   22 minutes ago      Up 22 minutes                           k8s_etcd.92bf0224_kubernetes123-127.0.0.1_default_6fde80142812f40cf848367ebaeef544_81ff2e71
c626b5467b14        gcr.io/google_containers/pause:0.8.0        "/pause"                 22 minutes ago      Up 22 minutes                           k8s_POD.e4cc795_kubernetes123-127.0.0.1_default_6fde80142812f40cf848367ebaeef544_5079623e
8b7eee9ead53        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube kubelet -"   22 minutes ago      Up 22 minutes                           master
root@skippbox:~# 
```

VirtualBox
----------

```console
$ ./docker-machine create -d virtualbox --virtualbox-boot2docker-url=https://github.com/skippbox/boot2k8s/releases/download/v1.0.3-rc.1/boot2k8s.iso foobar
Running pre-create checks...
Creating machine...
Waiting for machine to be running, this may take a few minutes...
Machine is running, waiting for SSH to be available...
Detecting operating system of created instance...
Provisioning created instance...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Configuring kubernetes...
Copying certs to the remote system...
To see how to connect Docker to this machine, run: ./docker-machine env foobar
```

Update your local configuration and you are ready to use Kubernetes.

```console
./docker-machine env foobar
kubectl config set-cluster foobar --server=https://192.168.99.111:6443 --insecure-skip-tls-verify=false
kubectl config set-cluster foobar --server=https://192.168.99.111:6443 --certificate-authority=/Users/sebastiengoasguen/.docker/machine/machines/foobar/ca.pem
kubectl config set-credentials kuser --token=IeDQCuzjHkmq70wqa35u6vygAVdTB6Ml
kubectl config set-context foobar --user=kuser --cluster=foobar
kubectl config use-context foobar
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.111:2376"
export DOCKER_CERT_PATH="/Users/sebastiengoasguen/.docker/machine/machines/foobar"
export DOCKER_MACHINE_NAME="foobar"
# Run this command to configure your shell: 
# eval "$(./docker-machine env foobar)"
```

Since it is fully compatible with `docker-machine`, things like getting to your machine via SSH work:

```console
$ ./docker-machine ssh foobar
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.9.0, build master : d81f2f4 - Thu Nov  5 20:40:42 UTC 2015
Docker version 1.9.0, build 76d6bc9
docker@foobar:~$ docker ps
CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS              PORTS               NAMES
6c9b9b42f336        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube apiserver"   15 minutes ago      Up 15 minutes                           k8s_apiserver.18e5aff9_foobar-foobar_default_89de857e00cf225431816ef4afd91195_8328b012
abe5dcbc3dd1        b.gcr.io/kuar/etcd:2.1.1                    "/etcd --data-dir=/va"   15 minutes ago      Up 15 minutes                           k8s_etcd.92bf0224_foobar-foobar_default_89de857e00cf225431816ef4afd91195_21051d04
7b01bf31f701        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube scheduler"   15 minutes ago      Up 15 minutes                           k8s_scheduler.6346e99c_foobar-foobar_default_89de857e00cf225431816ef4afd91195_4a793b67
26f6f00f79d4        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube proxy --m"   15 minutes ago      Up 15 minutes                           k8s_proxy.7d0a1297_foobar-foobar_default_89de857e00cf225431816ef4afd91195_2165ac73
a7ceff86eaae        gcr.io/google_containers/hyperkube:v1.0.3   "/hyperkube controlle"   15 minutes ago      Up 15 minutes                           k8s_controller-manager.7a35f0b6_foobar-foobar_default_89de857e00cf225431816ef4afd91195_daa5ca02
c00bfdf7fcfa        gcr.io/google_containers/pause:0.8.0        "/pause"                 18 minutes ago      Up 18 minutes                           k8s_POD.e4cc795_foobar-foobar_default_89de857e00cf225431816ef4afd91195_a4f67919
```

Documentation
-------------

kmachine is currently rebased on docker-machine 0.5.0 (latest) and all drivers are used the same way.
The binaries are called `docker-machin` but we will get them renamed.

Build
-----

The build mechanism is identical to docker-machine, you need a Docker host and then:

```console
$ export USE_CONTAINER=true
$ make cross
```

or specify your OS and ARCH

```console
$ TARGET_OS=darwin TARGET_ARCH="amd64" make
```

The binaries will be in the `build` directory and you will be able to test them with:

```console
$ PATH=$PWD:$PATH ./kmachine create -d digitalocean foobar
```

Support
-------

If you experience problems with `kmachine` or want to suggest improvements please file an [issue](https://github.com/skippbox/machine/issues).

