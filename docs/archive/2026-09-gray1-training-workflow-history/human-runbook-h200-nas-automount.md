# Human Runbook: H200 ywang NAS Automatic Mount

Created: 2026-09-09  
Audience: H200 administrator or `ywang` operator with sudo access  
Status: online configuration observed active and verified on 2026-09-09; controlled reboot acceptance remains pending

## 1. Goal

Configure the H200 host to register the ywang NAS mount at every boot without making H200 boot or unrelated services depend on NAS availability.

Desired behavior:

```text
H200 boots
-> systemd registers /mnt/ywang-nas as an automount
-> boot and local services continue even if NAS is offline
-> first access triggers an SMB/CIFS mount attempt
-> an offline attempt times out after about 10 seconds
-> the share is unmounted after 10 minutes of inactivity
```

This is an on-demand automatic mount. The NAS is not necessarily mounted immediately during boot.

## 2. Known H200 and NAS Values

Verified on 2026-09-09:

```text
H200 systemd: 249
CIFS helper: /usr/sbin/mount.cifs
NAS share: //10.2.26.26/902_data
H200 mount point: /mnt/ywang-nas
ywang UID: 1012
ywang GID: 1012
credentials file: /etc/samba/credentials-ywang
mount unit: mnt-ywang\x2dnas.mount
automount unit: mnt-ywang\x2dnas.automount
```

Latest read-only verification found:

```text
/mnt/ywang-nas is mounted from //10.2.26.26/902_data as CIFS through systemd automount
/mnt/ywang-nas owner is ywang:ywang
/mnt/ywang-nas mode is 0700
/etc/samba/credentials-ywang owner is root:root
/etc/samba/credentials-ywang mode is 0600
```

## 3. Safety Rules

- Perform these commands interactively through `ssh H200`.
- Do not send the NAS password to an Agent or place it in chat, command arguments, logs, Git, or documentation.
- Do not change Docker, NVIDIA drivers, networking, firewall, kernel, APT, or unrelated systemd services.
- Do not restart `remote-fs.target` or unrelated services.
- Do not use forced or lazy unmount on a mount that may be in use.
- Keep Quantitize API/Web and other always-on services on local `/data3`.
- If any command gives an unexpected result, stop before editing the next file.

## 4. Log In and Recheck Preconditions

```bash
ssh H200
id ywang
systemd --version | head -n 1
command -v mount.cifs
mountpoint /mnt/ywang-nas || true
findmnt -rn -t cifs,nfs,nfs4 -o TARGET,SOURCE,FSTYPE,OPTIONS || true
```

Expected:

- `ywang` is UID/GID `1012:1012`.
- `mount.cifs` exists.
- `/mnt/ywang-nas` is not already mounted from an unexpected source.

Stop if UID/GID differ from `1012:1012`; the `/etc/fstab` line below must then be regenerated with the current numeric values.

## 5. Secure the Credential File

Correct ownership and permissions before doing anything else:

```bash
sudo chown root:root /etc/samba/credentials-ywang
sudo chmod 0600 /etc/samba/credentials-ywang
sudo stat -c 'owner=%U:%G mode=%a path=%n' /etc/samba/credentials-ywang
```

Expected:

```text
owner=root:root mode=600 path=/etc/samba/credentials-ywang
```

If the file is empty or contains incorrect credentials, edit it interactively:

```bash
sudoedit /etc/samba/credentials-ywang
```

File format:

```ini
username=orange
password=<enter the NAS password interactively>
```

Do not display the completed file with `cat`, `sed`, `grep`, or an Agent command.

## 6. Secure the Mount Point

```bash
sudo chown ywang:ywang /mnt/ywang-nas
sudo chmod 0700 /mnt/ywang-nas
stat -c 'owner=%U:%G mode=%a path=%n' /mnt/ywang-nas
```

Expected:

```text
owner=ywang:ywang mode=700 path=/mnt/ywang-nas
```

## 7. Back Up and Edit `/etc/fstab`

First confirm the NAS entry does not already exist:

```bash
grep -nF '//10.2.26.26/902_data' /etc/fstab || true
grep -nF '/mnt/ywang-nas' /etc/fstab || true
```

Stop if a conflicting entry exists. Do not add a duplicate.

Create a root-owned reference backup and open the file interactively:

```bash
sudo cp --preserve=all /etc/fstab /etc/fstab.pre-ywang-nas-20260909
sudoedit /etc/fstab
```

Add exactly one line:

```fstab
//10.2.26.26/902_data /mnt/ywang-nas cifs credentials=/etc/samba/credentials-ywang,vers=3.0,iocharset=utf8,uid=1012,gid=1012,file_mode=0600,dir_mode=0700,nosuid,nodev,noexec,_netdev,nofail,x-systemd.automount,x-systemd.idle-timeout=10min,x-systemd.mount-timeout=10s 0 0
```

