# mytop

A powerful MySQL and MariaDB monitoring tool for the command line, **mytop** displays real-time database server performance information similar to `top`.


## Current Version
**v2.0** - released on Feb 3rd, 2026

---

## I know mytop - just show me how to install it

1. Download mytop into /usr/local/sbin & make executable
```bash
wget -O /usr/local/sbin/mytop https://raw.githubusercontent.com/fevangelou/mytop/refs/heads/main/mytop && chmod +x /usr/local/sbin/mytop
```

2. Just run (the defaults work just fine on Debian 11+, Ubuntu 22.04+ and RHEL distros v7 or newer)
```bash
mytop
```

---

## ABOUT

This is a fork of the original mytop Perl script, updated for MySQL 8.x & MariaDB 10.3+ (or newer) releases.

It is not a half-baked todo-when-I-get-the-time fork.

As a day & night sysadmin, I use mytop... daily. Or at least, I did. My gradual switch to Ubuntu Server 24.04 left me with Innotop, which simply does not work right (especially when it comes to explaining queries - plus it never "clicked" as you had to configure it just for the basics you get from mytop out of the box - go figure). As such, forking mytop was the only sensible thing to do. And as long as I'm active as a sysadmin (17+ years already), I'll keep it updated.

Yes, it has been "vibe coded" with the help of Anthropic's Claude (Perl was never my domain), but I have tested it extensively and since it's a daily tool for me, I'm gonna keep on maintaining it, as well as examine a possible rewrite to amother language.

Sincere thanks to Jeremy D. Zawodny (original author) & Mark Grennan (updated it for MySQL 5.x).

*P.S. Whoever said AI is killing open source is wrong. Needs and ways have just shifted. Embrace the new tooling.*


## WHAT'S NEW

### MySQL 8.0 & MariaDB 10.3 (or newer) compatibility fixes

#### Query Cache Removal
- MySQL 8.0 completely removed the query cache feature
- Added intelligent detection that checks for `have_query_cache` variable
- Falls back to checking `Qcache_hits` status variable for older versions
- Safely defaults to disabled for MySQL 8.0 or newer

#### Safe Status Variables
- All `Qcache_*` status variable references now use null coalescing
- Prevents undefined variable warnings when query cache doesn't exist
- Cache hit ratio calculations won't crash on MySQL 8+

#### Enhanced EXPLAIN
- Wrapped EXPLAIN execution in eval block for proper error handling
- Catches non-explainable queries (DDL, some admin commands)
- Added support for new `filtered` column in MySQL 8+ EXPLAIN output
- Dynamic column detection for future MySQL versions

#### System Compatibility
- Fixed `/proc/loadavg` reading for non-Linux systems
- Added proper error handling with fallback to empty string

### User Experience Improvements

#### Dependency Checking
- Automatic detection of missing Perl modules (DBI, DBD::mysql, Term::ReadKey)
- Provides exact installation commands based on your distribution

#### Better Defaults
- Default database changed from `test` to `mysql`
- Refresh delay reduced from 5 seconds to 1 second
- Idle thread display threshold set to 2

#### Code Quality
- Fixed all Perl syntax warnings
- Renamed conflicting variables to avoid masking
- Fixed operator typo (`=>` to `>=` in cache ratio comparison)
- Code formatted with Perl Tidy

---

## INSTALLATION INSTRUCTIONS

### Prerequisites

mytop requires the following Perl modules:

#### On Debian 11+ or Ubuntu 22.04+
```bash
sudo apt install libdbi-perl libdbd-mysql-perl libterm-readkey-perl
```

#### On RHEL distros (v7 or newer)
```bash
sudo yum install perl-DBI perl-DBD-MySQL perl-TermReadKey
```
or

```bash
sudo dnf install perl-DBI perl-DBD-MySQL perl-TermReadKey
```

#### Via CPAN (for the purists)
```bash
cpan DBI DBD::mysql Term::ReadKey
```

### One-line installation

No need to run `make` and `make install` as with previous versions - mytop is a standalone Perl script:

```bash

# 1. Download & make executable
wget -O /usr/local/sbin/mytop https://raw.githubusercontent.com/fevangelou/mytop/refs/heads/main/mytop && chmod +x /usr/local/sbin/mytop

# 2. Just run (the defaults work just fine on Debian 11+, Ubuntu 22.04+ and RHEL distros v7 or newer)
mytop

```

---

## CONFIGURATION

### The .mytop Configuration File

When run as a root user, the defaults should work out of the box. Should you need to adjust them or allow regular system users to access mytop, create a config file at `~/.mytop` (either for root or for each user you want to allow using mytop) with your connection settings:

```bash
# Place at ~/.mytop - adjust as needed
host=localhost
port=3306
db=mysql
user=myuser
pass=mypassword
delay=1
socket=/var/run/mysqld/mysqld.sock
```

