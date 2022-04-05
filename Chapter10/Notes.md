# Notes From ***Time***

- [Notes From ***Time***](#notes-from-time)
  - [Calendar Time](#calendar-time)
  - [Time-Conversion Functions](#time-conversion-functions)
    - [Converting `time_t` to Printable Form](#converting-time_t-to-printable-form)
    - [Converting Between `time_t` and Broken-Down Time](#converting-between-time_t-and-broken-down-time)
    - [Converting Between Broken-Down Time and Printable Form](#converting-between-broken-down-time-and-printable-form)
      - [Converting from broken-down time to printable form](#converting-from-broken-down-time-to-printable-form)
      - [Converting from printable form to broken-down time](#converting-from-printable-form-to-broken-down-time)
  - [Timezones](#timezones)
    - [Timezone definitions](#timezone-definitions)
    - [Specifying the timezone for a program](#specifying-the-timezone-for-a-program)
  - [Locales](#locales)
    - [Locale definitions](#locale-definitions)
    - [Specifying the locale for a program](#specifying-the-locale-for-a-program)
  - [Updating the System Clock](#updating-the-system-clock)
  - [The Software Clock (Jiffies)](#the-software-clock-jiffies)
  - [Process Time](#process-time)
  - [END](#end)

**Real time**: This is the time as measured either from some *standard point* (calendar time) or from some *fixed point* (typically the start) in the life of a *process* (elapsed or wall clock time).

**Process time**: This is the amount of CPU time used by a process.

## Calendar Time

Regardless of geographic location, UNIX systems represent time internally as a measure of seconds since the `Epoch`.

> Epoch: refers to midnight on the morning of *1 January 1970, Universal Coordinated Time*.  
> Fun fact: look for why *time_t* would be expired at *19 January 2038 03:14:07* on 32bit systems. You'll find some real fun stuff and meet new legends :).

The `gettimeofday()` system call returns the calendar time in the buffer pointed to by *tv*.

```c
int gettimeofday(struct timeval * tv , struct timezone * tz );
```

The `time()` system call returns the number of seconds since the Epoch.

```c
time_t time(time_t * timep );
```

---

## Time-Conversion Functions

### Converting `time_t` to Printable Form

The `ctime()` function provides a simple method of converting a time_t value into printable form.

```c
// output be like: Wed Jun 8 14:22:34 2011

char *ctime(const time_t * timep );
```

### Converting Between `time_t` and Broken-Down Time

The `gmtime()` and `localtime()` functions convert a `time_t` value into a so-called *brokendown time*.

> NOTE: The broken-down time is placed in a statically allocated structure whose address is returned as the function result.  
> Reentrant versions of these functions are provided as `gmtime_r()` and `localtime_r().`

```c
struct tm *gmtime(const time_t * timep );
struct tm *localtime(const time_t * timep );
```

The `mktime()` function translates a *broken-down time*, expressed as local time, into a `time_t` value,

```c
time_t mktime(struct tm * timeptr );
```

### Converting Between Broken-Down Time and Printable Form

#### Converting from broken-down time to printable form

`asctime()` returns a pointer to a statically allocated string containing the time in the same form as`ctime()`.

```c
char *asctime(const struct tm * timeptr );
```

The `strftime()` function provides us with more precise control when converting a broken-down time into printable form.

```c
size_t strftime(char * outstr , size_t maxsize , const char * format , const struct tm * timeptr );
```

#### Converting from printable form to broken-down time

The `strptime()` function is the converse of `strftime().` It converts a date-plus-time string to a broken-down time.

```c
char *strptime(const char * str , const char * format , struct tm * timeptr );
```

---

## Timezones

### Timezone definitions

The system maintains timezone information in files in standard formats.  
These files reside in the directory `/usr/share/zoneinfo`.

The local time for the system is defined by the timezone file `/etc/localtime`, which is often **linked** to one of the files in `/usr/share/zoneinfo`.

### Specifying the timezone for a program

To specify a timezone when running a program, we set the **TZ** *environment variable* to a string consisting of a *colon* ( : ) followed by one of the timezone names defined in `/usr/share/zoneinfo`.

---

## Locales

### Locale definitions

The system maintains locale information in files in standard formats, stored under `/usr/share/locale` directory (or `/usr/lib/locale` in some distributions).

These directories are named using the following convention:

```bash
language [_ territory [. codeset ]][@ modifier ]

# An example
de_DE.utf-8@euro # German language, Germany,
# UTF-8 character encoding, employing the euro as the monetary unit.
```

### Specifying the locale for a program

he `sewtlocale()` function is used to both **set** and **query** a program’s current locale.

```c
char *setlocale(int category , const char * locale );
```

## Updating the System Clock

`settimeofday()` and `adjtime()` update system clock, though these two interfaces are rarely used by application programs

The `settimeofday()` system call performs the converse of `gettimeofday()`: it sets the system’s calendar time to the number of seconds and microseconds specified.

```c
int settimeofday(const struct timeval * tv , const struct timezone * tz );
```

When making small changes to the time (of the order of a few seconds), it is usually preferable to use the `adjtime()` library function, which causes the system clock to **gradually** *adjust* to the desired value.

```c
int adjtime(struct timeval * delta , struct timeval * olddelta );
```

---

## The Software Clock (Jiffies)

The accuracy of various time-related system calls described in this book is **limited** to the resolution of the *system software clock*, which measures time in units called `jiffies`.

---

## Process Time

**Process time** is the amount of *CPU time* used by a process since it was created.

The kernel separates CPU time into the following two components:

- **User CPU** time is the amount of time spent executing in user mode. Sometimes referred to as *virtual time*.

- **System CPU** time is amount of time spent executing in kernel mode.

The `times()` system call retrieves process time information, returning it in the structure pointed to by *buf*.

```c
clock_t times(struct tms * buf );
```

The `clock()` function provides a simpler interface for retrieving the process time.

```c
clock_t clock(void);
```

## END
