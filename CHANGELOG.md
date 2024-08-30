# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.1] - 2024-08-30

### Added

- Metadata in `meta/main.yaml` for Ansible Galaxy (`namespace`, `role_name` and
`galaxy_tags`).

## [1.1.0] - 2024-08-30

Ansible Galaxy Release.

This release improves the documentation, particularly the installation instructions. It now mentions the new option to download from Ansible Galaxy first.

### Added

- Released role on Ansible Galaxy as [lingling9000.btrfssubvol](https://galaxy.ansible.com/ui/standalone/roles/lingling9000/btrfssubvol/).
- Installation instructions using Ansible Galaxy as source.
- Changelog on basis of <https://keepachangelog.com/en/1.1.0/>.
- Code of conduct on basis of [Contributor Covenant
v2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/).

### Changed

- To reflect the Ansible Galaxy namespace `lingling9000` on Codeberg, an
organization has been created and the repository moved. It is now located at
<https://codeberg.org/lingling9000/ansible-role-btrfssubvol>. The corresponding
links in the documentation have been updated.
- Examples reflect the role name from Ansible Galaxy.
- Minor wording and formatting improvements in documentation.

## [1.0.0] - 2024-08-29

This first stable release includes the following features:

- Creation of new Btrfs subvolumes.
- Mount and unmount of Btrfs subvolumes.
- Adding and removing of /etc/fstab entries.
- Configuration of permissions on a mounted subvolume.
  - Recursive change is possible
- Configuration of mount options globally or per subvolume.
- Optional detection and removal of unmanaged mount points.

[Unreleased]: ../../../../../lingling9000/ansible-role-btrfssubvol/compare/v1.1.1..HEAD
[1.1.0]: ../../../../../lingling9000/ansible-role-btrfssubvol/compare/v1.1.0..v1.1.1
[1.1.0]: ../../../../../lingling9000/ansible-role-btrfssubvol/compare/v1.0.0..v1.1.0
[1.0.0]: ../../../../../lingling9000/ansible-role-btrfssubvol/releases/tag/v1.0.0
