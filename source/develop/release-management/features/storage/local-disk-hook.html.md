---
title: Local disk hook
category: feature
authors: nsoffer
feature_name: Local disk hook
feature_modules: engine,vdsm
feature_status: Design
---

# Local disk hook


## Overview

This hook add the ability to use fast local storage on instead of shared
storage, while using shared storage for managing VM templates.  In the
current system, a user have to choose between fast local storage sharing
nothing with other hosts, or shared storage, where everything is shared
between the hosts and fast local storage cannot be used. This feature
try to mix both local and shared storage.


## Owner

- Name: [Nir Soffer](https://github.com/nirs)
- Email: <nsoffer@redhat.com>


## Detailed Description


### How it works

A user will create a VM normally on shared storage of any type. To use
the VM with local storage, the user will have to pin the VM to a certain
host, and enable the localdisk hook.

When starting the VM on the pinned host, the localdisk hook will copy
the VM disks from shared storage into the host local storage, and modify
the VM XML to use the local copy of the disk.

The original disk may be a single volume or a chain of volumes based on
a template. The local copy is a raw preallocated volume using a LVM
logical volume on the special "ovirt-local" volume group.

We assume that local disks content is not valuable, and it is OK to
loose the content any time, and create a new copy from the template.
Users that care about their VM disks should use shared storage instead.

Most operations will be disabled on a VM using the localdisk hook. For
example, the VM cannot be migrated to another host, cannot create
snapshots, copy the local disk to other storage, etc.

To change storage on a VM using local storage, the localdisk hook must
be disabled.

When the VM is stopped, the local disk will be deleted from the local
host storage. If this option is disabled, the user will be responsible
for deleting unused disks.


### Open issues

The system administrator will be responsible for creating the host
"ovirt-local" volume group and extending it with new devices if needed.

If a VM stopped while vdsm is not running, the local copy of the vm disk
will not be deleted. This can be fixed by deleting unused local disks
when vdsm starts, but we don't have yet a startup hook.
