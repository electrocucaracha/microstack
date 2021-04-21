# MicroStack lab
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![GitHub Super-Linter](https://github.com/electrocucaracha/microstack/workflows/Lint%20Code%20Base/badge.svg)](https://github.com/marketplace/actions/super-linter)

## Summary

[MicroStack][1] project provides a full Zero-ops OpenStack deployment in a
single snap package. This approach is suitable for edge and IoT deployments.
This project collects the minimal instructions required for deploying a
Multi-node OpenStack cluster.

## Setup

This project uses [Vagrant tool][2] for provisioning Virtual Machines
automatically. It's highly recommended to use the  `setup.sh` script
of the [bootstrap-vagrant project][3] for installing Vagrant
dependencies and plugins required for its project. The script
supports two Virtualization providers (Libvirt and VirtualBox).

    curl -fsSL http://bit.ly/initVagrant | PROVIDER=libvirt bash

Once Vagrant is installed, it's possible to deploy an OpenStack
cluster with the following instruction:

    vagrant up

For adding more compute nodes to the existing cluster execute the
following instruction:

    vagrant up compute1

[1]: https://microstack.run/
[2]: https://www.vagrantup.com/
[3]: https://github.com/electrocucaracha/bootstrap-vagrant
