# NotePlan Tools
`npTools.rb` is a Ruby script that adds funationality to the [NotePlan app](https://noteplan.co/). Particularly when run frequently, this provides a more flexible system of repeating tasks, allows for due dates to be expressed as offsets which allows for templates, and moves or files items from Daily files to Note files. It incorporates an ealier script to 'clean' or tidy up NotePlan's data files.

Each time the script runs, it:

**Tidies up** data files, by:
1. removing the time part of any `@done(...)` mentions that NotePlan automatically adds when the 'Append Completion Date' option is on.
2. removing `#waiting` or `#high` tags or `>dates` from completed or cancelled tasks (configurable)
3. removing any lines with just * or -
4. removing any trailing blank lines.

Moves any Daily note entries with a `[[Note link]]` in it to that note, **filing** them directly after the header section. In more detail:
- where the line is a task, it moves the task and any following indented lines (optionally terminated by a blank line)
- where the line is a heading, it moves the heading and all following lines until a blank line, or the next heading of the same level
- This feature can be turned off using the `-n` option.

Changes any mentions of **date offset patterns** (e.g. `{-10d}`, `{+2w}`, `{-3m}` to being scheduled dates (e.g. `>2020-02-27`), if it can find a DD-MM-YYYY date pattern in the previous markdown heading or previous main task if it has sub-tasks. This allows for users to define **templates** and copy and paste them into the note, set the due date at the start, and the other dates are then worked out for you.
- Valid intervals are specified as `[+][0-9][dwmqy]`. This allows for `d`ays, `w`eeks, `m`onths, `q`uarters or `y`ears.
- There's also the special case `{0d}` meaning on the day itself
- It also ignores offsets in a section with a heading that includes a #template hashtag.

| This example ...                                                                                                                                                                        | ... becomes                                                                                                                                                                                                 |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| \#\#\# Christmas Cards 25/12/2020<br />\* Write cards {-20d}<br />\* Post overseas cards {-15d}<br />\* Post cards to this country {-10d}<br />\* Store spare cards for next year {+3d} | \#\#\# Christmas Cards 25/12/2020<br />\* Write cards >2020-12-05<br />\* Post overseas cards >2020-12-10<br />* Post cards to this country >2020-12-15<br />\* Store spare cards for next year >2020-12-28 |
| \* Bob's birthday on 14/09/2020<br />&nbsp;&nbsp;\* Find present {-6d}<br />&nbsp;&nbsp;\* Wrap & post present {-3d} <br />&nbsp;&nbsp;\* Call Bob {0d}                                 | \* Bob's birthday on 14/09/2020<br />&nbsp;&nbsp;\* Find present >2020-09-08<br />&nbsp;&nbsp;\* Wrap & post present >2020-09-11<br />&nbsp;&nbsp;\* Call Bob >2020-09-14                                   |

**Creates new repeats** for newly completed tasks that include a `@repeat(interval)`, on the appropriate future date.
- Valid intervals are specified as `[+][0-9][dwmqy]`. This allows for `d`ays, `w`eeks, `m`onths, `q`uarters or `y`ears.
- When _interval_ is of the form `+2w` it will duplicate the task for 2 weeks after the date the _task was completed_.
- When _interval_ is of the form `2w` it will duplicate the task for 2 weeks after the date the _task was last due_. If this can't be determined, then it defaults to the first option.

<!-- In future, extending the **archiving** system. -->

## Running the Tools
There are 2 ways of running the script:
1. with no arguments (`ruby npTools.rb`), it checks all files updated in the last 24 hours. This is the way to use it automatically, running one or more times each day. (This is configurable by HOURS_TO_PROCESS below.)
2. with passed filename pattern(s), where it works on any matching Calendar or Note files. For example, to match the Daily file from 24/3/2020 give `ruby npTools.rb 20200324.txt`. It can include wildcard *patterns* to match multiple files, for example `202003*.txt` to process all Daily files from March 2020.

You can also specify options:
- `-h` for help, 
- `-a` (`--noarchive`) don't archive completed tasks into the ## Done section
- `-n` (`--nomove`) to turn off moving mentions of [[Note]] in a calendar day file to the [[Note]]
- `-s` (`--keepschedules`) keep the scheduled (>) dates of completed tasks
- `-v` for verbose output 
- `-w` for more verbose output

It works with all 3 storage options for storing NotePlan data: CloudKit (the default from NotePlan v3), iCloud Drive and Dropbox.

## Installation and Configuration
1. Check you have a working Ruby installation.
2. Install  two ruby gems (libraries) (`gem install colorize optparse`)
3. Download and install the script to a place where it can be found on your filepath (perhaps `/usr/local/bin` or `/bin`)
4. Make the script executable (`chmod 755 npTools.rb`)
5. Change the following constants at the top of the script, as required:
- `STORAGE_TYPE`: select whether you're using `iCloud` for storage (the default) or `CloudKit` (from v3.0) or `Drobpox`. If you're not sure, see NotePlan's `Sync Settings`.
- `USERNAME`: your macOS username
- `HOURS_TO_PROCESS`: will process all files changed within this number of hours (default 24)
- `NUM_HEADER_LINES`: number of lines at the start of a note file to regard as the header. The default is 1. Relevant when moving lines around.
- `TAGS_TO_REMOVE`: list of tags to remove. Default ["#waiting","#high"]

### Automatic running
If you wish to run this automatically in the background on macOS, you can do this using the built-in `launchctl` system. Here's the configuration file `jgc.npTools.plist` that I use to automatically run `npTools.rb` several times a day:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<!-- To load this:   launchctl load ~/Library/LaunchAgents/jgc.npClean.plist
     To unload this: launchctl unload ~/Library/LaunchAgents/jgc.npClean.plist -->
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>jgc.npClean</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ruby</string>
        <string>/Users/jonathan/bin/npClean</string>
    </array>
    <key>StartCalendarInterval</key>
    <array>
        <dict>
            <key>Hour</key>
            <integer>09</integer>
            <key>Minute</key>
            <integer>09</integer>
        </dict>
        <dict>
            <key>Hour</key>
            <integer>12</integer>
            <key>Minute</key>
            <integer>09</integer>
        </dict>
        <dict>
            <key>Hour</key>
            <integer>18</integer>
            <key>Minute</key>
            <integer>09</integer>
        </dict>
     </array>
    <key>StandardOutPath</key>
     <string>/tmp/jgc.npClean.stdout</string>
     <key>StandardErrorPath</key>
     <string>/tmp/jgc.npClean.stderr</string>
</dict>
</plist>
```
Update the filepaths to suit your particular configuration, place this in the `~/Library/LaunchAgents` directory,  and then run the following terminal command:
```
launchctl load ~/Library/LaunchAgents/jgc.npClean.plist
```

## TODO
See the [GitHub project](https://github.com/jgclark/NotePlan-tools) for issues, or suggest improvements.
