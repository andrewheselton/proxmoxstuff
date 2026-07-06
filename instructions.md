# Proxmox Remote VM Migration Command

This document explains the following Proxmox `qm remote-migrate` command:

```bash
qm remote-migrate 100 100 'host=11.0.0.159,apitoken=PVEAPIToken=root@pam!migration-token=87634b7e-779c-47cc-b888-ab3dd5093380,fingerprint=39:16:8D:50:55:02:36:5E:6E:E0:57:18:60:DC:F4:AA:39:E0:05:20:DA:81:3C:ED:96:32:17:47:53:62:02:B3' --target-bridge vmbr0 --target-storage local-lvm
```

> **Important security note:** the API token value in this command is sensitive.  
> Anyone with that token and suitable permissions may be able to access or control parts of your Proxmox server.  
> After testing, consider deleting or rotating the token.

---

## What the command does

This command migrates a virtual machine from one Proxmox host to another Proxmox host using Proxmox's remote migration feature.

In this example:

- The source VM ID is `100`
- The destination VM ID will also be `100`
- The target Proxmox host is `11.0.0.159`
- The target bridge is `vmbr0`
- The target storage is `local-lvm`

---

## Command breakdown

### `qm`

```bash
qm
```

`qm` is the Proxmox command-line tool used to manage QEMU/KVM virtual machines.

You use it for tasks such as:

- Creating VMs
- Starting and stopping VMs
- Changing VM settings
- Migrating VMs
- Managing disks and snapshots

---

### `remote-migrate`

```bash
remote-migrate
```

This tells Proxmox that you want to migrate a VM to another Proxmox host that is not necessarily part of the same cluster.

This is useful when moving a VM between:

- Two separate Proxmox servers
- A new Proxmox installation
- A replacement host
- A standalone Proxmox machine

---

### Source VM ID

```bash
100
```

The first `100` is the VM ID on the current/source Proxmox server.

This means:

> Migrate VM `100` from this Proxmox host.

You can check VM IDs with:

```bash
qm list
```

---

### Target VM ID

```bash
100
```

The second `100` is the VM ID that will be used on the destination Proxmox server.

This means:

> Create the migrated VM on the remote host as VM ID `100`.

The target VM ID must not already exist on the destination server.

If VM `100` already exists on the target host, choose another ID, for example:

```bash
qm remote-migrate 100 101 ...
```

That would migrate source VM `100` to destination VM `101`.

---

## Remote host connection string

```bash
'host=11.0.0.159,apitoken=PVEAPIToken=root@pam!migration-token=87634b7e-779c-47cc-b888-ab3dd5093380,fingerprint=39:16:8D:50:55:02:36:5E:6E:E0:57:18:60:DC:F4:AA:39:E0:05:20:DA:81:3C:ED:96:32:17:47:53:62:02:B3'
```

This section tells Proxmox how to connect to the destination server.

It is wrapped in single quotes so the shell treats the whole section as one argument.

---

### `host=11.0.0.159`

```bash
host=11.0.0.159
```

This is the IP address of the destination Proxmox server.

In this example, Proxmox will try to migrate the VM to:

```text
11.0.0.159
```

Make sure the source Proxmox host can reach this IP address.

You can test this with:

```bash
ping 11.0.0.159
```

You can also check that the Proxmox web/API port is reachable:

```bash
curl -k https://11.0.0.159:8006
```

---

### `apitoken=...`

```bash
apitoken=PVEAPIToken=root@pam!migration-token=87634b7e-779c-47cc-b888-ab3dd5093380
```

This is the API token used to authenticate to the destination Proxmox server.

The format is:

```text
PVEAPIToken=username@realm!token-name=token-value
```

In this example:

| Part | Meaning |
|---|---|
| `PVEAPIToken` | Tells Proxmox this is an API token |
| `root@pam` | The Proxmox user account |
| `migration-token` | The name of the API token |
| `87634b7e-779c-47cc-b888-ab3dd5093380` | The secret token value |

The token must have enough permissions on the target Proxmox host to create and configure the migrated VM.

---

### `fingerprint=...`

```bash
fingerprint=39:16:8D:50:55:02:36:5E:6E:E0:57:18:60:DC:F4:AA:39:E0:05:20:DA:81:3C:ED:96:32:17:47:53:62:02:B3
```

