# Ansible Role: btrfssubvol

Configure subvolumes on [Btrfs](https://btrfs.readthedocs.io/) devices.

The motivation behind this role is to automate the tedious task of creating
Btrfs subvolumes for a volume that does not have its root volume mounted.
(`subvolid=5`). This includes mounting the root volume, creating subvolumes,
adding subvolumes to /etc/fstab, and unmounting the root volume. This role has
been designed with flexibility in mind, while keeping the configuration overhead
as low as possible.


# Features

- Create new Btrfs subvolumes.
- Mount and unmount Btrfs subvolumes.
- Add and delete `/etc/fstab` entries.
- Configure user and group per subvolume. Recursive change is possible.
- Configure mount options globally or per subvolume.
- Detection and removal of duplicated subvolume mounts and `/etc/fstab` entries
(e. g. after a path change). Can be disabled by
`btrfssubvol_unmount_unmanaged_mounts`.


# Limitations

- While the mount points can be removed, the subvolumes itself will be
preserved.
- This role is currently only tested on Arch Linux, but it should be generic
enough to work on every distribution. Please open an issue if you face any
problems.
- Unmounting the unmanaged mounts is not always possible, because the targets
might be busy. Therefore, errors on these tasks are ignored by default. If it is
preferred that the role fail in such a case, it is possible to either modify
`btrfssubvol_unmount_ignore_errors` or set `unmount_ignore_errors` to `false` in
the `btrfssubvol_subvolumes` item definition.


# Requirements

- An already existing device formatted to Btrfs on the target host.
- Btrfs file system utilities need to be available on the target host.
- Users and groups for the subvolumes need to exist on the target host.
- Collection
[community.general](https://docs.ansible.com/ansible/latest/collections/community/general/index.html)
installed on the control host.


# Installation

This role is currently a standalone project and cannot be published on Ansible
Galaxy. It may be converted to a collection and published there at some
point. In the meantime, the following installation options are available:

- Using `requirements.yaml` / `requirements.yml` in the Ansible project
directory (**recommended**).

    - From [Codeberg Source](https://codeberg.org/lingling/ansible-role-btrfssubvol):

        ```yaml
        ---
        roles:
          - name: btrfssubvol
            src: https://codeberg.org/lingling/ansible-role-btrfssubvol
            scm: git
            version: v1.0.0
        ```    

    - Or from [GitHub Mirror](https://github.com/lingling9000/ansible-role-btrfssubvol):

        ```yaml
        ---
        roles:
          - name: btrfssubvol
            src: https://github.com/lingling9000/ansible-role-btrfssubvole
            scm: git
            version: v1.0.0
        ```    

    - Afterwards, install it using Ansible Galaxy CLI:

        ```bash
        ansible-galaxy install -r requirements.yaml
        ```

- Using [Git Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules).
In this case, the Ansible project directory must be a git repository.

    ```bash
    # Change directory to your Ansible project root.
    cd ~/path/to/ansible-project
    git submodule add https://codeberg.org/lingling/ansible-role-btrfssubvol roles/btrfssubvol
    ```

- [Git Clone](https://git-scm.com/docs/git-clone) in the projects roles folder
or a globally accessible roles path (see [Ansible Configuration Settings -
DEFAULT_ROLES_PATH](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-roles-path)).

    ```bash
    git clone https://codeberg.org/lingling/ansible-role-btrfssubvol ~/.ansible/roles/btrfssubvol
    ```


# Role Variables

This role is designed to run even without configuration. But then it won't do
anything except syntax checking.

The subvolumes are defined in the `btrfssubvol_subvolumes` list. However, before
defining subvolumes, **the UUID of the target device(s) must be obtained**. This
is necessary for the role to know on which device(s) the subvolumes should be
created. Obtain the UUID(s) with following shell command:

```bash
# Replace /dev/sdXY with your device.
blkid --match-tag UUID --output value /dev/sdXY
```

Set `btrfssubvol_device_uuid` to the obtained UUID. Alternatively, set the UUID
per item in `btrfssubvol_subvolumes` using the `device_uuid` key. The latter is
useful when managing multiple Btrfs devices.

See [Example Playbook](#example-playbook) for a quick start or read on for an
explanation of all the available variables.


## External Variables

None.


## Public Role Variables

**Define the subvolumes**:

- `btrfssubvol_subvolumes` (list / elements=dictionary), default
`[]`. List of dictionaries defining the subvolumes to be managed. See [Defining
Subvolumes](#defining-subvolumes) for a tutorial or jump directly to the [list
of all supported keys](#all-subvolume-options).


**Defaults for all subvolumes** (can be overwritten per subvolume):

- `btrfssubvol_device_uuid` (string), default `none`. UUID of the
device, which should contain the Btrfs subvolumes. Is used to determine the
right device for mounting and fstab entries.
- `btrfssubvol_mount_parent` (string), default `/media`. The parent directory
for the subvolume mounts.
- `btrfssubvol_mount_options` (list / elements=string), default `["defaults",
"compress=zstd", "discard=async", "noatime"]`. Contains a list of mount options
used for the subvolume mounts. See [Btrfs Mount Options](#btrfs-mount-options)
for a detailed explanation.
- `btrfssubvol_unmount_ignore_errors` (boolean), default `true`. If set to
`true`, errors will be ignored when unmounting subvolumes. Note that this only
affects unmounting the unmanaged mounts. It does not affect to changes of the
mount state of the managed mount point.
- `btrfssubvol_unmount_unmanaged_mounts` (boolean), default `true`. When set to
`true`, the role attempts to remove mounts of a subvolume that are not managed
by this role. This also ensures that there are no remaining mounts after
changing the mount path with either the `btrfssubvol_mount_parent` or
`mount_point` key in the `btrfssubvol_subvolumes` list.


### Defining Subvolumes

Before digging into subvolume definition, note that this role automatically
appends an `@` to the subvolume name on the root volume (`subvolid=5`). This is
a common convention for identifying Btrfs subvolumes and is therefore hardcoded
into the role. Also, the following examples assume that
`btrfssubvol_device_uuid` is set. Otherwise, the `device_uuid` must be set on
each subvolume.

Defining subvolumes requires only the `name` key:

```yaml
btrfssubvol_subvolumes:
  - name: documents
  - name: photos
```

The role will then create the subvolumes, create an `/etc/fstab` entry, create
the mount point, and mount the subvolumes. The subvolume will be mounted under
the directory specified in `btrfssubvol_mount_parent`. Assuming the default is
used, this would be `/media/documents` and `/media/photos`, respectively. The
mount point can be set per subvolume definition with the `mount_point` key.

```yaml
btrfssubvol_subvolumes:
  # - name: documents   # Remove the corresponding item to stop managing a subvolume.
  - name: photos
    mount_point: /srv/nfs/photos
```

With this definition the photos subvolume will be mounted at `/srv/nfs/photos`.
By default, the new subvolume will be owned by the _root_ user and group. This
can be modified by specifying the `owner` and `group` keys:

```yaml
btrfssubvol_subvolumes:
  - name: photos
    owner: alice
    group: photo-users
    mount_point: /srv/nfs/photos
```

This will set the group and owner on the mounted subvolume folder. However, this
only affects the subvolume folder itself. If it is desired to change the
permissions on all files in the subvolume, set `recurse` to `true`. Note that
the users and groups must exist on the system, creating users and groups is
outside of the scope of this role. Refer to the
[ansible.builtin.user](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
and
[ansible.builtin.group](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group_module.html)
modules for user and group creation, respectively.

Mounting behavior can be adjusted with the`mount_state` key, which accepts all
states of the
[ansible.posix.mount](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-state)
module:

```yaml
btrfssubvol_subvolumes:
  - name: photos
    owner: alice
    group: photo-users
    mount_point: /srv/nfs/photos
    mount_state: absent
```

This configuration removes the `/etc/fstab` entry, unmounts the subvolume, and
removes the mount point. It doesn't remove the subvolume itself. Note that the
role will fail if the device can't be unmounted, which happens when the device
is already in use (_target is busy_). Use `absent_from_fstab` to avoid failures,
but remove the entries from `/etc/fstab`, so they will be gone after the next
reboot. Another way to remove the mount is to set `mount_point` to an empty
string:

```yaml
btrfssubvol_subvolumes:
  - name: photos
    owner: alice
    group: photo-users
    mount_point: ""
    # mount_state: absent              # has no effect when mount_point is set to ""
    unmount_ignore_errors: true        # already the default
    unmount_unmanaged_mounts: true     # already the default
```

Setting `mount_point` to `""` causes the initial mount task to be skipped. With
this configuration, all mounts of the subvolume _photos_ will be considered as
unmanaged and therefore unmounted as long as `unmount_unmanaged_mounts` is set
to `true`. 

Note that it's is not possible to set the permissions when the subvolume isn't
mounted. However, it is good practice to keep the appropriate keys (`owner`,
`group`, `recurse`) defined, as they will take effect the next time the
subvolume is mounted. Otherwise, the subvolume will be set as owned by _root_ on
the next time it is mounted.


#### All Subvolume Options

Some values are inherited or composed from the defaults set for this role. The
defaults can be overridden on a per-item item in `btrfssubvol_subvolumes`. The
following table shows all the options for defining a subvolume:

| Key                                  | Default                                      | Description                                                                                                               |
|--------------------------------------|----------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| `device_uuid` (string)               | `{{ btrfssubvol_device_uuid }}`              | Same as `btrfssubvol_device_uuid`, but per subvolume.                                                                     |
| `group` (string)                     | -                                            | The group of the subvolume. Will be _root_ if unset.                                                                      |
| **`name` (string, required**)        | -                                            | The name of the subvolume. Must be a filename compatible string. This will be the name for the subvolume and mount point. |
| `mount_options` (list)               | `{{ btrfssubvol_mount_options }}`            | Same as `btrfssubvol_mount_options`, but per subvolume.                                                                   |
| `mount_point` (string)               | `{{ btrfssubvol_mount_parent }}/{{ name }}`  | The mount point of the subvolume. Overrides the default of being a subdirectory of `btrfssubvol_mount_parent`.            |
| `mount_state` (string)               | -                                            | One of `["absent", "absent_from_fstab", "mounted", "present", "unmounted", "remounted", "ephemeral"]`. See [ansible.posix.mount](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#parameter-state) documentation. Will be `mounted` if unset. |
| `owner` (string)                     | -                                            | The owner of the subvolume. Will be _root_ if unset.                                                                      |
| `recurse` (boolean)                  | -                                            | If the user and group should be modified recursively. Useful for modifying an already existing subvolume.                 |
| `unmount_ignore_errors` (boolean)    | `{{ btrfssubvol_unmount_ignore_errors }}`    | Same as `btrfssubvol_unmount_ignore_errors`, but per subvolume.                                                           |
| `unmount_unmanaged_mounts` (boolean) | `{{ btrfssubvol_unmount_unmanaged_mounts }}` | Same as `btrfssubvol_unmount_unmanaged_mounts`, but per subvolume.                                                        |

Complete example with all possible values set:

```yaml
btrfssubvol_subvolumes:
  - name: photos
    device_uuid: yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
    owner: alice
    group: photo-users
    recurse: true
    mount_options:
      - defaults
      - compress=no
      - relatime
    mount_point: /srv/nfs/photos
    mount_state: remounted
    unmount_unmanaged_mounts: true
    unmount_ignore_errors: false
```


### Btrfs Mount Options

To understand and customize the mount options, see the following man pages:

- [**mount(8)** - FILESYSTEM-INDEPENDENT MOUNT OPTIONS](https://man.archlinux.org/man/mount.8#FILESYSTEM-INDEPENDENT_MOUNT_OPTIONS)
- [**btrfs(5)** - BTRFS SPECIFIC MOUNT OPTIONS](https://btrfs.readthedocs.io/en/latest/Administration.html#btrfs-specific-mount-options)

Explanation of the default of `btrfssubvol_mount_options`:

- `defaults`: See link to **mount(8)** above.
- `compress=zstd`: Btrfs specific mount option to enable compression with
[zstd](https://facebook.github.io/zstd/). On volumes containing already
compressed files (such as image, video, or audio files), it may be desirable to
disable compression completely.
- `discard=async`: Using this mount option enables asynchronous execution of the
SSD TRIM command, which is a special feature of Btrfs. In the case of full disk
encryption, or in some other cases, it is may be desirable to disable this with
`nodiscard`. Refer to the following resources to make an informed decision:
    - [Trim/discard (BTRFS
    documentation)](https://btrfs.readthedocs.io/en/latest/Trim.html)
    - [Solid State Drive - TRIM (Arch
    Wiki)](https://wiki.archlinux.org/title/Solid_state_drive#TRIM)
    - [dm-crypt - Discard/TRIM support for solid state drives (SSD)
    (ArchWiki)](https://wiki.archlinux.org/title/Dm-crypt/Specialties#Discard/TRIM_support_for_solid_state_drives_(SSD))
- `noatime`: This option can improve the performance of a Btrfs volume. Refer to
[**btrfs(5)** - NOTES ON GENERIC MOUNT
OPTIONS](https://btrfs.readthedocs.io/en/latest/btrfs-man5.html#notes-on-generic-mount-options)
for details.

The `subvol=<path>` option is derived from the name field in
`btrfssubvol_subvolumes`. **Therefore, it neither necessary nor allowed to set
`subvol=` or `subvolid=`!** The `btrfssubvol_mount_options` will be merged with
the derived `subvol=<path>` option at execution time.

Redefine `btrfssubvol_mount_options` to change the options. If this variable
is set to an empty list (`[]`), the mount option `defaults` is set at execution
time.


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
# Replace with the real UUID obtained from `blkid`!
btrfssubvol_device_uuid: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

btrfssubvol_subvolumes:
  # Subvolume backups will be owned by user root and group root.
  - name: backups

  # Subvolume documents will be owned by user bob and group office. If the
  # permission should be changed recursively, set `recurse: true`
  - name: documents
    owner: bob
    group: office

  # Subvolume photos is on another device and mounted on a specified mount point.
  - name: photos
    device_uuid: yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
    mount_point: /srv/nfs/photos
```

**Playbook (`playbook.yaml`)**:

```yaml
- name: Configure
  hosts: all
  tasks:
    - name: Include role btrfssubvol
      ansible.builtin.include_role:
        name: btrfssubvol
```


## Pre-Check Variable Syntax

In a larger playbook, it may be desirable to check variable syntax at an early
stage. This prevents the playbook from aborting in the middle of a run. Two
files are provided for this purpose:

- [checks-syntax.yaml](./tasks/checks-syntax.yaml): Checks the syntax of role
variables.
- [checks-host-config.yaml](./tasks/checks-host-config.yaml): Checks the host
configuration, especially if the users and groups specified in
`btrfssubvol_subvolumes` exist.

For example, the following task can be added at the desired position in the
playbook to perform the syntax checks separately:

```yaml
- name: Include btrfssubvol role variable syntax checks
  ansible.builtin.include_role:
    name: btrfssubvol 
    tasks_from: checks-syntax.yaml
```


# License

[UNLICENSE](./LICENSE)


# Author Information

[lingling](../../../../../lingling)
