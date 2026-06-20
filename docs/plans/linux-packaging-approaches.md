# Linux Packaging

**Context:** Open-source GitHub release, dogfooded internally. Qt dynamically linked; FTDI and internal libs static.  
**Requirements:** Docker support, USB/serial access, self-contained where feasible.

## Comparison

| Criterion                  | Tarball     | AppImage | .deb  | Flatpak |
|----------------------------|-------------|----------|-------|---------|
| Self-contained             | No ¹        | Yes      | No    | Yes     |
| Docker-friendly            | Yes         | No ²     | Yes   | No ³    |
| USB / serial access        | Yes         | Yes      | Yes   | Needs config |
| Cross-distro               | Yes         | Yes      | No    | Yes     |
| udev rules auto-install    | No          | No       | Yes   | No      |
| Build complexity           | Low         | Medium   | Low   | High    |
| Artifact size              | Small       | ~100 MB  | Med   | Large   |

¹ `libxcb`, `libGL`, `libpulse` must exist on host.  
² Requires `libfuse2`, absent in most base container images.  
³ Requires systemd user session and kernel namespaces.

## Approaches

**Tarball:** System-level xcb/GL/pulse dependencies remain on the host regardless of whether RPATH or a launcher script is used. Not genuinely self-contained.

**AppImage:** Bundles everything including xcb and GL shims. Ideal for bare-metal desktop users on unknown distros. Poor fit for Docker-based workflows due to the FUSE requirement.

**`.deb`:** System Qt declared as a minimum-version dependency, resolved at install time by the package manager. Installs cleanly into Ubuntu/Debian Docker images with no special privileges. `postinst` can deploy udev rules automatically. RPM-based distros not covered.

**Flatpak:** USB/hardware access and Docker compatibility both require non-trivial configuration. Not suitable.

## Recommendation

Ship both AppImage and `.deb`. Build output is identical, only the packaging step differs.

| Artifact | Target |
|----------|--------|
| `.AppImage` | Bare-metal / unknown distro |
| `.deb` | Ubuntu/Debian, Docker workflows |
