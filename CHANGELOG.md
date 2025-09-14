<!--
Copyright (C) 2025 Robert Wimmer
SPDX-License-Identifier: GPL-3.0-or-later
-->

# Changelog

## 0.6.0+1.7.3

This is a major release update of Longhorn. Please read [Longhorn Important Notes](https://longhorn.io/docs/archives/1.7.3/important-notes) before upgrading!

Consider doing a [Longhorn System Backup](https://longhorn.io/docs/archives/1.7.0/advanced-resources/system-backup-restore/backup-longhorn-system/) before the upgrade!

Starting with v1.5.0, Longhorn only supports upgrades from one minor version to the next. For example, upgrading from 1.5.x to 1.6.x is supported, but skipping versions (e.g., from 1.4.x to 1.6.x) is not.

Moreover, Longhorn does not support downgrades to earlier versions. This restriction helps prevent unexpected system behavior and issues associated with function incompatibility, deprecation, or removal.

### Changes to the role

- update Longhorn to `v1.7.3`
- ensure `dm_crypt` kernel module is loaded
- update `.gitignore`

### Further reading

Additional important information:

- [Best practices](https://longhorn.io/docs/archives/1.7.3/best-practices/)

### Changelogs

- [Longhorn v1.7.0 Release Notes](https://github.com/longhorn/longhorn/releases/tag/v1.7.0)
- [Longhorn v1.7.1 Release Notes](https://github.com/longhorn/longhorn/releases/tag/v1.7.1)
- [Longhorn v1.7.2 Release Notes](https://github.com/longhorn/longhorn/releases/tag/v1.7.2)
- [Longhorn v1.7.3 Release Notes](https://github.com/longhorn/longhorn/releases/tag/v1.7.3)

## 0.5.1+1.6.3

- update Longhorn to `v1.6.3`
- update `.yamllint`
- update Molecule test

Further reading:

- [Longhorn v1.6.3 Release Notes](https://github.com/longhorn/longhorn/releases/tag/v1.6.3)
- [Upgrading from v1.6.x (< v1.6.3) or v1.5.x](https://longhorn.io/docs/1.6.3/deploy/upgrade/longhorn-manager/)
- [Node Maintenance and Kubernetes Upgrade Guide](https://longhorn.io/docs/1.6.3/maintenance/maintenance/)

## 0.5.0+1.6.1

This is a major release update of Longhorn. Please read [Longhorn Important Notes](https://longhorn.io/docs/1.6.1/deploy/important-notes/) before upgrading!

Further reading:

- [Longhorn v1.6.1 Release Notes](https://github.com/longhorn/longhorn/releases/tag/v1.6.1)
- [Upgrading from v1.6.x (< v1.6.3) or v1.5.x](https://longhorn.io/docs/1.6.3/deploy/upgrade/longhorn-manager/)
- [Node Maintenance and Kubernetes Upgrade Guide](https://longhorn.io/docs/1.6.1/maintenance/maintenance/)

Other updates:

- update Longhorn to `v1.6.1`
- add `cryptsetup` and `dmsetup` as required OS packages

## 0.4.3+1.5.5

- update Longhorn to `v1.5.5`
- add more information to the UPGRADE section in README

## 0.4.2+1.5.4

- fix typo in README

## 0.4.1+1.5.4

- update Longhorn to `v1.5.4`
- Molecule: use `alvistack` instead of `generic` Vagrant boxes
- Molecule: various updates

## 0.4.0+1.5.3

Please read [Longhorn Important Notes](https://longhorn.io/docs/1.5.3/deploy/important-notes) before upgrading!

- **POTENTIALLY BREAKING**: `templates/longhorn_values_default.yml.j2`: removed `mkfsExt4Parameters` (was removed in Longhorn v1.5). This can now be configured in the [StorageClass](https://longhorn.io/docs/1.5.3/references/storage-class-parameters/). Also see [values.yaml](https://github.com/longhorn/longhorn/blob/v1.5.3/chart/values.yaml#L86)
- `templates/longhorn_values_default.yml.j2`: add two links
- update Longhorn to `v1.5.3`
- adjust Github action because of Ansible Galaxy changes
- update `meta/main.yml`: Add Ubuntu 22.04
- refactor Molecule test scenario

## 0.3.2+1.4.1

- rename `githubixx.harden-linux` to `githubixx.harden_linux`

## 0.3.1+1.4.1

- rename `githubixx.kubernetes-ca` to `githubixx.kubernetes_ca`
- rename `githubixx.kubernetes-controller` to `githubixx.kubernetes_controller`
- rename `githubixx.kubernetes-worker` to `githubixx.kubernetes_worker`

## 0.3.0+1.4.1

- K8s can't handle two keys with the same name but different values. So renamed `longhorn.components: user` to `longhorn.user.components: yes` and `longhorn.components: system` to `longhorn.system.components: yes`
- fix K8s node group handling to make it work on Ansible controller nodes different to localhost
- `templates/longhorn_values_default.yml.j2`: fix `systemManagedComponentsNodeSelector` value. Caused all settings to be completely ignored by Longhorn
- `templates/longhorn_values_default.yml.j2`: put all values into double quotes
- `tasks/install.yml`: add `force: true` property/value to install Helm chart
- `molecule/default/molecule.yml`: add two worker nodes to k8s_longhorn_system hosts group

## 0.2.1+1.4.1

- update Longhorn to `v1.4.1`

## 0.2.0+1.4.0

- **BREAKING**: `longhorn_nodes` variable was replaced by `longhorn_nodes_user` and `longhorn_nodes_system` variables.
- introduce `longhorn_label_nodes` variable to let the role set labels on the K8s worker nodes.
- introduce `longhorn_node_selector_user` and `longhorn_node_selector_system` variable. This makes it possible to run Longhorn user and system components on specific K8s nodes which have a specific node label set.
- introduce `longhorn_action` `add-node-label` and `remove-node-label`.
- set default `longhorn_action` in `molecule/default/converge.yml` to `template`.
- add variables for testing node labels and node selectors to `molecule/default/group_vars/all.yml`.
- change/add Ansible host groups in `molecule/default/molecule.yml` for testing node labels and node selectors.
- rename `action` variable to `cilium_action` in `molecule/default/prepare.yml`
- add tasks for K8s node labeling
- handle K8s node labeling in `tasks/(delete|install|upgrade).yml`
- change order of execution in `tasks/main.yml`. Task `Set default action...` should run before `Execute OS specific tasks` task
- `templates/longhorn_values_default.yml.j2`: add `csi.kubeletRootDir` property / add more comments / add `storageMinimalAvailablePercentage: 10` value / add `replicaAutoBalance: best-effort` value / add handling for K8s node labels

## 0.1.0+1.4.0

- initial commit for Longhorn `v1.4.0`