Important option meanings:

- `_netdev`: treat this as a network filesystem.
- `nofail`: do not make boot success depend on NAS availability.
- `x-systemd.automount`: mount on first access.
- `x-systemd.mount-timeout=10s`: bound an offline mount attempt.
- `x-systemd.idle-timeout=10min`: unmount after inactivity.
- `nosuid,nodev,noexec`: reduce privilege and executable-file exposure.
- `noexec`: datasets, weights, logs, and backups work normally, but programs/scripts cannot be executed directly from NAS.
- Final `0 0`: disable dump participation and filesystem checks for this network share.

## 8. Validate Without Rebooting

Validate the complete fstab before asking systemd to use the new entry:

```bash
sudo findmnt --verify --verbose
```

Stop if it reports an error related to `/mnt/ywang-nas`.

Reload systemd and start only the generated automount unit:

```bash
sudo systemctl daemon-reload
sudo systemctl start 'mnt-ywang\x2dnas.automount'
systemctl is-active 'mnt-ywang\x2dnas.automount'
systemctl status --no-pager 'mnt-ywang\x2dnas.automount'
```

Expected automount status: `active`.

Trigger one bounded online access test:

```bash
timeout 15s stat /mnt/ywang-nas/.
findmnt -rn -T /mnt/ywang-nas -o TARGET,SOURCE,FSTYPE,OPTIONS
df -h /mnt/ywang-nas
```

Expected source and type:

```text
source: //10.2.26.26/902_data
type: cifs
```

Do not create dataset or backup directories until an Agent has completed the Step 8A read-only verification and the user has approved the destination.

## 9. NAS Offline and Recovery Behavior

Do not intentionally disable NAS, networking, or firewall on the shared H200 to test failure behavior.

If NAS is offline:

- H200 boot and unrelated local services should continue.
- Only a process that accesses `/mnt/ywang-nas` may wait up to the mount timeout and fail.
- Local source data and artifacts must remain untouched.

After NAS returns, if the mount unit remains failed:

```bash
sudo systemctl reset-failed 'mnt-ywang\x2dnas.mount'
timeout 15s stat /mnt/ywang-nas/.
findmnt -rn -T /mnt/ywang-nas -o TARGET,SOURCE,FSTYPE,OPTIONS
```

## 10. Service Isolation Requirements

- Do not add `/mnt/ywang-nas` as an unconditional bind mount or startup dependency of Quantitize API/Web.
- Start NAS-backed training or backup containers only after a mount preflight.
- Every NAS job must confirm that `/mnt/ywang-nas` is a real CIFS mount from `//10.2.26.26/902_data`, not an unmounted local directory.
- A NAS failure may fail the requesting job only; it must not restart Docker or unrelated services.
- Never delete a local source or artifact because a NAS transfer failed.

## 11. Controlled Reboot Acceptance

Perform this only during an approved reboot window. After reboot, before accessing the NAS path:

```bash
systemctl is-active 'mnt-ywang\x2dnas.automount'
systemctl is-active 'mnt-ywang\x2dnas.mount' || true
systemctl --failed --no-pager
```

The automount should be active. The mount itself may be inactive until first access; that is expected.

Then run:

```bash
timeout 15s stat /mnt/ywang-nas/.
findmnt -rn -T /mnt/ywang-nas -o TARGET,SOURCE,FSTYPE,OPTIONS
```

Confirm unrelated H200 services started normally.

## 12. Rollback

Before stopping a live mount, check whether any process is using it:

```bash
sudo fuser -vm /mnt/ywang-nas || true
```

If it is in use, stop and coordinate with the process owner. Do not force unmount.

Use `sudoedit /etc/fstab` to remove only the exact `/mnt/ywang-nas` line added by this procedure. Then:

```bash
sudo systemctl stop 'mnt-ywang\x2dnas.automount'
sudo systemctl stop 'mnt-ywang\x2dnas.mount' || true
sudo systemctl daemon-reload
```

Do not blindly overwrite `/etc/fstab` with the backup if unrelated entries were changed later. Use the backup only as a comparison reference.

## 13. Handoff to the Agent

After Sections 5–8 pass, tell the Agent:

```text
NAS automount configured. Execute Step 8A read-only verification.
Do not populate or back up data yet.
```

Step 8A is read-only. Step 8B data/environment backup still requires separate explicit authorization.

## 14. References

- [systemd.mount manual](https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html)
- [systemd.automount manual](https://www.freedesktop.org/software/systemd/man/latest/systemd.automount.html)
- `docs/agent/CONTEXT.md`
- `docs/reference/RECOVERY.md`
- `docs/train/agent-runbook-ywang-yolo-gray1-training-workflow.md`
