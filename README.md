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

But wait, it can filter output based on a provided regular expression too!

```
$ mailq | mailqparser -r 'foo@.*'
3F8C6D4AC616;MAILER-DAEMON;foo@bar.com,friendoffoo@bar.com
3FDCAD4BC871;foo@baz.com;superfunguy@doodeedoo.net
```
* Use the -f <filename> option to toss it a file chock full of patterns to match against any of them
* Use the -v option to invert the match (you know you want to!)
* By default we'll match either the sender or recipient(s). Specify either to or from with the -m option if you want to restrict this.


```
$ mailqparser --help
usage: mailqparser [OPTIONS]

optional arguments:
  -h, --help            show this help message and exit
  -p, --print           Print messages matching the filter (default)
  -v                    Invert the match (like "grep -v")
  -m {any,from,to}, --match {any,from,to}
                        Match against sender or recipient (default both)
  -f PATTERN_FILE, --file PATTERN_FILE
                        Filename to use for patterns
  -r PATTERN, --regex PATTERN
                        Regular expression to match queue items against
```
