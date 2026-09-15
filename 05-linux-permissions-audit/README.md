# Linux File Permissions Audit — Least Privilege Enforcement

**Type:** Access control remediation | **Tool:** Linux (`ls -la`, `chmod`) | **Focus:** Auditing and correcting file/directory permissions against a least-privilege policy

## Scenario

A research team's project directory (`/home/researcher2/projects`) had never had its permissions formally reviewed. The task: inspect every file and directory, identify anything that violated least-privilege (specifically, nothing should be writable — or in one case, readable — by "others"), and remediate it with targeted `chmod` commands, including a hidden file and a directory whose contents also needed correcting.

## Step 1 — Inspect current permissions

```bash
ls -la /home/researcher2/projects
```

```
-rw-rw-rw- 1 researcher2 researcher2 1024 Aug 31 project_k.txt
-rw-r----- 1 researcher2 researcher2 1024 Aug 31 project_m.txt
-rw-rw-r-- 1 researcher2 researcher2 1024 Aug 31 project_r.txt
-rw-rw-r-- 1 researcher2 researcher2 1024 Aug 31 project_t.txt
-rw--w---- 1 researcher2 researcher2 512  Aug 31 .project_x.txt
drwx--x--- 2 researcher2 researcher2 4096 Aug 31 drafts
```

`ls -la` lists every file including hidden ones (leading `.`) with the full permission string, owner, and group.

## Step 2 — Read the permission string

The string is one type character plus three permission triads:

- Character 1: file type — `-` file, `d` directory, `l` symbolic link.
- Characters 2–4: **owner** permissions (read/write/execute).
- Characters 5–7: **group** permissions.
- Characters 8–10: **others** (world) permissions.

Reading the initial state against a least-privilege policy immediately surfaces the problems:

| File | Permissions | Issue |
|---|---|---|
| `project_k.txt` | `-rw-rw-rw-` | "Others" can write — should never happen on a shared project file |
| `project_r.txt` / `project_t.txt` | `-rw-rw-r--` | "Others" can read — too broad for internal project files |
| `.project_x.txt` | `-rw--w----` | Hidden file, but group can write and the mode is inverted from policy (should be read-only for owner/group, nothing for others) |
| `drafts/` | `drwx--x---` | Group has execute (can enter/traverse) on a directory meant to be private drafts |

## Step 3 — Remediate

```bash
cd /home/researcher2/projects

chmod o-w project_k.txt          # remove write for others
chmod o-r project_r.txt          # remove read for others
chmod o-r project_t.txt          # remove read for others
chmod 440 .project_x.txt         # owner+group read-only, nothing for others
chmod -R 700 drafts              # owner-only, applied recursively to contents
```

## Step 4 — Verify

```
-rw-rw-r-- 1 researcher2 researcher2 1024 Aug 31 project_k.txt
-rw-r----- 1 researcher2 researcher2 1024 Aug 31 project_m.txt
-rw-rw---- 1 researcher2 researcher2 1024 Aug 31 project_r.txt
-rw-rw---- 1 researcher2 researcher2 1024 Aug 31 project_t.txt
-r--r----- 1 researcher2 researcher2 512  Aug 31 .project_x.txt
drwx------ 2 researcher2 researcher2 4096 Aug 31 drafts
```

| Command | Effect |
|---|---|
| `chmod o-w` | Removes only the write bit for "others," leaving owner/group access untouched |
| `chmod o-r` | Removes only the read bit for "others" |
| `chmod 440` | Sets an exact mode: owner read-only, group read-only, others nothing — appropriate for a hidden file that shouldn't be edited casually |
| `chmod -R 700` | Owner gets full control, group and others get nothing, applied recursively so nothing inside `drafts/` is left exposed |

## Why this matters for SOC work

Overly permissive file permissions are a recurring root cause in real incidents — a world-writable config file or a group-readable secrets folder is exactly the kind of low-effort finding that turns into a high-impact compromise. Reading a permission string correctly and knowing the precise `chmod` flag to fix *only* the bit that's wrong (instead of resetting the whole file and breaking legitimate access) is a base-level Linux skill every SOC analyst is expected to have, whether it's during an audit, a hardening review, or while investigating how an attacker got read access to something they shouldn't have.

## Summary

Every file and directory in the target path was brought back in line with least-privilege: no file grants write to "others," the hidden file is locked to read-only for owner/group, and the `drafts` directory is fully private to its owner. The before/after state was captured with `ls -la` for audit evidence.

---
*Part of a self-directed cybersecurity training program (Google Cybersecurity Professional Certificate). See the [main portfolio](../README.md) for other projects.*
