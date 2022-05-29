# Notes From ***USERS AND GROUPS***

- [Notes From ***USERS AND GROUPS***](#notes-from-users-and-groups)
  - [The Password File: `/etc/passwd`](#the-password-file-etcpasswd)
  - [The Shadow Password File: `/etc/shadow`](#the-shadow-password-file-etcshadow)
  - [The Group File: `/etc/group`](#the-group-file-etcgroup)
  - [Retrieving User and Group Information](#retrieving-user-and-group-information)
    - [Retrieving records from the password file](#retrieving-records-from-the-password-file)
    - [Retrieving records from the group file](#retrieving-records-from-the-group-file)
    - [Scanning all records in the password and group files](#scanning-all-records-in-the-password-and-group-files)
    - [Retrieving records from the shadow password file](#retrieving-records-from-the-shadow-password-file)
  - [Password Encryption and User Authentication](#password-encryption-and-user-authentication)
  - [END](#end)

## The Password File: `/etc/passwd`

The system *password file*, `/etc/passwd` , contains one line for each user account on the system.

Each line is composed of *seven* fields separated by colons `:`.

for instance:

```conf
mtk:x:1000:100:Michael Kerrisk:/home/mtk:/bin/bash
```

- **Login name**: This is the unique name that the user must enter in order to log in. (**username**)

- **Encrypted password**: This field contains a 13-character encrypted password.

    > this field is ignored if **shadow** passwords have been enabled. then this field contains *letter x*.
- **UID**: This is the numeric ID for this user. If this field has the value **0**, then this account has **superuser** privileges.

- **GID**: This is the numeric ID of the first of the groups of which this user is a member.

- **Comment**: This field holds text about the user.

- **Home directory**: This is the initial directory into which the user is placed after logging in.

- **Login shell**: This is the program to which control is transferred once the user is logged in.

---

## The Shadow Password File: `/etc/shadow`

The **shadow password file**, `/etc/shadow`, was devised as a method of preventing to passwords be publicly visible.

checkout `shadow(5)` manual for more info.

---

## The Group File: `/etc/group`

For various administrative purposes, in particular, controlling access to files and other system resources, it is useful to organize users into **groups**. (identified by GID)

The group file, `/etc/group` , contains one line for each group in the system. Each line consists of *four* colon-separated fields.

for instance:

```conf
users:x:100:
jambit:x:106:claus,felli,frank,harti,markus,martin,mtk,paul
```

- **Group name**: This is the name of the group.

- **Encrypted password**: his field contains an *optional* password for the group.
    > If password shadowing is enabled, then this field is ignored (replaced by **x**)  and the encrypted passwords are actually kept in the *shadow group* file, `/etc/gshadow`

- **GID**: This is the numeric ID for this group.
    > There is normally one group defined with the group ID 0, named *root*.

- **User list**: This is a comma-separated list of names of users who are members of this group.

---

## Retrieving User and Group Information

### Retrieving records from the password file

The `getpwnam()` and `getpwuid()` functions retrieve records from the *password file*.

```c
struct passwd *getpwnam(const char * name );
struct passwd *getpwuid(uid_t uid );
```

### Retrieving records from the group file

```c
struct group *getgrnam(const char * name );
struct group *getgrgid(gid_t gid );
```

### Scanning all records in the password and group files

The `setpwent()`, `getpwent()`, and `endpwent()` functions are used to perform sequential
scans of the records in the password file.

```c
struct passwd *getpwent(void);

void setpwent(void);
void endpwent(void);
```

The `getgrent(),` `setgrent(),` and `endgrent()` functions perform analogous tasks for
the group file.

### Retrieving records from the shadow password file

The following functions are used to retrieve individual records from the shadow password file and to scan all records in that file.

```c
struct spwd *getspnam(const char * name );

struct spwd *getspent(void);

void setspent(void);
void endspent(void);
```

---

## Password Encryption and User Authentication

The only way of validating a candidate password is to encrypt it using the same method and see if the encrypted result matches the value stored in `/etc/shadow`.

The encryption algorithm is encapsulated in the `crypt()` function.

```c
char *crypt(const char * key , const char * salt );
```

---

## END
