
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

I wrote this program as a solution to display terminal output that was
captured with tee.
My other software - nccm (ncurses ssh connection manager) has an option
to pipe all ssh command output to tee, which results in a file that contains
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

-a / --autoplay :
Auto replay the file one char at a time.
Respects the sleep variable if supplied, else uses the default of
0.01 seconds. Applies only to autoplay mode.

-d / --debug :
Log extra debugging information to syslog. Applies to both modes.

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
If you enter a value not 5-60 it will force 10 seconds.

-s / --sleep <seconds> :
Sleep <seconds> between chars output. Can be any number including
times less than 1 (eg: 0.5 , 0.001 , 2.3 , etc).
Applies to both modes.


Interactive mode controls
-------------------------

- 0-9 :               Play back 10^n chars where n is the key pressed.
                      0 plays back 1 char, 1 plays back 10 chars,
                      2 plays back 100 chars, etc
- q :                 Quit catstep
- p :                 Log filename and position in file


Limitations
-----------

There's no step back option to 'replay backwards' because catstep
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
With some files - especially those with lots of cursor controls, catstep may
display a summary char count lower than the actual file size. I have not seen
catstep miss any output display so it could be that certain cursor control
chars don't get counted. If anyone experiences problems or can narrow it down
to particular cases please report this in the issues.


Troubleshooting
---------------

Use the `-d` to add verbose logging to syslog.

At this stage I haven't added many error capturing mechanisms, so expect
a lot of python exceptions when unexpected commands are given.

