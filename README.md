wail2ban
========

[![No Maintenance Intended](http://unmaintained.tech/badge.svg)](http://unmaintained.tech/)

![Saddest Whale](http://i.imgur.com/NVlsY.png "Saddest Whale")

wail2ban is a windows port of the basic functionality of [fail2ban](http://www.fail2ban.org/), and combining elements of [ts_block](https://github.com/EvanAnderson/ts_block).  This fork also includes elements inspired by [dosDeflate](https://github.com/jgmdev/ddos-deflate). 


overview
--------

wail2ban is a system that takes incoming failed access events for a customly configurable set of known event ids, and given sufficient failed attacks in a period of time, creates temporary firewall rules to block access. 


installation 
------------

Installing wail2ban is a case of a view simple tasks: 

 * copy all the repository files to a location on the client machine, e.g. `C:\scripts\wail2ban`
 * Using Task Scheduler, import the `start wail2ban onstartup.xml` file to automatically create a scheduled task to start the script when the machine boots. 
 * Initiate the script by running the `start wail2ban.bat` file. This is what the scheduled task starts. 

commandline execution
---------------------

wail2ban has `write-debug` things through it, just uncomment the `$DebugPreference` line to enable. This will output nice things to CMD, if running ad-hoc.

There are also a number of options that can be run against the script to control it: 
 
 * `-config` : dumps a parsed output of the configuration file to standard out, including timing and whitelist configurations. 
 * `-jail`   : shows the current set of banned IPs on the machine
 * `-jailbreak`: unbans every IP currently banned by the script. 
 * `-ban`: ban a provided IP address for 600 seconds, or as specified in -expire
 * `-unban`:  unban a provided IP address
 * `-parole`: manually check for expired IP blocks
 * `-dosDeflate X`: anually check IP volume over given value X. -trial paramater will show what would be banned. -expire paramater accepts seconds to ban for

technical overview 
------------------

Event logs for various software packages are configured to produce messages when failed connections occur. The contents of the events usually contain an IP, an a message something along the lines of "This IP failed to connect to your server."

Typical examples of this include: 

 * Security Event ID 4625, "Windows Security Auditing". 
  * `An account failed to log in. ... Source Network Address: 11.22.33.44`

Database products also include these kind of events, such as: 

 * Application Event ID 18456, "Microsoft SQL Server".
  *  `Login failed for user 'sa'. Reason: Password did not match that for the login provided. [CLIENT: 11.22.33.44]`

These events are produced any time someone mistypes a password, or similar. 

The issue occurs when automated brute-force entry systems attempt to access systems multiple times a second. 

dosDeflate manual mode
------------------

Wail2Ban can be run manually with the -dosDeflate paramater to block any IP address with over X number of active connections to the server.  

The -trial paramater allows you to preview what would have been blocked

Default block time period is 10 minutes, however -expire SECONDS can be passed to change the time period

`.\Wail2ban.ps1 -dosDeflate 20 -expire 86400 ` would block any IP with 20 open connections for 1 day.

If you are not running Wail2Ban as a scheduled task.  You can periodically run `.\Wail2ban.ps1 -parole` to manually cleanup the expired blocks. 

what wail2ban does
------------------

wail2ban is a real-time event sink for these messages. As messages come in, wail2ban takes note of the time of the attempt and the IP used in the attempt. Given enough attempts in a specific period of time, wail2ban will generate a firewall rule to block all access to the client machine for a certain period of time. 

In a default setup, if an IP attempts 5 failed passwords in a 2 minute period, they get banned from attempting again for a period of time.

How long? Well, that depends on how many times they've been banned before!

There is a file called BannnedIPLog.ini that will keep a count of how many times an IP has been banned. 

The punishment time is based on the function `y=5^x`, where x is the amount of times it has been banned, and y is the amount of minutes it's banned for. 

This allows for scaling of bans, but prevent permenant bans, which may cause issues in the future as IPs are reassigned around the blagosphere. 

There is also a `$MAX_BANDURATION` in place, which means that an IP cannot be banned for more than 3 months. Given the ban duration function gives values of years at the 10th increment, it's better to cap things out.

failsafes 
---------

As with all automated systems, there can be some false-positives. 

**Whitelists** - this script can be configured with a whitelist of IPs that it will never ban, such as a company IP block. 

**Self-list** - the script automatically adds a set of IPs to the whitelist that it knows as not to ban, based on the configured static IPs on the host machine. That is, it will ignore attempts from itself (or event logs which list it's own IP in the message). 

**Timeouts** - IPs are only banned for specific period of time. After this time, they are removed from the firewall by the script. The timeouts are parsed once a new failed attempt is captured by the system. This may mean that IPs are unbanned after their exact unlock time, but for sufficiently attacked systems, this difference is not a major issue.

**Jailbreak** - a configuration called `-jailbreak` can be run against the script at any time to immediately remove all banned IPs. All their counters are reset, and it is as if the IP never tried to attack the machine.




