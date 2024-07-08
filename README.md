# Ansible Role: btrfssubvol

Configure subvolumes for btrfs.


# Features

- Create new btrfs subvolumes.
- Mount btrfs subvolumes.
- Add fstab entries.
- Configure user and group of subvolumes. Recursive change is possible.


# Limitations

- Removal of subvolumes is not possible.


# Requirements

None.


# Role Variables


## External Variables

None.


## Public Role Variables

- `btrfssubvol_device_uuid` (string, mandatory), default `none`. UUID of the
device, which should contain the btrfs subvolumes. Is used to determine the
right device for mounting and fstab entries.
- `btrfssubvol_rootvol_mount_point` (string, mandatory), default `/mnt`.
Temporary mount point for the root volume of the btrfs device. Need only to be
set if the mount point `/mnt` is not available.
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


# Dependencies

None.


# Example Playbook

```yaml
- hosts: all
  tasks:
    - name: Include role btrfssubvol
      ansible.builtin.include_role:
        name: btrfssubvol
```


# License

[UNLICENSE](./LICENSE)


# Author Information

lingling
