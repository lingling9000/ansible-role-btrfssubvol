# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Add changelog on basis of <https://keepachangelog.com/en/1.1.0/>

## [1.0.0] - 2024-08-29

This first stable release includes the following features:

- Creation of new Btrfs subvolumes.
- Mount and unmount of Btrfs subvolumes.
- Adding and removing of /etc/fstab entries.
- Configuration of permissions on a mounted subvolume.
  - Recursive change is possible
- Configuration of mount options globally or per subvolume.
- Optional detection and removal of unmanaged mount points.

[Unreleased]: ../../../compare/v1.0.0..HEAD
[1.0.0]: ../../../releases/tag/v1.0.0