This is the SSL certificate fingerprint of the destination Proxmox server.

It allows the source server to confirm it is connecting to the expected destination host.

This helps protect against connecting to the wrong server.

You can usually find the fingerprint in the Proxmox web interface under the node certificate details, or check it from the command line on the target Proxmox host.

---

## Target bridge

```bash
--target-bridge vmbr0
```

This tells Proxmox which network bridge to connect the migrated VM's network card to on the destination server.

In this example, the VM will use:

```text
vmbr0
```

`vmbr0` is commonly the main Linux bridge used by Proxmox for VM networking.

Before running the migration, check that the target host has a bridge called `vmbr0`.

On the target Proxmox host, you can check with:

```bash
ip link show vmbr0
```

or:

```bash
cat /etc/network/interfaces
```

---

## Target storage

```bash
--target-storage local-lvm
```

This tells Proxmox where to place the VM disks on the destination server.

In this example, the migrated VM disks will be stored on:

```text
local-lvm
```

`local-lvm` is a common Proxmox storage type that uses LVM-thin.

Before migration, check that the storage exists on the target host:

```bash
pvesm status
```

You should see something like:

```text
Name        Type      Status
local       dir       active
local-lvm   lvmthin   active
```

---

## Full command, formatted for readability

```bash
qm remote-migrate 100 100 \
  'host=11.0.0.159,apitoken=PVEAPIToken=root@pam!migration-token=87634b7e-779c-47cc-b888-ab3dd5093380,fingerprint=39:16:8D:50:55:02:36:5E:6E:E0:57:18:60:DC:F4:AA:39:E0:05:20:DA:81:3C:ED:96:32:17:47:53:62:02:B3' \
  --target-bridge vmbr0 \
  --target-storage local-lvm
```

---

## Pre-migration checklist

Before running the command, check the following:

- The destination Proxmox server is powered on
- The source server can reach `11.0.0.159`
- The API token is valid
- The API token has enough permissions
- The certificate fingerprint is correct
- VM ID `100` does not already exist on the destination server
- The destination server has a bridge called `vmbr0`
- The destination server has storage called `local-lvm`
- The destination storage has enough free space
- The VM is backed up before migration

---

## Useful checks

### List VMs on the source host

```bash
qm list
```

### Check target storage

Run this on the destination Proxmox host:

```bash
pvesm status
```

### Check target bridge

Run this on the destination Proxmox host:

```bash
ip link show vmbr0
```

### Check the VM config before migration

Run this on the source Proxmox host:

```bash
qm config 100
```

---

## Safer test version

If you want to avoid overwriting or clashing with an existing VM ID on the target host, migrate it to a different target VM ID, for example:

```bash
qm remote-migrate 100 101 'host=11.0.0.159,apitoken=PVEAPIToken=root@pam!migration-token=YOUR-TOKEN-HERE,fingerprint=TARGET-FINGERPRINT-HERE' --target-bridge vmbr0 --target-storage local-lvm
```

This migrates:

```text
Source VM 100 -> Target VM 101
```

---

## Troubleshooting

### Authentication error

Check:

- API token name
- API token value
- User realm, for example `root@pam`
- Permissions assigned to the token
- Whether privilege separation is enabled on the token

---

### Target storage not found

If you see an error about `local-lvm`, check the destination storage names:

```bash
pvesm status
```

Then change:

```bash
--target-storage local-lvm
```

to the correct storage name.

Example:

```bash
--target-storage local
```

---

### Target bridge not found

If `vmbr0` does not exist on the destination server, check:

```bash
cat /etc/network/interfaces
```

Then use the correct bridge name.

Example:

```bash
--target-bridge vmbr1
```

---

### VM ID already exists

If VM `100` already exists on the destination server, use another target VM ID:

```bash
qm remote-migrate 100 101 ...
```

---

## Summary

This command migrates VM `100` from the current Proxmox host to another Proxmox host at `11.0.0.159`.

It authenticates using an API token, verifies the remote host using its certificate fingerprint, connects the VM to `vmbr0`, and stores the VM disks on `local-lvm`.

In plain English:

> Move VM 100 to the Proxmox server at 11.0.0.159, keep its VM ID as 100, attach it to bridge vmbr0, and place its disks on local-lvm.
