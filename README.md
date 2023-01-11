<!--
Copyright (C) 2023 Robert Wimmer
SPDX-License-Identifier: GPL-3.0-or-later
-->

ansible-role-longhorn_kubernetes
================================

This Ansible role installs [Longhorn](https://longhorn.io/) cloud native distributed block storage on a Kubernetes cluster. Behind the doors it uses the official [Helm chart](https://charts.longhorn.io). Currently procedures like installing, upgrading and deleting the `Longhorn` deployment are supported.

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `0.1.0+1.4.0` means this is release `0.1.0` of this role and it contains Longhorn chart version `1.4.0` (which normally is the same as the Longhorn version itself). If the role itself changes `X.Y.Z` before `+` will increase. If the Longhorn chart version changes `X.Y.Z` after `+` will increase too. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Longhorn release.

Requirements
------------

You need to have [Helm 3](https://helm.sh/) binary installed on that host where `ansible-playbook` is executed or on that host where you delegated the playbooks to (e.g. by using `longhorn_delegate_to` variable). You can either

- use your favorite package manager if your distribution includes `helm` in its repository (for Archlinux use `sudo pacman -S helm` e.g.)
- or use one of the Ansible `Helm` roles (e.g. [helm](https://galaxy.ansible.com/gantsign/helm) - which gets also installed if you use `ansible-galaxy role install -vr requirements.yml`
- or directly download the binary from [Helm releases)[https://github.com/helm/helm/releases]) and put it into `/usr/local/bin/` directory e.g.

A properly configured `KUBECONFIG` is also needed (which is located at `${HOME}/.kube/config` by default). Normally if `kubectl` works with your K8s cluster then everything should be already fine in this regards.

Additionally the Ansible `kubernetes.core` collection needs to be installed. This can be done by using the `collections.yml` file included in this role: `ansible-galaxy install -r collections.yml`. At least `kubernetes.core >= v2.3.0` is needed.

And of course you need a Kubernetes Cluster ;-)

Additional remarks
------------------

By default Longhorn uses the path `/var/lib/longhorn` on the K8s worker nodes to store data or replicas of data. First it's a good idea to just keep that path if possible. The same is true for the default namespace `longhorn-system`. This might save you from some trouble later. Second since Longhorn doesn’t currently support sharding between the different disks, its recommend using LVM to aggregate all the disks for Longhorn into a single partition, so it can be easily extended in the future. This role doesn't manage LVM but you can use my [LVM role](https://github.com/githubixx/ansible-role-lvm) or any other Ansible LVM role or whatever automation tool you want or just configure everything manually. The Molecule test (see below) contains an example LVM setup.

Role variables
--------------

```yaml
# Helm chart version
longhorn_chart_version: "1.4.0"

# Helm release name
longhorn_release_name: "longhorn"

# Helm repository name
longhorn_repo_name: "longhorn"

# Helm chart name
longhorn_chart_name: "{{ longhorn_repo_name }}/{{ longhorn_release_name }}"

# Helm chart URL
longhorn_chart_url: "https://charts.longhorn.io"

# Kubernetes namespace where Longhorn resources should be installed
longhorn_namespace: "longhorn-system"

# By default all tasks that needs to communicate with the Kubernetes
# cluster are executed on your local host (127.0.0.1). But if that one
# doesn't have direct connection to this cluster or should be executed
# elsewhere this variable can be changed accordingly.
longhorn_delegate_to: "127.0.0.1"

# Shows the "helm" command that was executed if a task uses Helm to
# install, update/upgrade or deletes such a resource.
longhorn_helm_show_commands: false

# Without "longhorn_action" variable defined this role will only render a file
# with all the resources that will be installed or upgraded. The rendered
# file with the resources will be called "template.yml" and will be
# placed in the directory specified below. The default will create a directory
# "longhorn/template" in users "$HOME" directory.
longhorn_template_output_directory: "{{ '~/longhorn/template' | expanduser }}"

# The Ansible group name of the nodes where Longhorn will store data
# or replicas of the data. As the role needs to install a few OS packages
# on that nodes the role needs to know on which hosts the tools/packages
# are needed. E.g. if Longhorn should only be installed on nodes with a
# specific label or taint then only these nodes need the packages installed.
longhorn_nodes: "k8s_longhorn"

# Enable multipathd blacklist. For more information see:
# https://longhorn.io/kb/troubleshooting-volume-with-multipath/
# longhorn_multipathd_blacklist_directory: "/etc/multipath/conf.d"
# longhorn_multipathd_blacklist_directory_perm: "0755"
# longhorn_multipathd_blacklist_file: "10-longhorn.conf"
# longhorn_multipathd_blacklist_file_perm: "0644"
```

Usage
-----

Before you start installing Longhorn you REALLY want to read the [The Longhorn Documentation](https://longhorn.io/docs/1.4.0/)! As data is the most valuable thing you can have you should understand how Longhorn works and don't forget to add backups later ;-). Esp. have a look at the [best practices](https://longhorn.io/docs/1.4.0/best-practices/).

That said: The first thing to do is to check `templates/longhorn_values_default.yml.j2`. This file contains the values/settings for the Longhorn Helm chart that are partly default anyways (just to avoid that someone changes the defaults) or different to the default ones which are located [here](https://github.com/longhorn/longhorn/blob/v1.4.0/chart/values.yaml). All settings can be found in the [Settings Reference](https://longhorn.io/docs/1.4.0/references/settings/).

To use your own values just create a file called `longhorn_values_user.yml.j2` and put it into the `templates` directory. Then this Longhorn role will use that file to render the Helm values. You can use `templates/longhorn_values_default.yml.j2` as a template or just start from scratch. As mentioned above you can modify all settings for the Longhorn Helm chart that are different to the default ones which are located [here](https://github.com/longhorn/longhorn/blob/v1.4.0/chart/values.yaml).

After the values file (`templates/longhorn_values_default.yml.j2` or `templates/longhorn_values_user.yml.j2`) is in place and the `defaults/main.yml` values are checked and maybe adjusted accordingly, the role can be installed. Quite a few tasks need to communicate with the Kubernetes API server or executing [Helm](https://helm.sh/) commands. By default these commands are executed on the host where the `ansible-playbook` gets executed and the current user is used. But you can delegate this kind of tasks to a different host by using `longhorn_delegate_to` variable (see above).

The default action is to just render the Kubernetes resources YAML file after replacing all Jinja2 variables and stuff like that. In the `Example Playbook` section below there is an `Example 2 (assign tag to role)`. The role `githubixx.longhorn_kubernetes` has a tag `role-longhorn-kubernetes` assigned and all further `ansible-playbook` examples below will refer to `Example 2`!

Assuming that the values for the Helm chart should be rendered (nothing will be installed in this case) and the playbook is called `k8s.yml` execute the following command:

```bash
ansible-playbook --tags=role-longhorn-kubernetes k8s.yml
```

To render the template into a different directory change `longhorn_template_output_directory` variable value e.g.:

```bash
ansible-playbook --tags=role-longhorn-kubernetes --extra-vars longhorn_template_output_directory="/tmp/longhorn" k8s.yml
```

If you want to see the `helm` commands and the parameters which were executed in the logs you can also specify `--extra-vars longhorn_helm_show_commands=true`.

One of the final tasks is called `TASK [githubixx.longhorn_kubernetes : Write templates to file]`. This renders the template with the resources that will be created into the directory specified in `longhorn_template_output_directory`. The file will be called `template.yml`. The directory/file will be placed either on your local machine or on the host specified with `longhorn_delegate_to`.

If the rendered output contains everything you need, the role can be installed which finally deploys Longhorn:

```bash
ansible-playbook --tags=role-longhorn-kubernetes --extra-vars longhorn_action=install k8s.yml
```

To check if everything was deployed use the usual `kubectl` commands like `kubectl -n <longhorn_namespace> get pods -o wide`. The first installation will take quite some time if your internet connection isn't the fastest one. Lots of container images need to be downloaded.

As Longhorn gets updates/upgrades every few weeks/months the role also can do upgrades. For updates/upgrades (esp. major upgrades) have a look at `tasks/upgrade.yml` to see what's happening before, during and after the update. Of course you should consult Longhorn's [upgrade guide](https://longhorn.io/docs/1.4.0/deploy/upgrade/) (the link is for upgrading to Longhorn `v1.4.0`) to check for major changes and stuff like that before upgrading. Now is also a good time to check if the backups are in place and if the backups are actually valid ;-)

After consulting Longhorn's [upgrade guide](https://longhorn.io/docs/1.4.0/deploy/upgrade/) you basically only need to change `longhorn_chart_version` variable e.g. from `1.4.0` to `1.4.1` for a patch release or from `1.4.0` to `1.5.0` for a major upgrade. And of course the Helm values need to be adjusted for potential breaking changes (if any are mentioned in the upgrade guide e.g.).

You can also use the upgrade method if you keep the version number and just want to change some Helm values or other settings. But please be aware that changing some of settings might have some serious consequences if you already have volumes deployed! Not all Longhorn settings can be changed just by changing a number or a string. So you really want to consult the [Settings reference](https://longhorn.io/docs/1.4.0/references/settings/) to figure out what might happen if you change this or that setting or what you need to do before you apply a changed setting!

That said to actually do the update/upgrade run

```bash
ansible-playbook --tags=role-longhorn-kubernetes --extra-vars longhorn_action=upgrade k8s.yml
```

And finally if you REALLY want to get rid of Longhorn you can delete all Longhorn resources again. Make sure that you have backups! Nobody will be able to recover/restore data that was deleted and there is no backup! Make sure that all Longhorn volumes are gone before you delete everything! That said you have to provide two variables in order do be able to delete Longhorn resources via this role e.g.:

```bash
ansible-playbook \
  --tags=role-longhorn-kubernetes \
  --extra-vars longhorn_action=delete \
  --extra-vars longhorn_delete=true \
  k8s.yml
```

Longhorn has a [Deleting Confirmation Flag](https://longhorn.io/docs/1.4.0/references/settings/#deleting-confirmation-flag) which is set to `false` by default. In this case Longhorn refuses to be uninstalled. By setting `--extra-vars longhorn_delete=true` the Ansible role will set this flag to `true` and afterwards the Longhorn resources can be deleted by the role. Without `longhorn_delete` variable the role will refuse to finish uninstallation.

Example Playbook
----------------

Example 1 (without role tag):

```yaml
---
- name: Setup Longhorn
  hosts: longhorn
  tasks:
    - name: Include Longhorn role
      ansible.builtin.include_role:
        name: githubixx.longhorn_kubernetes
```

Example 2 (assign tag to role):

```yaml
---
-
  hosts: longhorn
  roles:
    -
      role: githubixx.longhorn_kubernetes
      tags: role-longhorn-kubernetes
```

Testing
-------

This role has a (full blown) Kubernetes test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-longhorn-kubernetes/tree/master/molecule/default).

Afterwards molecule can be executed. The following command will setup a K8s cluster with 8 VMs (Three controller and four Kubernetes worker and one Ansible controller node) and render a template of the resources (default "longhorn_action" see above) that will be created (so Longhorn won't be installed by default):

```bash
molecule converge
```

To actually install `Longhorn` and the required resources use this command:

```bash
molecule converge -- --extra-vars longhorn_action=install
```

Upgrading `Longhorn` or changing parameters:

```bash
molecule converge -- --extra-vars longhorn_action=upgrade
```

Deleting `Longhorn` and its resources:

```bash
molecule converge -- --extra-vars longhorn_action=delete --extra-vars longhorn_delete=true
```

To clean up run

```bash
molecule destroy
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
