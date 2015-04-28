# mailqparser
A handy, dandy, tiny parser for the postfix mail queue

Turns the output of `mailq`/`postqueue -p`/`sendmail -bp` from this:
```
$ mailq
-Queue ID- --Size-- ----Arrival Time---- -Sender/Recipient-------
3F8C6D4AC616*   18059 Tue Mar 17 19:08:03  MAILER-DAEMON
                                           foo@bar.com
                                           friendoffoo@bar.com

252CED43D8C5   31669 Tue Mar 17 06:48:32  MAILER-DAEMON
                                          yoohoo@tootles.com

C5B34D4ACEFC*   35764 Tue Mar 17 19:10:33  somebodythatiusedtoknow@woot.com
(delivery temporarily suspended: lost connection with some.crappy.mailrelay.net while sending RCPT TO)
                                          god@heaven.io
```
Into something more useful for machine consumption, like this:

```
$ mailq | mailqparser
3F8C6D4AC616;MAILER-DAEMON;foo@bar.com,friendoffoo@bar.com
252CED43D8C5;MAILER-DAEMON;yoohoo@tootles.com
C5B34D4ACEFC;somebodythatiusedtoknow@woot.com;god@heaven.io
```

That is:

```
<queue id>;<sender>;<comma-delimited list of recipients>
```

Use the ```-p|--print-format``` and ```-d|--delim``` options to adjust how data is ultimately presented:

```
$ mailq | mailqparser -p id,size,datestring -d '|'
3F8C6D4AC616|18059|Tue Mar 17 19:08:03
252CED43D8C5|31669|Tue Mar 17 06:48:32
C5B34D4ACEFC|35764|Tue Mar 17 19:10:33
```

You can also filter output based on a provided regular expression:

```
$ mailq | mailqparser -r 'foo@.*'
3F8C6D4AC616;MAILER-DAEMON;foo@bar.com,friendoffoo@bar.com
3FDCAD4BC871;foo@baz.com;superfunguy@doodeedoo.net
```
* Use the ```-f <filename>``` option to toss it a newline-separated filer of patterns to match against any of them
* Use the ```-v``` flag to invert the match
* By default we'll match either the sender or recipient(s). Specify either to or from with the ```-m <fieldname>``` if you want to restrict this.

* Use the -t/--time arguments to filter based on message age

```
$ mailqparser --help
usage: mailqparser [OPTIONS]

optional arguments:
  -h, --help            show this help message and exit
  -p PRINT_FORMAT, --print-format PRINT_FORMAT
                        Comma-delimited list of output fields and order.
                        Fields available: id, from, to, datestring, size,
                        error. Default is "id,from,to"
  -d DELIM, --delim DELIM
                        Output field separator to use (default is a semicolon)
  -t TFILTER, --time TFILTER
                        Filter messages based on age (e.g. "-t=+1d" or "-t=-10m")
  -v                    Invert the match (like "grep -v")
  -m {any,from,to}, --match {any,from,to}
                        Match against sender or recipient (default any)
  -f PATTERN_FILE, --file PATTERN_FILE
                        Filename to use for patterns
  -r PATTERN, --regex PATTERN
                        Regular expression to match queue items against
```
