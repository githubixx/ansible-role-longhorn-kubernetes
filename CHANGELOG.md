<!--
Copyright (C) 2023 Robert Wimmer
SPDX-License-Identifier: GPL-3.0-or-later
-->

# Changelog

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
