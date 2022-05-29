# Notes From ***PROCESS CREDENTIALS***

- [Notes From ***PROCESS CREDENTIALS***](#notes-from-process-credentials)
  - [Real User ID and Real Group ID](#real-user-id-and-real-group-id)
  - [Effective User ID and Effective Group ID](#effective-user-id-and-effective-group-id)
  - [Set-User-ID and Set-Group-ID Programs](#set-user-id-and-set-group-id-programs)
  - [Saved Set-User-ID and Saved Set-Group-ID](#saved-set-user-id-and-saved-set-group-id)
  - [File-System User ID and File-System Group ID](#file-system-user-id-and-file-system-group-id)
  - [Supplementary Group IDs](#supplementary-group-ids)
  - [Retrieving and Modifying Process Credentials](#retrieving-and-modifying-process-credentials)
    - [Retrieving and Modifying Real, Effective, and Saved Set IDs](#retrieving-and-modifying-real-effective-and-saved-set-ids)
      - [Retrieving real and effective IDs](#retrieving-real-and-effective-ids)
      - [Modifying effective IDs](#modifying-effective-ids)
      - [Modifying real and effective IDs](#modifying-real-and-effective-ids)
      - [Retrieving real, effective, and saved set IDs](#retrieving-real-effective-and-saved-set-ids)
      - [Modifying real, effective, and saved set IDs](#modifying-real-effective-and-saved-set-ids)
      - [Retrieving and Modifying File-System IDs](#retrieving-and-modifying-file-system-ids)
      - [Retrieving and Modifying Supplementary Group IDs](#retrieving-and-modifying-supplementary-group-ids)
  - [END](#end)

## Real User ID and Real Group ID

The **real** user ID and group ID identify the user and group to which the process belongs.

---

## Effective User ID and Effective Group ID

The **effective** user ID and group ID, in conjunction with the *supplementary* group IDs, are used to determine the permissions granted to a process when it tries to perform various operations.

Normally, the *effective* user and group IDs have the **same** values as the corresponding **real** IDs, but there are two ways in which the effective IDs can assume different values.

- Use of system calls
- Execution of **set-user-ID** and **set-group-ID** programs

---

## Set-User-ID and Set-Group-ID Programs

A **set-user-ID** program allows a process to gain *privileges* it would not normally have, by setting the process’s *effective user ID* to the same value as the *user ID* (owner) of the executable file.

A **set-group-ID** program performs the analogous task for the process’s *effective group ID*.

---

## Saved Set-User-ID and Saved Set-Group-ID

The **saved** set-user-ID and **saved** set-group-ID are designed for use with set-user-ID and set-group-ID programs.

## File-System User ID and File-System Group ID

On Linux, it is the **file-system** user and group IDs, rather than the effective user and group IDs to determine permissions when performing file-system operations

Since the **file-system** IDs follow the *effective IDs* in this way, this means that Linux effectively behaves just like any other UNIX implementation when privileges and permissions are being checked.

> The file-system IDs differ from the corresponding effective IDs, and hence Linux differs from other UNIX implementations, only when we use two Linux-specific system calls, `setfsuid()` and `setfsgid()`, to **explicitly** make them different.

## Supplementary Group IDs

The supplementary group IDs are a set of additional groups to which a process
belongs.

These IDs are used in conjunction with the effective and file-system IDs to determine permissions for accessing files, System V IPC objects, and other system resources.

## Retrieving and Modifying Process Credentials

### Retrieving and Modifying Real, Effective, and Saved Set IDs

#### Retrieving real and effective IDs

The `getuid()` and `getgid()` system calls return, respectively, the *real user ID* and *real group ID* of the calling process.

The `geteuid()` and `getegid()` system calls perform the corresponding tasks for the *effective IDs*.

> These system calls are always successful.

```c
uid_t getuid(void);

uid_t geteuid(void);

gid_t getgid(void);

gid_t getegid(void);
```

#### Modifying effective IDs

The `setuid()` system call changes the effective user ID.

```c
int setuid(uid_t uid );

int setgid(gid_t gid );
```

A process can use `seteuid()` to change its **effective** user ID, and `setegid()` to change its **effective** group ID

```c
int seteuid(uid_t euid );

int setegid(gid_t egid );
```

#### Modifying real and effective IDs

The `setreuid()` system call allows the calling process to independently change the values of its **real** and **effective** user IDs. The `setregid()` system call performs the analogous task for the **real** and **effective** group IDs.

```c
int setreuid(uid_t ruid , uid_t euid );

int setregid(gid_t rgid , gid_t egid );
```

#### Retrieving real, effective, and saved set IDs

Linux provides two (nonstandard) system calls allowing us to retrieve (or update) saved set-user-ID and saved set-group-ID.: `getresuid()` and`getresgid().`

```c
int getresuid(uid_t * ruid , uid_t * euid , uid_t * suid );

int getresgid(gid_t * rgid , gid_t * egid , gid_t * sgid );
```

#### Modifying real, effective, and saved set IDs

The `setresuid()` system call allows the calling process to independently change the values of all three of its user IDs.

```c
int setresuid(uid_t ruid , uid_t euid , uid_t suid );

int setresgid(gid_t rgid , gid_t egid , gid_t sgid );
```

#### Retrieving and Modifying File-System IDs

All of the previously described system calls that change the process’s effective user or group ID also always change the corresponding file-system ID.

To change the file-system IDs independently of the effective IDs, we must employ two Linux-specific system calls: `setfsuid()` and`setfsgid()`.

  ```c
int setfsuid(uid_t fsuid );

int setfsgid(gid_t fsgid );
```

#### Retrieving and Modifying Supplementary Group IDs

The `getgroups()` system call returns the set of groups of which the calling process is currently a member.

```c
int getgroups(int gidsetsize , gid_t grouplist []);
```

A privileged process can use `setgroups()` and `initgroups()` to change its set of supplementary group IDs.

```c
int setgroups(size_t gidsetsize , const gid_t * grouplist );

int initgroups(const char * user , gid_t group );
```

---

## END
