
catstep
=======


Copyright (C) 2021 Kenneth Aaron.

flyingrhino AT orcon DOT net DOT nz

Freedom makes a better world: released under GNU GPLv3.

https://www.gnu.org/licenses/gpl-3.0.en.html

This software can be used by anyone at no cost, however, if you like
using my software and can support - please donate money to a
children's hospital of your choice.

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation:
GNU GPLv3. You must include this entire text with your distribution.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
See the GNU General Public License for more details.


Introduction
------------

- I wrote this program ( catstep: https://github.com/flyingrhinonz/catstep ) as
a solution to display terminal output that was captured with `tee` although
it can be used to view any text file.
- My other software - `nccm` ( ncurses ssh connection manager:
https://github.com/flyingrhinonz/nccm ) has an option
to pipe all ssh command output to `tee`, which results in a file that contains
a mixture of text and cursor/screen controls that's very hard to read in
a regular viewer. It does however display properly when you `cat` it to
screen but that flies away faster than you can read it.
`catstep` aims to fix this by allowing you to replay the file at a
controlled pace, or step through it manually.
- Output is sent to your screen, which requires the display size to be
identical or larger than the terminal size at capture time otherwise
output will probably be wrong.
- `catstep` is written only using standard library modules. While I could have
used a keyboard module that would have saved some coding, doing it this way
makes catstep usable in the widest range of scenarios without requiring the
user to add modules with pip3 (helpful in some work environments that block
access to external software sources such as pip3).


Manual install instructions
---------------------------

This is the easiest way, although you can simply download catstep and make it
executable...

- Clone the project from the git repository:
`git clone https://github.com/flyingrhinonz/catstep catstep.git`
- `cd catstep.git/`
- `sudo install -m 755 catstep -t /usr/local/bin/`


Command line arguments
----------------------

At it's simplest `catstep <file>` will get you going and put you into
interactive mode. Once in interactive mode - use the keyboard controls
listed in the next section.

file :
The file you want to display.

-a / --autoplay :
Starts catstep in autoplay mode. If not supplied, catstep will wait for
user input to control playback.

-d / --debug :
Log extra debugging information to syslog.

-e / --nomemtest :
catstep won't open a file if it is larger than half the available RAM.
Use this to bypass the memory test and open the file anyway.

-f / --fastforward <chars> :
Replay n chars at maximum speed before returning control to interactive mode.
This lets you speed through the file until the actual point you
want to view at a controlled speed.
This is used when catstep is started and applies to file offset 0.

-h / --help :
Displays the command line options.

-i / --info :
Displays information about the file before and after it is displayed.
This includes file name, number of lines and chars.

-m / --man :
Displays this man page.

-p / --periodic status <seconds> :
Logs the current position in the file every n seconds (default 10).
This can be really helpful if you speed past something and want to know where
you were a few seconds ago.
It logs even if you are not actively progressing through the file, but syslog
may suppress the logging of identical lines and subsequent lines may not
be logged.
Supply a value of 0 to explicitly disable periodic logging.
If you supply a value not 0, 5-60 it will force 10 seconds.
Does not run during the FastForward period.

-s / --sleep <seconds> :
Sleep <seconds> between chars output. Can be any number including
times less than 1 (eg: 0.5 , 0.001 , 2.3 , etc).
In interactive mode you can speed this up or slow it down.
If not supplied, the default value of 0.01 will be used.

-t / --smartsleep :
Sleep s seconds only for printable chars. Off by default. This yields a more
fluid playback when catstep encounters non-printable chars that would otherwise
also incur sleep.
Results will vary depending upon the type of chars in your file. Since catstep
works on a per-char basis, if you have control chars that at the single-char
level are printable - the sleep delay will be incurred.


Interactive mode controls
-------------------------

- 0-9 :             Play back 10^n chars where n is the key pressed.
                    0 plays back 1 char, 1 plays back 10 chars,
                    2 plays back 100 chars, etc.
- a :               Toggle auto play.
- d :               Turn on debug level logging. You can't turn it off again.
- f :               Make output faster by halving the sleep value.
- i :               Turn on info display at exit. You can't turn it off again.
- l :               Log settings, filename and position in file.
- q :               Quit catstep
- s :               Make output slower by doubling the sleep value.
- t :               Toggle SmartSleep


Limitations
-----------

- There's no step back option to 'replay backwards' because catstep
doesn't retain state - your display is updated as per the controls in
the file so the current state is what you see on your terminal. At first
glance this may seem like a killer limitation, but for that I added
the `l` interactive control which logs the position in the file of the
last char displayed.
If you need to check out what happened a short while ago (say the screen
was cleared), just hit `l`, view the log file to see the line you were at,
then restart catstep using the `-f` arg to fast forward the file a few
chars less. Then step through it slowly.
Also catstep periodically logs the current position, so you can use that
to get an idea of where you were in the file.

- With some files - especially those with lots of cursor controls, catstep may
display a summary char count lower than the actual file size.
This is the same count as `wc -m` gives. I have not seen catstep miss any
output display so it could be that certain cursor control chars don't get
counted. If anyone experiences problems or can narrow it down to particular
cases please report this in the issues.

- I've seen some terminal output (such as refreshes of the menus in
raspi-config running under screen when captured by `nccm` using `tee`) breaks
up horizontally - which BTW also happens in linux `cat` too.
But it's not unusable and you can easily read the output. This is possibly not
caused by `catstep` or `cat`, but a consequence of the `tee`. I have not
investigated this yet.

- The entire file is read at once into catstep. Normally this shouldn't be a
problem and you'll have plenty of RAM available - especially if you're using
catstep for it's main design purpose of viewing nccm / tee terminal captures
which tend to be quite small.

- If you set the sleep to larger values (slower replay) you will notice that
char display speed varies if the text changes color or if the cursor is
moved. This is because these control characters are also chars in the file and
the same sleep value applies to them even if they don't result in visible
output. Use the -t option to fix this which results in smoother output in some
cases and jumpy output in others (more noticable at slow replay).

- When you quit catstep or after it completes displaying a file naturally,
your terminal may be a bit messed up if the file you displayed changed any
colors or anything else. Use `reset` from the command line to fix this. While I
can call this from python, it's probably better for you to do it manually if
and when necessary.


Troubleshooting
---------------

Use the `-d` to add verbose logging to syslog.

At this stage I haven't added many error capturing mechanisms, so expect
a lot of python exceptions when unexpected commands are given.

