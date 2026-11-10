# DRAFT - ojuny-home-server-template

Status: draft scope. Nothing is built. This file describes what the repository
is supposed to contain, not what it contains today.

## Purpose

A headless home NAS and server template for Ojuny users. It is a complete NixOS
flake with a web panel for configuration, and it offers the services a home
network usually needs as an optional stack: DNS, ad blocking, reverse proxy,
VPN, build cache, media, file sharing and backup.

The goal is appliance-like: install the image, answer a few questions, and log
in to the web panel. Everything except what is needed for the server and panel
to work at all is optional and off by default. After install the server does
almost nothing until the user turns services on from the web panel.

## What the repository will contain

```text
ojuny-home-server-template/
  flake.nix                  server host and service modules
  flake.lock
  modules/
    01-users/                administrator and service accounts
    02-core/                 base system and updates
    03-services/             optional service pages
    04-storage/              disks, pools and shares
    05-network/              DNS, proxy, VPN and firewall
    06-media/                ARR stack, Jellyfin and Jellyseerr
    07-build/                Hydra and nix-cache-discovery
    08-backup/               backup targets and schedules
    09-panel/                web panel and dashboard
    backend/                 generated service configuration
    patches/                 patch switch
  scripts/                   setup, disks, lifecycle
  docs/                      service guide, storage, network, recovery
  tests/                     VM and service checks
```

### Required for the server to work

- Base NixOS system, networking and the firewall.
- Web panel and dashboard, reachable by IP and port on first boot.
- A first administrator account.
- Updates and a service health page that only lists enabled services.

Everything else in this document is optional and disabled by default.

### Optional services

- DNS server, automatically configured from the services the user enables.
- Reverse proxy with generated virtual hosts for enabled services.
- Ad blocking DNS through AdGuard Home or Pi-hole.
- Personal VPN using WireGuard, with a documented port-forward requirement.
- Hydra build server, enabling `nix-serve-ng` when a cache is not already
  present.
- `nix-cache-discovery` for a home LAN.
- ARR stack with Jellyfin and Jellyseerr.
- NFS shares with a generated Ojuny patch that enables them on a user's device.
- ownCloud Infinite Scale for cloud storage.
- Automatic backup with retention and scope.
- A media library, downloads and indexer pages for the ARR stack.

### Dashboard

- One page listing every enabled service and its status.
- Links to each service's own web panel.
- A generated DNS name per service when DNS is enabled, and an IP and port
  link when it is not.
- Storage, network and backup health summaries.
- A service enable and disable flow that updates DNS and the proxy together
  when those services are enabled.

### Storage and shares

- Disk detection, pool creation and mount layout.
- NFS exports with per-device and per-user access.
- A generated patch for an Ojuny desktop or server that mounts the shares over
  both the local network and the VPN.
- Snapshot and backup policy per share.
- Disk health and SMART reporting.
- No silent disk erasure. Every destructive action requires confirmation.

### Media

- ARR stack with a web page per service.
- Jellyfin and Jellyseerr wired to the same library and indexers.
- Optional hardware transcoding when the GPU supports it.
- Storage paths owned by the media service accounts.
- No bundled media and no indexer credentials in tracked files.

### Build services

- Hydra with a local project and jobset model.
- `nix-serve-ng` as the cache when Hydra is enabled.
- `nix-cache-discovery` as the LAN cache when enabled.
- A clear precedence rule so only one cache is the primary substituter.
- Cache signing and trust documented.

### Authentication

- Local accounts declared in Nix, shared across services where the service
  supports it.
- The first admin account created during setup.
- Service accounts that are not login accounts.
- No SSO, LDAP or Kerberos dependency.
- No default password and no anonymous administrative access.

### Documentation

- The service matrix: required, default, optional and unsupported.
- The network diagram and every port.
- DNS and reverse proxy behavior when those services are enabled.
- Storage layout, pools and share permissions.
- VPN setup and the port-forward requirement.
- Backup and restore.
- Recovery when the panel or a service fails.

### Tests

- The server flake evaluates and builds.
- A default install exposes only the panel and no optional service.
- The panel starts and lists the enabled services.
- Enabling DNS makes it resolve every enabled service.
- Enabling the reverse proxy makes it reach each enabled service.
- A share mounts from a test Ojuny client over the LAN.
- `nix-cache-discovery` serves a path to a peer when enabled.
- Hydra builds a test job and the cache serves it when enabled.
- Backup and restore round-trip a share.
- Disabling a service removes its DNS record and proxy route.

## Limits and non-goals

- A single home server, not a high-availability cluster.
- No GlusterFS, etcd, Patroni or Galera cluster support in this template.
- No public multi-tenant hosting.
- No automatic port forwarding. The VPN needs a forwarded port unless the user
  chooses an overlay that does not.
- No media acquisition automation that bypasses licensing.
- No guarantee that every service supports central authentication.
- No backup to a cloud provider by default.
- No disk hot-plug or enterprise storage certification.
- No service enabled by default beyond what the panel needs.

## Open questions

- Panel technology and whether it shares `nix-web-gui`.
- Which ad blocking service to offer first, and whether both are supported.
- Which NFS export model and how the generated client patch is versioned.
- Whether the ARR stack runs natively, in containers, or both.
- Whether ownCloud Infinite Scale is in the first scope.
- How the first-boot flow creates the admin account and storage pool.
- Which services deserve a one-click enable flow in the panel.

## Sources

- `nix-cache-discovery/README.md`.
- `nixos-configs/dendrite/modules/services/hydra.nix`,
  `nix-serve.nix` and `glusterfs.nix` as prior art, not as scope.
- `ojuny-docs/08-scope-and-non-goals.md`.
- `ojuny-template/docs/00-overview.md`, `01-architecture.md`,
  `03-host-setup.md`.
