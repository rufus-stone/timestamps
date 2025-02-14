# Timestamps

A handy reference to some of the more common (and some less common) ways to represent timestamps that aren't just a variation on a normal human-readable datetime string.


## Contents

- [Unix Epoch](#unix-epoch1)
- [Windows FILETIME](#windows-filetime--active-directory-datetime2)
- [MS-DOS DateTime / FAT DateTime](#ms-dos-datetime--fat-datetime3)
- [WebKit / Chrome](#webkit--chrome6)
- [Apple CFAbsoluteTime](#apple-cfabsolutetime7)
- [Apple HFS+](#apple-hfs8)
- [Garmin FIT](#garmin-fit9)
- [NTP / SNTP](#ntp--sntp-10)


## Unix Epoch[^1]

* Number of seconds elapsed since **1970-01-01 00:00:00Z**
* Typically expressed as an integer, but also sometimes as a floating point value
* Sub-second representations are also common

### Examples and common variations

| Value               | Datetime             | Accuracy                                             |
|---------------------|----------------------|------------------------------------------------------|
| 1739442600          | 2025-02-13 10:30:00Z | Seconds                                              |
| 1739442600000       | 2025-02-13 10:30:00Z | Milliseconds                                         |
| 1739442600000000    | 2025-02-13 10:30:00Z | Microseconds                                         |
| 1739442600000000000 | 2025-02-13 10:30:00Z | Nanoseconds                                          |
| 1739442600.000      | 2025-02-13 10:30:00Z | Seconds as floating point with millisecond precision |


## Windows FILETIME / Active Directory Datetime[^2]

* Number of 100-nanosecond intervals elapsed since **1601-01-01 00:00:00Z**

### Examples

| Value              | Datetime             |
|--------------------|----------------------|
| 132223104000000000 | 2020-01-01 00:00:00Z |
| 133839460990000000 | 2025-02-13 18:48:19Z |

### Conversion to/from Unix Epoch seconds

```python
from datetime import datetime, timezone

'''
Conversion to/from Unix Epoch seconds may be subject to rounding errors
'''

def filetime2unix(value):
    return datetime(1601, 1, 1, tzinfo=timezone.utc).timestamp() + (value / 10000000)

def unix2filetime(value):
    return int((value - datetime(1601, 1, 1, tzinfo=timezone.utc).timestamp()) * 10000000)
```


## MS-DOS Datetime / FAT Datetime[^3]

* A packed 32-bit value with 2-second accuracy[^4]
* Year is given as an offset from **1980**
* Pay attention to byte order when working with these
* ZIP file headers use MS-DOS format to encode the file last modified timestamp[^5]

### Structure

| Bits  | Description                               |
|-------|-------------------------------------------|
| 0-4   | Seconds (divided by 2)                    |
| 5-10  | Minute (0-59)                             |
| 11-15 | Hour (0-23)                               |
| 16-20 | Day (1-31)                                |
| 21-24 | Month (January == 1, February == 2, etc.) |
| 25-31 | Year (offset from 1980)                   |

#### Visual Structure

         
          0                 1                 2                 3
          0 1 2 3 4 5 6 7   0 1 2 3 4 5 6 7   0 1 2 3 4 5 6 7   0 1 2 3 4 5 6 7
         +-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+
         |s|s|s|s|s|m|m|m| |m|m|m|h|h|h|h|h| |D|D|D|D|D|M|M|M| |M|Y|Y|Y|Y|Y|Y|Y|
         +-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+
         \________/\____________/\_________/ \________/\_________/\____________/
          second      minute       hour        day       month        year

### Examples

| Raw Bytes (Little Endian) | Value                        | Datetime            |
|---------------------------|------------------------------|---------------------|
| \xf4\x58\x83\x59          | 1501780212                   | 2024-12-03 11:07:40 |
| \xd8\x54\x4d\x5a          | 1515017432                   | 2025-02-13 10:38:48 |
| \xdc\x54\x4d\x5a          | 1515017436                   | 2025-02-13 10:38:56 |

### Conversion to/from Unix Epoch seconds

```python
from datetime import datetime, timezone

def dos2unix(value):
    year = ((value >> 25) & 0x7F) + 1980
    month = (value >> 21) & 0x0F
    day = (value >> 16) & 0x1F
    hour = (value >> 11) & 0x1F
    minute = (value >> 5) & 0x3F
    second = (value & 0x1F) << 1

    return int(datetime(year, month, day, hour, minute, second, tzinfo=timezone.utc).timestamp())

def unix2dos(value):
    dtg = datetime.fromtimestamp(value, tz=timezone.utc)
    return ((dtg.year - 1980) << 25
            | dtg.month << 21
            | dtg.day << 16
            | dtg.hour << 11
            | dtg.minute << 5
            | dtg.hour << 11
            | dtg.minute << 5
            | dtg.second >> 1)
```


## WebKit / Chrome[^6]

* Number of microseconds elapsed since **1601-01-01 00:00:00Z**
* Very similar to [Windows FILETIME](#windows-filetime--active-directory-datetime2), but only accurate to 1000 nanoseconds (i.e. 1 microsecond) compared to Windows FILETIME's 100 nanosecond accuracy

### Examples

| Value             | Datetime             |
|-------------------|----------------------|
| 13222310400000000 | 2020-01-01 00:00:00Z |
| 13383946099000000 | 2025-02-13 18:48:19Z |

### Conversion to/from Unix Epoch seconds

```python
from datetime import datetime, timezone

'''
Conversion to/from Unix Epoch seconds may be subject to rounding errors
'''

def webkit2unix(value):
    return datetime(1601, 1, 1, tzinfo=timezone.utc).timestamp() + (value / 1000000)

def unix2webkit(value):
    return int((value - datetime(1601, 1, 1, tzinfo=timezone.utc).timestamp()) * 1000000)
```


## Apple CFAbsoluteTime[^7]

* Number of seconds elapsed since **2001-01-01 00:00:00Z**
* Typically expressed as a floating point value with microsecond precision

### Examples

| Value            | Datetime             |
|------------------|----------------------|
| 729637877.045605 | 2024-02-14 21:11:17Z |
| 753421722.968995 | 2024-11-16 03:48:42Z |

### Conversion to/from Unix Epoch seconds

```python
from datetime import datetime, timezone

def cfabsolutetime2unix(value):
    return datetime(2001, 1, 1, tzinfo=timezone.utc).timestamp() + value

def unix2cfabsolutetime(value):
    return value - datetime(2001, 1, 1, tzinfo=timezone.utc).timestamp()
```


## Apple HFS+[^8]

* Number of seconds elapsed since **1904-01-01 00:00:00Z**
* Only capable of representing values between 1904-01-01 00:00:00Z and 2040-02-06 06:28:15Z
* Typically expressed as an integer
* Values do not account for leap seconds, but do include a leap day in every year that is evenly divisible by four

### Examples

| Value            | Datetime             |
|------------------|----------------------|
| 3203381950       | 2005-07-05 04:19:10Z |
| 4294967295       | 2040-02-06 06:28:15Z |

### Conversion to/from Unix Epoch seconds

```python
from datetime import datetime, timezone

def hfsplus2unix(value):
    return datetime(1904, 1, 1, tzinfo=timezone.utc).timestamp() + value

def unix2hfsplus(value):
    return value - datetime(1904, 1, 1, tzinfo=timezone.utc).timestamp()
```


## Garmin FIT[^9]

* Number of seconds elapsed since **1989-12-31 00:00:00Z**
* Typically expressed as an integer

### Examples

| Value            | Datetime             |
|------------------|----------------------|
| 938622559        | 2019-09-28 16:29:19Z |
| 987654321        | 2021-04-18 04:25:21Z |

### Conversion to/from Unix Epoch seconds

```python
from datetime import datetime, timezone

def garmin2unix(value):
    return datetime(1989, 12, 31, tzinfo=timezone.utc).timestamp() + value

def unix2garmin(value):
    return value - datetime(1989, 12, 31, tzinfo=timezone.utc).timestamp()
```


## NTP / SNTP [^10]

* A fixed point 64-bit value where the most significant 32 bits represent the number of seconds elapsed since **1900-01-01 00:00:00Z**, and the least significant 32 bits represent the fractional component of the second (approx. 0.2 nanosecond precision[^11])

### Examples

| Value                | Datetime                    |
|----------------------|-----------------------------|
| 14195914145254784551 | 2004-09-27 03:17:07.694744Z |
| 14195914386443616565 | 2004-09-27 03:18:03.850895Z |

### Conversion to/from Unix Epoch seconds

```python
from datetime import datetime, timezone

'''
Conversion to/from Unix Epoch seconds may be subject to rounding errors
'''

def ntp2unix(value):
    secs = value >> 32
    frac = value & 0xFFFFFFFF
    frac /= (1<<32)
    return datetime(1900, 1, 1, tzinfo=timezone.utc).timestamp() + secs + frac

def unix2ntp(value):
    secs = int(int(value) - datetime(1900, 1, 1, tzinfo=timezone.utc).timestamp())
    frac = value - int(value)
    frac *= (1<<32)
    return secs << 32 | int(frac)
```


[^1]:https://en.wikipedia.org/wiki/Unix_time
[^2]:https://learn.microsoft.com/en-us/windows/win32/api/minwinbase/ns-minwinbase-filetime
[^3]:https://learn.microsoft.com/en-us/windows/win32/sysinfo/ms-dos-date-and-time
[^4]:https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-dosdatetimetofiletime
[^5]:https://pkwaredownloads.blob.core.windows.net/pem/APPNOTE.txt
[^6]:https://github.com/chromium/chromium/blob/43c03c93ed7c80a400ebd3be9823681f306046ce/base/time/time.h#L7
[^7]:https://developer.apple.com/documentation/corefoundation/cfabsolutetime
[^8]:https://developer.apple.com/library/archive/technotes/tn/tn1150.html#HFSPlusDates
[^9]:https://developer.garmin.com/fit/cookbook/datetime/
[^10]:https://datatracker.ietf.org/doc/html/rfc2030#section-3
[^11]:https://tickelton.gitlab.io/articles/ntp-timestamps/