### Command Line Options

Run `mytop --help` to see all options:

- `-u <user>` - MySQL/MariaDB username (default: root)
- `-p <password>` - MySQL/MariaDB password
- `-h <host>` - MySQL/MariaDB server host (default: localhost)
- `-P <port>` - MySQL/MariaDB server port (default: 3306)
- `-S <socket>` - MySQL/MariaDB socket file
- `-d <database>` - Database to use (default: mysql)
- `-b` - Batch mode, print and exit
- `-s <seconds>` - Refresh delay in seconds (default: 1)
- `-i` - Idle mode - only show non-sleeping threads
- `-I` - Idle (big I) - mark idle threads with "." instead of full command
- `-o <field>` - Sort order (default: last_cputime)
  - `last_cputime` - Time spent
  - `user` - Username
  - `db` - Database name
  - `host` - Client host
  - `id` - Thread ID
  - `state` - Query state
  - `time` - Time in current state
- `-r` - Reverse sort order
- `--header` - Print header rows in batch mode
- `--color` - Force color on (even in batch mode)
- `--nocolor` - Disable color output

### Interactive Commands

Once mytop is running, use these keys:

- `?` - Display help
- `!` - Force past a replication error (at your own risk)
- `c` - Switch to command summary mode
- `C` - Toggle color display on/off
- `d` - Filter by specific database
- `E` - Display current replication error
- `e` - Explain a query by thread ID
- `F` - Remove all filters
- `f` - Show full query info for a thread
- `h` - Filter by hostname
- `H` - Toggle header display
- `i` - Toggle idle (sleeping) thread display
- `I` - Show InnoDB status
- `k` - Kill a specific thread
- `K` - Kill all threads owned by a specific user
- `l` - Change long running query highlighting threshold
- `m` - Switch to QPS (queries per second) mode
- `o` - Reverse sort order
- `p` - Pause display updates
- `q` - Quit
- `r` - Reset status counters (FLUSH STATUS)
- `R` - Toggle reverse DNS lookup for IP addresses
- `s` - Change refresh delay in seconds
- `S` - Change slow query highlighting threshold
- `t` - Filter by thread state
- `u` - Filter by specific user
- `V` - Show MySQL/MariaDB variables

### Display Information

mytop shows:

1. **Header Section**: MySQL/MariaDB uptime, queries per second (QPS), load average, query cache hit rate (if available)
2. **Active Threads**: Currently running queries with:
   - Thread ID
   - Username
   - Host
   - Database
   - Time
   - Command/State
   - Query text
3. **Command Summary** (mode 'm'): Aggregated count by command type

```bash
mytop --user root --password secret --host localhost --db mysql
```

---

## COMPATIBILITY MATRIX

| Database Version | Status |
|-----------------|--------|
| MySQL 5.x | ✅ Full support (with query cache) |
| MySQL 8.0+ | ✅ Full support (without query cache) |
| MariaDB 10.x | ✅ Full support |
| MariaDB 11.x | ✅ Full support |

| Operating System | Status |
|-----------------|--------|
| Linux | ✅ Full support |
| Other Unix | ✅ Full support (no load average) |
| macOS | ✅ Full support |
| Windows | ⚠️ Seriously? |

---

## USAGE

### Basic Usage

```bash
# Connect to local MySQL/MariaDB with default settings
mytop

# Connect to remote server
mytop --host db.example.com --user admin --password secret

# Use a specific database
mytop --db information_schema

# Batch mode (non-interactive)
mytop --batch
```

---

## CHANGELOG

*This changelog provides a historical record of all previous versions.*

### Version 2.0 - February 3rd, 2026
MySQL 8.0 & MariaDB 10.3 or newer compatibility patches

- Fixed query cache detection for MySQL 8.0+ (query cache was removed)
- Added fallback detection for query cache based on Qcache_hits status variable
- Made all Qcache status variable references safe with null coalescing
- Fixed /proc/loadavg reading to handle non-Linux systems gracefully
- Enhanced EXPLAIN error handling to catch non-explainable queries
- Updated PrintTable to handle additional MySQL 8+ EXPLAIN columns (filtered, etc.)
- Added dynamic column detection for future MySQL version compatibility
- Improved error messages for EXPLAIN failures
- Made query cache hit ratio calculations safe when variables don't exist
- Updated default configuration (db: mysql, delay: 1s, idle: 2)
- Added Term::ReadKey to dependency checks
- Fixed all Perl syntax warnings and code quality issues

This version is fully compatible with:
- MySQL 5.x (with query cache)
- MySQL 8.0+ (without query cache)
- MariaDB 10.3+
- On Debian 11+, Ubuntu 22.04+ and RHEL distros v7+

Version 2.0 was based off v1.91 from Mark Grennan.

