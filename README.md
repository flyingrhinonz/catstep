
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

I wrote this program ( catstep: https://github.com/flyingrhinonz/catstep ) as
a solution to display terminal output that was captured with `tee`.
My other software - `nccm` ( ncurses ssh connection manager:
https://github.com/flyingrhinonz/nccm ) has an option
to pipe all ssh command output to `tee`, which results in a file that contains
a mixture of text and cursor/screen controls that's very hard to read in
a regular viewer. It does however display properly when you `cat` it to
screen but that flies away faster than you can read it.
catstep aims to fix this by allowing you to auto replay the file at a
controlled pace, or step through it manually.
Output is sent to your screen, which requires the display size to be
identical or larger than the terminal size at capture time otherwise
output will probably be wrong.


Command line arguments
----------------------

At it's simplest `catstep <file>` will get you going and put you into
interactive mode. Once in interactive mode use the keyboard controls
listed in the following section.

<file> :
The file you want to display.

-a / --autoplay :
Auto replay the file one char at a time.
Respects the sleep variable if supplied, else uses the default of
0.01 seconds. Applies only to autoplay mode.

-d / --debug :
Log extra debugging information to syslog. Applies to both modes.

-e / --nomemtest :
catstep won't open a file if it is larger than half the available RAM.
Bypass the memory test and open the file anyway.

-f / --fastforward <chars> :
Replay n chars at maximum speed before returning control to the regular
view. This lets you speed through the file until the actual point you
want to view at a controlled speed. Applies to both modes.

-h / --help :
Displays the command line options.

-i / --info :
Displays information about the file before and after it is displayed.
This includes file name, number of lines and chars.
Applies to both modes.

-m / --man :
Displays this man page.

-p / --periodic status <seconds> :
Logs the current position in the file every n seconds (default 10).
In interactive mode this only works if you're actively progressing in the file.
In autoplay mode this works every n seconds.
This can be really helpful if you speed past something and want to know where
you were a few seconds ago.
Supply a value of 0 to explicitly disable periodic logging.
If you supply a value not 0, 5-60 it will force 10 seconds.

-s / --sleep <seconds> :
Sleep <seconds> between chars output. Can be any number including
times less than 1 (eg: 0.5 , 0.001 , 2.3 , etc).
Applies to both modes.

-t / --smartsleep :
Sleep s seconds only for printable chars. Off by default. This yields a more
fluid playback when catstep encounters non-printable chars that would otherwise
also incur sleep. Applies to both modes.

Interactive mode controls
-------------------------

- 0-9 :             Play back 10^n chars where n is the key pressed.
                    0 plays back 1 char, 1 plays back 10 chars,
                    2 plays back 100 chars, etc
- q :               Quit catstep
- l :               Log filename and position in file
- f :               Make output faster by halving the sleep value.
- s :               Make output slower by doubling the sleep value.


Limitations
-----------

- There's no step back option to 'replay backwards' because catstep
doesn't retain state - your display is updated as per the controls in
the file so the current state is what you see on your terminal. At first
glance this may seem like a killer limitation, but for that I added
the `p` interactive control which logs the char currently displayed.
If you need to check out what happened a short while ago (say the screen
was cleared), just hit `p`, view the log file to see the line you were at,
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

- In interactive mode any keypress advances the output by 1 char even if the
key is not in use or has a completely different use.
All keys except 1-9 advance output by 1 char which is a consequence of
the way the loop is written - the keyboard read is part of the main loop which
speeds up catstep by avoiding calls to a separate function. If this behavior
becomes an annoyance or if enough people complain, I will replace this with
a generator.


Troubleshooting
---------------

Use the `-d` to add verbose logging to syslog.

At this stage I haven't added many error capturing mechanisms, so expect
a lot of python exceptions when unexpected commands are given.

