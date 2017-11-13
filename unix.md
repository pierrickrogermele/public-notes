UNIX
====

These notes refer to UNIX and Linux operating systems.

## Network

### wget

Download recursively:
```bash
wget -r http://some.site.fr/
```

```bash
wget -i blabla -o zop http://fsgjkbnkfjg.bgjnfdgb/dfbkjgn.xml
```

Quiet:
```bash
wget -q -O myfile.html http://jfvndfjvndf/fdj.html
```

### mail

Read mail from system mailbox:
```bash
mail
```

Send a mail to the system:
```bash
mail -s "my_subject" user.name
```
and then type your message body and finish by `ctrl+d`, or
```bash
echo "my text" | mail -s "my_subject" user.name
```

### nmap

Check that a port is open:
```bash
nmap -p 8180 mymachine
```

### Firewall & ports

Get port rules:
```bash
sudo iptables -L
```

If `ufw` (firewall) is running:
```bash
sudo ufw status
```

Under macos 10, for listing the rules:
```bash
sudo pfctl -s rules
```

### offlineimap

Synchronize locally with a IMAP account.
Can use a mail app to access local mail boxes.

Install on MacOS-X:
```bash
brew install offline-imap
```

 * [Folder filtering and Name translation](https://offlineimap.readthedocs.org/en/latest/nametrans.html).
 * [Use Mac OS X's Keychain for Password Retrieval in OfflineIMAP](https://blog.aedifice.org/2010/02/01/use-mac-os-xs-keychain-for-password-retrieval-in-offlineimap/).

For running offlineimap as daemon under Ubuntu, see `/usr/share/doc/offlineimap/examples`.

For running offlineimap as daemon under Ubuntu, run as root:
```bash
cat >/etc/systemd/user/offlineimap.service <<EOF
[Unit]
Description=OfflineIMAP Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/offlineimap
EOF
cat >/etc/systemd/user/offlineimap.timer <<EOF
[Timer]
OnUnitInactiveSec=120s
Unit=offlineimap.service
EOF
chmod a+r /etc/systemd/user/offlineimap.*
```
Then as a user:
```bash
systemctl --user daemon-reload
systemctl --user start offlineimap.timer
systemctl --user start offlineimap.service
```
See [OfflineIMAP: Periodically fetch emails with systemd's timers](https://kdecherf.com/blog/2012/12/23/offlineimap-periodically-fetch-emails-with-systemds-timers/) and [Integrating OfflineIMAP into systemd](http://www.offlineimap.org/doc/contrib/systemd.html).

### msmtp

Mail sender program, able to handle multiple SMTP accounts.

Using from command line:
```bash
echo "some text" | msmtp [-a account] email@address
```

In Mutt:
```bash
set sendmail="/usr/local/bin/msmtp"	# for normal msmtp
```

Using queuing in Mutt:
```bash
mkdir $HOME/.msmtp.queue
set sendmail="/usr/local/bin/msmtpq"	# for queueing version
```

Queued messages are sent later on when sending another email, when connected.
It's also possible to force sending with the following command:
```bash
msmtp-queue -r
```
Which means it could be used inside a cron task.

#### macOS install

msmtp exists in brew, but it doesn't install queueing scripts. However they are present in '/usr/local/Cellar/msmtp/<version>/share/msmtp/scripts/msmtpq/'.

### rsync
	
Use verbose flag to print processed files:
```bash
rsync -v ...
```

Synchronizing deletion of files in the source:
```bash
rsync -av --delete teddy:/home/data/documents .
```

Specify remote shell to use:
```bash
rsync -e ssh
```
or
```bash
export RSYNC_RSH=ssh
rsync ...
```
	
ERROR "rsync: failed to set times on":  add -O option, it tells rsync to omit directories when preserving times.
```bash
rsync -O ...
```
	
Compress:
```bash
rsync -z ...
```
	
Recurse in sub-directories:
```bash
rsync -r ...
```
	
Exclude files:
```bash
rsync --exclude '*~' ...
rsync --exlude '*~' --exlude '.DS_Store' --delete-excluded ... # also delete excluded files from dest dirs
rsync --exclude-from=FILE # read include patterns from FILE
```

To synchronize two directories:
```bash
rsync --delete /.../folder1/ /.../folder2
```
The slash at the end of the source path tells to synchronize all files, it makes --delete works for folder1.
	
Print the progress of transfer:
```bash
rsync --progress		
```
	
Regular sync (revursive, links, times), doesn't synchronize the permissions:
```bash
rsync -rlt ...
```

### ssh
	
Generate private and public keys:
```bash
ssh-keygen 
```

Login without password. In /etc/ssh/sshd_config:
```
RSAAuthentication yes
PubkeyAuthentication yes
```
Then:
```bash
ssh-copy-id -i ~/.ssh/id_dsa.pub username@remotebox
```

Using local tunneling for accessing mail server:
```bash
ssh -L 1993:imap.mail.me.com:993 -g -N server.addr
```
Local port used is 1993. Everything going to this local port will be forwarded to imap.mail.me.com:993 through server.addr:22.

### NTP, Network Time Protocol

 * [NTP configuration on Debian](https://wiki.debian.org/NTP).
 python3=$python36

### Wake On LAN

On Debian, see <https://wiki.debian.org/WakeOnLan>.

## System

### OS info

Get Ubuntu version:
```bash
lsb_release -a
```

Get system information on Ubuntu:
```bash
sudo lshw
sudo lshw -C network # Filter on network components
```

### Machine name

Under macOS, to set the name of a machine, run:
```bash
sudo scutil --set HostName lucy
sudo scutil --set LocalHostName lucy
sudo scutil --set ComputerName lucy
```

### Run levels

 * [Run levels](http://www.tldp.org/LDP/sag/html/run-levels-intro.html).

To get current level:
```bash
runlevel
```

Change run level:
```bash
init 5
```

The booting run level is defined inside the file `/etc/inittab`.

Services starting scripts are located inside `/etc/rc.d/init.d`.
They are run according to the runlevel. A symbolic is created inside the relevant runlevel folder for a service that we want to stop or start.
The runlevel folder is of the form `/etc/rc.d/rc?.d`, where `?` is replaced the runlevel number.
The name of the symbolic link begin by `K` for stopping the service and `S` for starting it, and is followed by a number. The number allows to sort the services in the order we want them to stop or to start.

LOGIN

Remove login manager:
```bash
update-rc.d -f gdm remove
```

Restore login manager:
```bash
update-rc.d -f gdm defaults
```


### Users and groups

```bash
usermod -a -G somegroup someuser
```
### date

Convert a date to Epoch time:
```bash
date -j '01171200' +%s # BSD
```

Flag        | Description
----------- | -----------------------
`-j`        | Do not try to set date.
`-f '...'`  | set another format for input date. Default is [[[mm]dd]HH]MM[[cc]yy][.ss] (mm=month, cc=century).


### Hardware

Get list of PCI devides in Debian:
```bash
lspci
```

Get list of USB devices in Debian:
```bash
lsusb
```

#### iPad/iPhone

##### Charging

For charging an iPad under Linux, see [Charge iPad / iPhone 4s / iPod Touch in Ubuntu](http://ubuntuguide.net/charge-ipad-iphone-4s-ipod-touch-in-ubuntu).
Under Ubuntu:
```bash
sudo apt-get install libusb-1.0-0 libusb-1.0-0-dev git git-core
git clone https://github.com/mkorenkov/ipad_charge.git
cd ipad_charge
make
sudo make install
```

Then to start charging connected devices:
```bash
ipad_charge
```

And to stop charging connected devices:
```bash
ipad_charge -0
```

#### Keyboard

 * [Configure Apple keyboard under Debian](https://wiki.debian.org/MacBook#Keyboard).

To see current keyboard configuration under Ubuntu:
```bash
localectl status
```

To configure the keyboard for the session only, run:
```bash
sudo loadkeys us
```
or for French AZERTY keyboard:
```bash
sudo loadkeys fr
```

To configure keyboard permanently under CentOS:
```bash
sudo system-config-keyboard
```

To configure keyboard permanently under Ubuntu, it will open an X window:
```bash
sudo dpkg-reconfigure keyboard-configuration 
```
Under Ubuntu, to configure permanently from command line, and make it also available at boot time (i.e.: at login console), edit the file `/etc/rc.local` and put the line `loadkeys fr` just before the line `exit 0`. `fr` is for french keyboard layout, replace it with whatever keyboard layout you want.

### Services

#### Ubuntu

List all services:
```bash
service --status-all
```

## File system

### mktemp

 * GNU version replaces Xs in template.
 * BSD version doens't use the Xs and appends its own random string.
 
We can test if we're running BSD or GNU version
```bash
test_template=tmp.XXXXXX
bsd_version=$(mktemp -u -t $test_template | grep '/$test_template')
if [ -n "$bsd_version" ] ; then
	options="-t $prefix"
else
	options="-t $prefix.XXXXXX"
fi
tmp_file=$(mktemp $options)
```

Or always set XXXXX template:
```bash
tmp_file=$(mktemp -t tmp.XXXXXX)
```

### basename

Return filename portion of pathname:
```bash
basename /my/path/to/a/file
```

### dirname

Return directory portion of pathname:
```bash
dirname /my/path/to/a/file
```

### du

To measure disk usage of a folder:
```bash
du -shc <folder>
```

See also:
```bash
ncdu <folder> # NCurses version of du
```

### file

Gives MIME type of a file using among other things the magic number stored in a magic list (see man file).

Prints human readable string:
```bash
file <file>
```

Prints only the type:
```bash
file -b --mime-type <file>
```

The `file` command also prints the character encoding. 

### find

Look for files containing a specified string in their name, starting from the root:
```bash
find / -name "gnomeprint"
```

Use regexp:
```bash
find / -regex <pattern>
find / -iregex <pattern> # case insensitive
find -E / -regex <pattern> # use extended regular expressions
```

Deleting found files:
```bash
find documents/ -iname '*~' -delete
```

Execute a command for each found file:
```bash
find <dir> -iname '*.txt' -exec printf '{}' \;
find . -iname '*.todo' -exec perl -e '$f="{}"; ($g=$f) =~ s/\.todo$/ N.txt/; printf "$f --> $g\n"; system("cat \"$f\" >>\"$g\"");' \;
```

Execute some code for each found file:
```bash
find . | while read f ; do 
	echo Found: $f
done
```

Levels of search
```bash
find . -maxdepth 1
```
level 0 tells to look only among the command line arguments
```bash
find R-* -maxdepth 0 -type d # return the list of all directories beginning wiht R-*
```

Filter on time:
```bash
find <dir> -newermt 2009-10-15
```
Find all files whose modification type is more recent than the specified date.

Filter on type:
```bash
find <dir> -type <type>
```

Type | Description
---- | -----------
b    | Block special.
c    | Character special.
d    | Directory.
f    | Regular file.
l    | Symbolic link.
p    | FIFO.
s    | Socket.

Operators:

Operator       | Description
-------------- | -----------
-false         | False.
-true          | True.
! expr         | Logical not.
-not expr      | Logical not.
expr expr      | Logical and.
expr -and expr | Logical and.
expr -or expr  | Logical or.
( expr )       | Grouping.

Filter on permissions:
```bash
find <dir> -perm -u+x       # Standard notation. 
find <dir> -perm +0422      # Octal notation.
find . -type f -perm 700    # Search for executable files.
```
Permission prefix | Decription
----------------- | ------------------------
-                 | All bits are set.
NOTHING           | Bits match exactly the file mode.
+                 | Any of the bist is set. Be careful with this + sign because it can be interpreted as a mode (not clear why for me, TODO).

Permissions GNU notation:
```bash
find <dir> -perm /u=x
```

Print output with \0 separator for results instead of \n:
```bash
find ... -print0
```

### chmod

When the sgid bit is set on a directory, all files & dirs created in this directory will automatically have the same group as the directory:
```bash
chmod g+s mydir
```

Set execute/search flag for directories only:
```
chmod -R g+X mypath
```

### umask

`umask` set the default permissions for new files.

## Compression and uncompression

### tar

To tar files without the parent directory, first change to that dir:
```bash
tar -cjf my_pkg.tbz -C my_dir .
```
The `-C <dir>` option, change to the specified directory.

Incremental backup (GNU only):
```bash
tar -cf /backup/my_dir.0.tar -g /backup/my_dir.snar /my/dir
tar -cf /backup/my_dir.1.tar -g /backup/my_dir.snar /my/dir
tar -cf /backup/my_dir.2.tar -g /backup/my_dir.snar /my/dir
```
Best practice is to do one full dump each week, and then a level 1 backup (incremental backup from the full dump) each day.
Note the option `--no-check-device`, which tells `tar` to do not rely on device numbers when preparing a list of changed files for an incremental dump. Device numbers are not reliable with NFS.
To restore an incremental backup:
```bash
tar -xf /backup/my_dir.0.tar -g /dev/null
tar -xf /backup/my_dir.1.tar -g /dev/null
tar -xf /backup/my_dir.2.tar -g /dev/null
```

Use xz:
```bash
tar -cJf mydir.tar.xz mydir
```

Extract in another directory:
```bash
tar -xzf my.file.tar.gz -C my/dest/dir
```

### zip, unzip

To compress a folder:
```bash
zip my_zip_file file1 file2 ...
zip -r my_dir my_dir # recursive
zip -j foo foo/* # to leave off the path
```

To uncompress:
```bash
unzip myfile.zip
```

List files contained inside a zip:
```bash
unzip -l myfile.zip
```

Unzip quietly:
```bash
unzip -q myfile.zip
unzip -qq myfile.zip
```

## File filtering

### tr

Set in uppercase:
```bash
tr '[:lower:]' '[:upper:]'
```

Removing all carriage returns:
```bash
tr -d '\r'
```

### cat

For printing special characters (carriage returns, ...):
```bash
cat -vte
```
Display ^M for the carriage return, and $ for the newline.

### sort

Sorting inplace:
```bash
sort myfile -o myfile
```

In MacOS-X, sort will not recognize unicide in UTF-8 files.
One must use iconv first to convert file into UTF8-MAC format:
```bash
iconv -t UTF8-MAC myfile.txt | sort
```

### split

Split a big file into several one giga bytes files:
```bash
split -b 1G myfile.bin myfile
```

### join

Join two files in a way similar to relational databases:
```bash
join a.txt b.txt >c.txt
```
By default `join` uses the first column as the join field.

Set the join fields, using column 3 in the first file and column 4 in the second file:
```bash
join -1 3 -2 4 a.txt b.txt >c.txt
```

### cut

To cut a portion of text of each line:
```bash
cut -c 3-5,10-20 <my_file
```
Take only characters 3 to 5 and 10 to 20.

Select fields separated by a character:
```bash
cut -d ',' -f 1,3,4 <my_file
```

### colrm
Remove columns from a file, where a column means a character column.

Remove all characters from 5 to end:
```bash
colrm 5
```

Remove all characters from 5 to 10:
```bash
colrm 5 10
```

### fold

To wrap lines to 80 columns:
```bash
fold -w 80 <my_file
```

To wrap at last space character:
```bash
fold -s -w 80 <my_file
```


### grep

Specifying multiple expressions:
```bash
grep -e toto -e titi # will match all lines of files that match either toto or titi
```

Displaying list of matching or non-matching files:
```bash
grep -l -e resto * # display matching files
grep -L -e resto * # display non-matching files
```

Labelling standard input:
```bash
gzip -cd foo.gz | grep --label=foo something
```

Print filename for each match:
```bash
grep -H toto *
```

Print line number for each match:
```bash
grep -n toto *
```

Print context (some lines before and/or after):
```bash
grep -A 2 toto * # 2 lines after
grep -B 2 toto * # 2 lines before
grep -C 2 toto * # 2 lines before and 2 lines after
```

Exclude files:
```bash
grep --exclude=*.class
```

Select files (include only specified files):
```bash
grep --include=*.java
```

Print only n lines:
```bash
grep -m <n> ...
```

Ignore binary files:
```bash
grep -I ...
```

### paste

Paste two or more files side by side:
```bash
paste a.txt b.txt c.txt >d.txt
```
By default, replace new line chars in a.txt and b.txt by tabulation.

### sed

To remove spaces at the end of a line:
```bash
sed -re 's/\s+$$//'
```

In shells, the `$` sign must be doubled in roder to do not be confused with a variable:
```bash
sed -re 's/\r$$//'
```

To remove carriage returns at the end of a line (Windows text file):
```bash
sed -re 's/\r$//'
```

To delete lines:
```bash
sed -re '/COUCOU/d'
```

To run multiple expressions in the same sed:
```bash
sed -re '...' -e '...' -e '...' ...
```

To edit in-place:
```bash
sed -e mycommand -i .bkp myfile # save backup with .bkp extension
sed -e mycommand -i '' myfile # no backup
```

To edit the first line only:
```bash
sed -e '1s/foo/bar/g' myfile
```

To add a carriage return to the end of a file:
```bash
sed -i '' -e '$a\' toto.txt
```

Print even lines:
```bash
sed -n 'n;p' filename
```

Print odd lines:
```bash
sed -n 'p;n' filename
```

Print only the lines inside a range of regex addresses:
```bash
sed -n /GOOGLE/,/ssl/p <~/.offlineimaprc
```

To insert a carriage return, you need to actually type it:
```bash
echo "MY LINE OF TEXT" | sed 's/ /\
/g'
```

### awk

#### Running

Run an Awk script:
```bash
awk -f myscript.awk
```

Run on a file:
```bash
awk '// { print $0 }' myfile
```

Passing variables:
```bash
awk -v var=value ...
```

Use a shebang:
```awk
#!/usr/bin/env awk -f
{ print $1 }
```

#### Records and rules

Rules apply on record (a record is a line, by default, depending of the definition of the record separator (`RS`)).

General form of a rule is `[pattern] [{ action }]`.

All rules are processed in order, one after the other.
The 'next' statement can be used to stop processing rules and pass to the next record.
```awk
/some regexp patterm/ { next }
```

Any expression can be used for a pattern, if it is true, the rule is executed:
```awk
expression {}
```

A pattern can be a regular expression:
```awk
/regular expression/ {}
```

Range pattern:
```awk
pat1, pat2 {}
```

Start and end patterns:
```awk
BEGIN { print "START !" }
END { print "DONE !" }
```

The null pattern matches every input record:
```awk
{print} 
```

A pattern can be negated:
```awk
! /regexp/
```

Patterns can be combined:
```awk
/regexp1/ && /regexp2/
/regexp1/ || /regexp2/
```

#### Regex

The match function can be used in a pattern expression to get captured group in a regular expression:
```awk
match($NF, /^([0-9]+)h$/, g) { time += g[1] }
```

Regexp on a particular field:
```awk
$2 ~ /ddd/
$3 !~ /aa/
```

#### Strings

Getting length:
```awk
{ l = length(s) }
```

Lowercase:
```awk
{ lc = tolower(s) }
{ uc = toupper(s) }
```

Split a string:
```awk
{ number_of_elements = split(string, array, fieldsep) }
```

#### Function

Declaration:
```awk
function name (parameters) {
	# body
}
```	
All other variables (global) are normally accessible from inside a function body, and thus can be modified.

The parameters contain : the arguments passed to the function AND the local variables declaration.

The parameters declared (arguments and local variables) hide the global variables of the same name.

The parameters that don't receive a value from the function caller, are set to the null string.

A common practice is to separate arguments and local variables declarations by spaces in the function header.

```awk
function abs(v) { return v < 0 ? -v : v }
```

#### Built-in variables

`FILENAME` is the name of the current file:
```awk
{print FILENAME;}
```

`NR` is the row number (from the start of input):
```awk
{if (NR!=1) {print}}
```

`NFR` is the row number inside the current file:
```awk
{print FILENAME, FNR;}
```

`NF` is the number of fields in a record (i.e.: row):
```awk
{print NF;}
```

Input field separator (space by default):
```awk
BEGIN{FS = "\t"}
```

Input record separator (end-of-line by default):
```awk
BEGIN{RS = ";\n"}
```

Output field separator (space by default):
```awk
BEGIN{OFS = "\t"}
```

Output record separator (end-of-line by default):
```awk
BEGIN{ORS = ";\n"}
```

#### Getting command line arguments

For Awk, the command line arguments are a list of files.
To get the filename currently being processed by Awk, use the built-in variable `FILENAME`:
```awk
{ print FILENAME }
```

Reading command line arguments:
```awk
BEGIN {
	for (i = 1; i < ARGC; i++)
		printf "%s ", ARGV[i]
	printf "\n"
}
```

The problem is that awk interprets command line arguments as files, and try to open them.
The only exception is when a argument is of the form `var=value`, in which case awk will set the variable `var` to the value "value".
A cleaner way to do this is to use the `-v` option:
```bash
awk -v var=value
```

#### Statements

##### If

```awk
{ if (NR == 3) { print } }
{ if (NR > 1 && ) { print } }
```

##### Switch
	
The switch statement is an optional statement for the GNU version of Awk, that must be included at compile time with the option --enable-swtich passed to configure script.

#### Removing columns

```awk
{$1=$2="";sub("  "," ")}1
```

Removing 8th column, using tabulation as a separator:
```awk
BEGIN{FS=OFS="\t"} {$8="";sub("\t\t","\t")}1
```

#### Print

Printing fields:
```awk
print a, b, c, d
```

Print fields and strings:
```awk
print $2":"$3":"$4" "$5"-"$6
```

Extract first column:
```bash
awk '{print $1}' spiid-inchi.txt >myids.txt
```

Remove duplicates on output:
```bash
awk '!x[$0]++'
```

#### Mathematical operations

Take absolute value:
```awk
function abs(v) { return v < 0 ? -v : v }
{ printf("%f\n", abs($1-$2)) }
```

Summing on a column:
```awk
{ sum += $2 }
END { print sum }
```

Taking the integer part of float:
```awk
{ i = int(f) }
```

Rounding a float to an integer:
```awk
{ i = int(f + 0.5) }
```

#### Arrays

In awk arrays are associative.
	
Looping on an (numerically) indexed array:
```awk
for (i = 1 ; i <= size_of_array ; ++i)
	print array[i]
```

Looping on array indexes:
```awk
for (k in array)
	print k
```

Sorting an array (the array must be numerically indexed):
```awk
{ n = asort(array) } # n is the number of elements of the array
{ n = asorti(array) } # case insensitive
```

## Power management

### pmset

BSD command.

Get battery information:
```bash
pmset -g batt
```

## tmux

Reload config file from within tmux:
```tmux
source-file ~/.tmux.conf
```

### Prefix

Prefix is by default `C-b`.

Set prefix:
```
unbind C-b
set -g prefix C-a
bind C-a send-prefix
```

### Attaching and detaching

Reattach an already attached session:
```bash
tmux attach -d
```

Command    | Description
---------- | ----------------------
`prefix d` | Detach.

### Windows & panes

Command                   | Description
------------------------- | ----------------------
`prefix d`                | Detach.
`prefix "`                | Split pane horizontally.
`prefix %`                | Split pane vertically.
`prefix arrow key`        | Switch pane.
`prefix {`                | Move pane to previous position.
`prefix }`                | Move pane to next position.
`prefix c`                | Create a new window.
`prefix n`                | Move to the next window.
`prefix p`                | Move to the previous window.

Resize the current pane down:
```
resize-pane -D
resize-pane -D 20
```
Use `-U` for upward, `-L` for left and `-R` for right.


Swap two windows:
```
swap-window -s 3 -t 1
```

Swap current window with a specific window:
```
swap-window -t 1
```

Split current pane and use window 4 as second pane:
```
join-pane -s 4
```

### Binding keys

Grabs the pane from the target window and joins it to the current:
```
bind-key j command-prompt -p "join pane from:"  "join-pane -s '%%'"
```

### Plugins

 * [Battery plugin](https://github.com/tmux-plugins/tmux-battery).

## `test` / `[`

File test operators:

Flag | Description
---- | -------------------
`-e` | File exists.
`-a` | File exists (deprecated).
`-f` | File is a regular file (not a directory or device file).
`-s` | File is not zero size.
`-d` | File is a directory.
`-b` | File is a block device.
`-h` | File is a symbolic link.
`-L` | File is a symbolic link.
`-r` | File has read permission (for the user running the test).
`-w` | File has write permission (for the user running the test).
`-x` | File has execute permission (for the user running the test).

```bash
if test -z "$var" ; then
	echo empty var
fi
```

You can use '[' instead of 'test'.
Be careful to put spaces before and after [ and ], because [ is a real command and ] is a real argument
```bash
if [ -z "$var" ] ; then
	echo empty var
fi
```

Compare integers
```bash
if [ $v -eq 1 ] ; then
	echo yesh
fi
```

## install

It has the interesting property of changing the ownership of any file it copies to the owner and group of the containing directory. So it automatically sets the owner and group of our installed files to root:root if the user tries to use the default /usr/local prefix, or to the user’s id and group if she tries to install into a location within her home directory.

Create directories:
```bash
install -d mydir
```

Install file:
```bash
install src_file dst_file
```

Install several files in a directory:
```bash
install src_file_1 src_file2 ... dst_directory
```

Setting mode (by default set to `rwxr-xr-x`):
```bash
install -m <MODE> ...
```

## mount

Monter un fichier iso en disque pour explorer son contenu :
```bash
mount -o loop file.iso <folder>
```

## xargs

Flag        | Description
----------- | -------------------------------
-I replstr  | Use a replace string, instead of appending arguments to the end of the command.

Execute the command with one argument at a time:
```bash
my_command | xargs -n 1 chmod a-x
```

To use \0 for strings separator instead of spaces or newlines:
```bash
... | xargs -0 ...
```

To place the arguments at a specified place instead of appending them, use the `-I` flag. The replace_string must be a distinct argument to xargs, it can't be inside a string for instance or it will not be replaced.
```bash
/bin/ls -1d [A-Z]* | xargs -I % cp -rp % destdir
```

## su & sudo

Login as root user and stay in current directory:
```bash
su
```

Login as root user inside root's home directory:
```bash
su -
```

Editing a file:
```bash
sudo -e /etc/some/system/filerc
```

Running a shell:
```bash
sudo -s
```

Running a command:
```bash
sudo <command>
```

## zenity

Opens a dialog box and possibly get user input.
```bash
zenity –question –title="Query" –text="Would you like to run the script?"
```

## crontab

To edit user's own crontab:
```bash
crontab -e
```

Environment variables already set by cron:

Variable    | Value
----------- | --------------------
`PATH`      | `/usr/bin:/bin`.
`HOME`      | From `/etc/passwd`.
`LOGNAME`   | From `/etc/passwd`.
`SHELL`     | `/bin/sh`.

To receive mails with output of run commands, you must set the `MAILTO` variable:
```crontab
MAILTO="username" # local mail
```
or
```crontab
MAILTO="someone@mail.server"
```

The format of a crontab line is `m h d M D command`, where 5 first fields define the time when to run the specified command.

Description of the five time fields, in order:

field        | allowed values
------------ | --------------
minute       | 0-59
hour         | 0-23
day of month | 1-31
month        | 1-12 (or names, see below)
day of week  | 0-7 (0 or 7 is Sun, or use names)

Ranges can be specified
```crontab
0 6-11 * * * command
```
Command will be run each day from 6 to 11 o'clock.

Several values or ranges can be separated by commas
```crontab
0 6,10,18 * * * command
```
Command will be run each day at 6:00, 10:00 and 18:00

Timesteps can be specified using a slash: `*/5`.

Run command each day at 9:00 and 21:00:
```crontab
0 9-23/12 * * * command
```

Run a command every 2 hours:
```crontab
0 */2 * * * command
```

## diff

To make a patch:
```bash
diff -ru <old sources> <new sources> >myfile.patch
diff -ruwNB old_v3 new_v3 > v3.patch
```

Flag | Description
---- | --------------------------------
`-w` | Ignore all white spaces.
`-B` | Ignore blank lines.
`-N` | Make patch also for new files.

Exclude files:
```bash
diff -x .c ...
```
It works for directory inside the path, for instance:
```bash
diff -x .git ...
```
Will exclude all files containing `.git` in the path, including `/.../.git/.../somefile`.

## patch

Patching a source tree:
```bash
patch -p1 < patch-file-name-here
```

Patching a file:
```bash
patch original_file patch_file
```

## seq

Generate a sequence of numbers:
```bash
seq 4 # --> 1 2 3 4
seq 100 104 # --> 100 101 102 103 104
```

## ls

BSD ls and GNU ls are different.
GNU ls command is part of the coreutils package.

Quoting names and escaping characters:
```bash
ls -Q
```

Print access time instead of last modification time:
```bash
ls -u *
```

In MacOS-X, a `@` character after permissions means that the file has additional attributes. You can see these additional attributes by typing the following command:
```bash
xattr -l <filename>
xattr -d <attrname> <filename> # to remove an attribute
```

In MacOS-X, a `+` character after permissions means that the file has an ACL, short for Access Control List, which is used to give fine grained control over file permissions, beyond what is available with the regular unix permission tables.
Entering the following command will show these additional permissions for files in a directory:
```bash
ls -le
```

`LSCOLORS` (see man ls), is an environment variable that defines the colors to be used by `ls` command. The default is "exfxcxdxbxegedabagacad", i.e. blue foreground and default background for regular directories, black foreground and red background for setuid executables, etc.

Color code | Descrption
---------- | -----------------------------------------------
a          |  Black.
b          |  Red.
c          |  Green.
d          |  Brown.
e          |  Blue.
f          |  Magenta.
g          |  Cyan.
h          |  Light grey.
A          |  Bold black, usually shows up as dark grey.
B          |  Bold red.
C          |  Bold green.
D          |  Bold brown, usually shows up as yellow.
E          |  Bold blue.
F          |  Bold magenta.
G          |  Bold cyan.
H          |  Bold light grey; looks like bright white.
x          |  Default foreground or background.

Note that the above are standard ANSI colors.  The actual display may differ depending on the color capabilities of the terminal in use.

The order of the attributes are as follows:

Rank | Description
---- | ---------------------------------
1    | Directory.
2    | Symbolic link.
3    | Socket.
4    | Pipe.
5    | Executable.
6    | Block special.
7    | Character special.
8    | Executable with setuid bit set.
9    | Executable with setgid bit set.
10   | Directory writable to others, with sticky bit.
11   | Directory writable to others, without sticky bit.

## hexdump

Tool for dumping a file content in hexadecimal format:
```bash
hexdump myfile
```

Display also ASCII characters:
```bash
hexdump -C myfile
```

## hexedit

Hexademical editor.

```bash
hexedit myfile
```

## dhex

Diff hexadecimal editor.

```bash
dhex myfileA myfileB
```

## ps

List every process (standard syntax):
```bash
ps -e
ps -ef # + user, full path
ps -eF # + user, fullpath, memory size
ps -ely
```

List every process (BSD syntax):
```bash
ps ax
```
Option          | Description
--------------- | ----------------------------------------------------
 -a             | Display also processes of all users.
 -x             | display also processes which do not have a controlling terminal.
 -U username    | Display the processes owned by the specified user.

## bc

Set the number of decimals to print for output:
```
scale=2
```
By default only integer parts are printed (zero decimals).

## calc

For printing a number in hexadecimal base:
```
base(16),134525
```

## expr

Measure length of a string:
```bash
expr 'abcd' : '.*'
```