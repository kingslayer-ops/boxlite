# Storage

## Rootfs preparation

The rootfs builder assembles a container filesystem from OCI image layers:

```text
Image Layers          Rootfs Builder              Box Rootfs
┌─────────┐          ┌─────────────┐          ┌─────────────┐
│ Layer 1 │────┐     │             │          │ /bin        │
├─────────┤    │     │  Extract &  │          │ /etc        │
│ Layer 2 │────┼────▶│   Overlay   │─────────▶│ /usr        │
├─────────┤    │     │             │          │ /var        │
│ Layer N │────┘     └─────────────┘          │ ...         │
└─────────┘                                   └─────────────┘
```

**Key operations:**

- Layer extraction and overlay mounting
- DNS configuration injection
- Copy-on-write snapshot creation

## Volume management

**Supported volume types:**

| Type           | Description              | Use Case                             |
|----------------|--------------------------|--------------------------------------|
| **virtiofs bind** | Caller-selected host directory or file | Sharing a specific host path with a box |
| **Named local volume** | BoxLite-managed directory shared over virtio-fs | Persistent data shared independently of box lifecycle |
| **QCOW2 disk** | Copy-on-write disk image | Box-private root filesystem, kept while the box exists |

Named volume identity and payload are separate: `{volume-id}/.metadata.json` stores id, name, and
creation time, while only `{volume-id}/_data/` is exposed to the guest. Box configuration stores the
canonical id rather than the host path. Volume removal is blocked while any persisted box
configuration references that id. See [Mounting volumes](../guides/volumes.md).

**QCOW2 features:**

- Thin provisioning (allocate on write)
- Snapshot support
- Shared base images across Boxes

## Home directory

BoxLite keeps boxes, images, and its database under `~/.boxlite` by default. The directory's
layout is in [File formats](../reference/file-formats.md#home-directory).
