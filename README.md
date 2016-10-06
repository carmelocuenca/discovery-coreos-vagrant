La primera herramienta para el descubrimiento es [confd](https://github.com/kelseyhightower/confd), una utilidad que interroga un repositorio de claves valor y que tiene un lenguaje de plantillas para actualizar un fichero, también tiene acciones para realizar cuando la plantilla cambia.
Como en CoreOS no tiene gestor de paquetes, el fichero confd.dockerfile define la imagen de la utilidad. Para construir la imagen

```
core@core-01 ~ $ docker build -f share/confd.dockerfile -t confd .
```

Para comprobar el funcionamiento

```
core@core-01 ~ $ docker run --rm -v /tmp/:/tmp -v $PWD/share/etc/confd:/etc/confd -e COREOS_PRIVATE_IPV4=$COREOS_PRIVATE_IPV4 confd
```

Y desde otro shell en core-01

```
core@core-01 ~ $ etcdctl set /myapp/database/url db.example.com
core@core-01 ~ $ etcdctl set /myapp/database/user rob
core@core-01 ~ $ cat /tmp/myconfig.conf
```

Para esta demo resulta importante compartir el directorio ```/tmp``` del host para comprobar que el fichero ha sido actualizado (si no, se actualizaría únicamente en el contenedor).

Instalado también un plugin para realizar scp a las máquinas vagrant

```
$ vagrant plugin install vagrant-scp
```

# Unit para PostgreSQL

El fichero postgresql contiene una vesión inicial.

Para habilitarlo
```
core@core-01 ~ $ sudo cp postgresql.service /etc/systemd/system/
core@core-01 ~ $ sudo systemctl enable postgresql.service
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /etc/systemd/system/postgresql.service.
```

Para lanzarlo

```
core@core-01 ~ $ sudo systemctl start postgresql.service
```

Para ver por donde va
```
core@core-01 ~ $ sudo journalctl -f -u postgresql.service
```

Para conectarte
```
core@core-01 ~ $ docker run -it --rm --link some-postgres:postgres postgres psql -h postgres -U postgres
```

# Unit para Ruby on Rails



# Inicialición de servicios
```
core@core-01 ~ $ sudo cp share/units/*.service /etc/systemd/system
core@core-01 ~ $ sudo systemctl start nginx.service
```

# 06 de octubre
Creadas 4 unidades de servicio


# CoreOS Vagrant

This repo provides a template Vagrantfile to create a CoreOS virtual machine using the VirtualBox software hypervisor.
After setup is complete you will have a single CoreOS virtual machine running on your local machine.

## Contact
IRC: #coreos on freenode.org

Mailing list: [coreos-dev](https://groups.google.com/forum/#!forum/coreos-dev)

## Streamlined setup

1) Install dependencies

* [VirtualBox][virtualbox] 4.3.10 or greater.
* [Vagrant][vagrant] 1.6.3 or greater.

2) Clone this project and get it running!

```
git clone https://github.com/coreos/coreos-vagrant/
cd coreos-vagrant
```

3) Startup and SSH

There are two "providers" for Vagrant with slightly different instructions.
Follow one of the following two options:

**VirtualBox Provider**

The VirtualBox provider is the default Vagrant provider. Use this if you are unsure.

```
vagrant up
vagrant ssh
```

**VMware Provider**

The VMware provider is a commercial addon from Hashicorp that offers better stability and speed.
If you use this provider follow these instructions.

VMware Fusion:
```
vagrant up --provider vmware_fusion
vagrant ssh
```

VMware Workstation:
```
vagrant up --provider vmware_workstation
vagrant ssh
```

``vagrant up`` triggers vagrant to download the CoreOS image (if necessary) and (re)launch the instance

``vagrant ssh`` connects you to the virtual machine.
Configuration is stored in the directory so you can always return to this machine by executing vagrant ssh from the directory where the Vagrantfile was located.

4) Get started [using CoreOS][using-coreos]

[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/

#### Shared Folder Setup

There is optional shared folder setup.
You can try it out by adding a section to your Vagrantfile like this.

```
config.vm.network "private_network", ip: "172.17.8.150"
config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true,  :mount_options   => ['nolock,vers=3,udp']
```

After a 'vagrant reload' you will be prompted for your local machine password.

#### Provisioning with user-data

The Vagrantfile will provision your CoreOS VM(s) with [coreos-cloudinit][coreos-cloudinit] if a `user-data` file is found in the project directory.
coreos-cloudinit simplifies the provisioning process through the use of a script or cloud-config document.

To get started, copy `user-data.sample` to `user-data` and make any necessary modifications.
Check out the [coreos-cloudinit documentation][coreos-cloudinit] to learn about the available features.

[coreos-cloudinit]: https://github.com/coreos/coreos-cloudinit

#### Configuration

The Vagrantfile will parse a `config.rb` file containing a set of options used to configure your CoreOS cluster.
See `config.rb.sample` for more information.

## Cluster Setup

Launching a CoreOS cluster on Vagrant is as simple as configuring `$num_instances` in a `config.rb` file to 3 (or more!) and running `vagrant up`.
Make sure you provide a fresh discovery URL in your `user-data` if you wish to bootstrap etcd in your cluster.

## New Box Versions

CoreOS is a rolling release distribution and versions that are out of date will automatically update.
If you want to start from the most up to date version you will need to make sure that you have the latest box file of CoreOS. You can do this by running
```
vagrant box update
```


## Docker Forwarding

By setting the `$expose_docker_tcp` configuration value you can forward a local TCP port to docker on
each CoreOS machine that you launch. The first machine will be available on the port that you specify
and each additional machine will increment the port by 1.

Follow the [Enable Remote API instructions][coreos-enabling-port-forwarding] to get the CoreOS VM setup to work with port forwarding.

[coreos-enabling-port-forwarding]: https://coreos.com/docs/launching-containers/building/customizing-docker/#enable-the-remote-api-on-a-new-socket

Then you can then use the `docker` command from your local shell by setting `DOCKER_HOST`:

    export DOCKER_HOST=tcp://localhost:2375
