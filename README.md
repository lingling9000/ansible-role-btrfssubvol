# Ansible Role: btrfssubvol

Configure subvolumes for [Btrfs](https://btrfs.readthedocs.io/).

The motivation behind this role is to automate the tedious task of creating
Btrfs subvolumes for a volume that has not mounted their root volume
(`subvolid=5`). That includes mounting the root volume, creating subvolumes, add
subvolumes to `/etc/fstab` and unmount root volume again.


# Features

- Create new Btrfs subvolumes.
- Mount Btrfs subvolumes.
- Add `/etc/fstab` entries.
- Configure user and group per subvolume. Recursive change is possible.


# Limitations

- Removal of subvolumes is not possible.
- Only sub directories of `/media/` are available as mount points (properly
changes in future).
- Check mode: Note that the Btrfs root volume is mounted and unmounted even when
in check mode. This is necessary to check the tasks in
[subvolume.yaml](./tasks/subvolumes.yaml).
- This role is currently only tested on Arch Linux, but it should be generic
enough to work on every distribution.


# Requirements

- An already existing device formatted to Btrfs.
- Btrfs file system utilities need to be available. 
- Users and groups for the subvolumes need to exist on the target system.


# Role Variables


## External Variables

None.


## Public Role Variables

- `btrfssubvol_device_uuid` (string, mandatory), default `none`. UUID of the
device, which should contain the Btrfs subvolumes. Is used to determine the
right device for mounting and fstab entries.
- `btrfssubvol_rootvol_mount_point` (string, mandatory), default `/mnt`.
Temporary mount point for the root volume of the Btrfs device. Modification is
only necessary if the mount point `/mnt` is unavailable. The role will fail if
the folder is occupied by another mount.
- `btrfssubvol_subvolumes` (list, mandatory), default `none`. List of mappings
defining the subvolumes to be managed.
    - `name` (string, mandatory). The name of the subvolume. Need to be a
    filename compatible string. This will be the name for the subvolume and
    mount point.
    - `owner` (string, optional). The owner (user) of the subvolume.
    - `group` (string, optional). The group, which will own the subvolume.
    - `recurse` (boolean, optional). If the user and group should be modified
    recursively. Useful for modifying a already existing subvolume.


## Internal Role Variables

None.


# Tags

To simplify the testing of this role, all tasks are tagged. Check Ansible
documentation of
[tags](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html)
for reference. Following tags are available:

- `checks`: Added to all tasks, which perform checks or gather information for
subsequent checks. This tag contains a superset of tasks having a tag beginning
with `checks-`.
- `checks-syntax`: Added only to tasks performing simple syntax checks on
variables, e. g. if they are defined and have the correct data type.
- `checks-host-config`: Added to tasks, which check if host configuration does
comply with the configuration in variables.
- `configure`: Added to tasks, which actually perform changes on the target host
or prepare for them.


# Dependencies

None.


# Example Playbook

**Host configuration (`host_vars/somehost.mydomain.tld/main.yaml`)**:

```yaml
# replace with an existing UUID on the system!
btrfssubvol_device_uuid: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
btrfssubvol_subvolumes:
  # subvolume backups will be owned by user root and group root
  - name: backups    

  # subvolume photos will be owned by user alice and group photo-users
  - name: photos
    owner: alice
    group: photo-users

  # subvolume documents will be owned by user bob and group root
  - name: documents
    owner: bob
```

**Playbook (`playbook.yaml`)**:

```yaml
- name: Configure
  hosts: somehost.mydomain.tld
  tasks:
    - name: Include role btrfssubvol
      ansible.builtin.include_role:
        name: btrfssubvol
```


## Pre-Check Variable Syntax

In a larger playbook it may desirable to check the variable syntax in an early
stage, to prevent aborts in the middle of the playbook run. Two files are
provided for this case:

- [checks-syntax.yaml](./tasks/checks-syntax.yaml): Checks the syntax of role
variables.
- [checks-host-config.yaml](./tasks/checks-host-config.yaml): Checks the host
configuration, especially if the users and groups specified in
`btrfssubvol_subvolumes` exist.

For example, following tasks can be added on the desired position in the
playbook to run the syntax checks separately:

```yaml
- name: Include btrfssubvol role variable syntax checks
  ansible.builtin.include_role:
    name: btrfssubvol 
    tasks_from: checks-syntax.yaml
```


# License

[UNLICENSE](./LICENSE)


# Author Information

[lingling](../../../../../)