### Version 1.9.1 - February 14, 2012 (Mark Grennan)
- Added display of system load level on the top line
- Added Readonly state and Delay to the replication line
- Added monitoring of Slave status
- New command '!' to force past a replication error
- Added number of rows sorted per second
- Added a new 'Cmd' column to display the State of the query along with the statement
- New command 'M' to change [mode] to status show changes

### Version 1.8 - May 21, 2010 (Mark Grennan)
- Added fixes for changes in MySQL 5.1 and 64-bit systems
- Completed TODO - "make 'C' toggle display color"
- New command 'S' command: Highlights slow queries less than 'S' seconds (default: 10 seconds)
- New command 'l' for long queries (default: 120 seconds) - colored Magenta
- New command 't' to filter based on State
- Removed the 'Daemon' command type from the idle display
- Fixed documentation errors

### Version 1.6 - February 16, 2007
- Added support for passwords with embedded spaces (patch from Jeffrey Friedl)

### Version 1.4 - August 3, 2003
- Used junk query filter from Steven Roussey
- Added -prompt option so password doesn't need to be on command-line or in script (patches from jon r. luini and Sean Leach)
- Added -resolve support to convert IPs to hostnames in thread view (patch from Yogish Baliga)
- Used excellent patch from Per Andreas Buer to tidy up the top display
  - Shows most values in short form (10k rather than 10000)
  - Fixed query cache hit % calculation bugs
- Strip port numbers from IP addresses in thread view
- Added "I" command to get SHOW INNODB STATUS output
- Added experimental support for reading options from [mytop] section of my.cnf or .my.cnf
- Various other cleanup and fixes

### Version 1.3 - April 17, 2003
- Added "c" command to switch between thread view and "command summary" view
- Command summary pulls Com_* values from SHOW STATUS
- Fixed various bugs
- Added regex support for filters
- Added ability to [K]ill all threads owned by a particular user
- Fixed query cache hit rate computation
- Use Com_select rather than Questions to calculate Qcache ratio (only SELECT queries are cache candidates)

### Version 1.2 - October 27, 2002
- Better handle MySQL server going offline (shouldn't die with illegal division by zero errors)
- Make Host column large enough to handle IP addresses (suggestion from Jeremy Tinley)
- Added query cache stats

### Version 1.1 - Not Released
- Updated option handling so both --pass and --password work (suggested by Paul DuBois)
- Noted that OS X is supported

### Version 1.0 - April 27, 2002
- Fixed cases when trying to remove domain name from display even if it's an IP address
- Fixed formatting bugs and "use of uninitialized value" errors
- Adjusted column widths and headings
- Added "Now/Sec" to header (real-time queries/sec since last refresh)
- Added 'o' key to toggle sort order
- Changed 'h' key to 'H' for toggling header
- Added 'h' key to filter based on hostname
- Changed "Query Info" column to "Query or State"
- Real-time queries/sec computed using Time::HiRes if available
- Added 'e' key to EXPLAIN a query
- Moved website to http://jeremy.zawodny.com/mysql/mytop/
- Created mailing list

### Version 0.9
- Set $0 to 'mytop' to make it easier to spot
- Included README in distribution

### Version 0.8
- Added "Queries Per Second" (qps) mode - press 'm' key
- Distributed as a true Perl package with Makefile.PL
- Can be installed via CPAN shell

### Version 0.7
- Basic support for Windows (screen clearing doesn't work)

### Version 0.6
- Minor code cleanup

### Version 0.5
- Fixed some field widths to handle larger values

### Version 0.4
- Added -P command-line argument (to specify port number)
- Fixes for better output on small terminals
- Added 'i' hotkey to toggle display of Idle (sleeping) threads
- Other minor updates

### Version 0.3
- Updated documentation to reflect command-line arguments
- Added support for long and short command-line arguments
- Optional color support via Term::ANSIColor
- Added batch mode (doesn't clear screen, outputs data once)
- Fixed division by zero bug in key cache code
- Implemented 'r' keystroke to reset status counters via FLUSH STATUS
- Adapts when xterm is resized
- Implemented 'p' keystroke to pause display
- Implemented 'h' keystroke to toggle header
- Implemented 'k' keystroke to kill a thread
- Fixed various formatting bugs
- Added more specific error messages from DBI on connect failure

### Version 0.2
- Added support for non-standard port numbers

---

## License

GNU General Public License

---

## Credits

### Core Contributors

- **Jeremy D. Zawodny** - Original author and maintainer (2000-2009)
  - http://jeremy.zawodny.com/mysql/mytop/
- **Mark Grennan** - MySQL 5.x updates and enhancements (2010-2012)
  - https://www.mysqlfanboy.com/mytop-3/
- **Fotis Evangelou (vibe coded with Claude)** - Updated to support MySQL 8.0 & MariaDB 10.3 or newer releases (2026+)

---

**mytop v2.0** - Keeping a classic MySQL/MariaDB monitoring tool alive for modern database versions.
