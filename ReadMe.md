### [tldr]

The midimacros.sh file is a bash script that (should) run on any Linux computer with X or Wayland running that has alsa installed, detect all your midi devices, and provide keystrokes and other actions that are defined (and permanently compiled into the script) in the midimacros-IDE.fods file. Running from within Wayland is not tested, but it is expected that it will have less success working across the entire computer than when run from X.  The .fods is "flat opendocument spreadsheet", and will open in LibreOffice, or REALLY old versions of OpenOffice (as in when Sun took the project over old.)  "Flat" means it's really just a big .xml file you can view with a text browser too. I use this for storing on github even though LibreOffice makes zillions of changes internally every time you press a key. 

The midimacros-IDE has code generating tabs for all MIDI keys on channel zero, and random events that align with controllers I own.  It also has these mapped on several tabs to use or print for reference, and a tab to copy new executable script if you want to customize the key layout. 

Most of this is the idea and programming from the superuser site described below.  Anything that I added, I release for all use without any additional restrictions.  For that original code, it seems simple and short enough that no license need be sought, but fair warning... I did indeed copy from superuser. 

### Running midimacros

After making sure it's executable,

> ./midimacros.sh 

Allowed arguments:

> ./midimacros.sh -l 

list all detected devices

> ./midimacros.sh -t (yourMidiDevice) 

test that alsa is seeing keystrokes. 

> ./midimacros.sh

By itself midimacros will try to detect all connected devices and translate every press on every device that matches the definitions file. 

> ./midimacros.sh (yourMidiDevice)

If you need to limit midimacros to only some of the connected devices, you can list them using the same means as aseqdump allows client name or port numbers comma separated with no space between clients or ports. As in midimacros Xonar,APC  ; or midimacros 24:0,24:1 ; and it's either numbers or names, no mixing.



## Disclaimers.

> Read through this entire document before attempting to use AV-MidiMacros.

> Proceed at your own risk! Running any program carries risk. This file and the resulting script may or may not act appropriately.  No warranties of any kind are provided. Following these instructions may provide usable code, but the responsibility for that code rests on the one following this guide. 

## Background.
### _(or, Why am I here?)_

I typeset... or rather _design_ printed books. In the past (and hopefully in the future) this includes books in non-latin scripts. This creates the need to be able to produce unicode glyphs that aren't on the standard US keyboard.  While I can produce these by character mapping tools, this is slow, which means touching up design flaws like an errant minus that should be a figure dash takes an outsize portion of my time. In addition, finishing a design involves fixing paragraphs so they break in certain ways (no single words on the last line and no single lines at column breaks, and hyphenation only occurs in certain ways.)  A large book has hundreds of pages, and thousands of paragraphs, so this becomes very repetitive. Calling macros to change the shape of a paragraph from a keyboard with live typing keys, where the immediate response is most of the words in the paragraph shift... Mistyping a live glyph instead of a function key can go unnoticed, introducing error instead of improvement. 

So, because of my background: The goal of midimacros is to provide

1. __A programmable button board__  that provides nothing but function keys (midimacros) for long sessions finishing books; and alternatively
2. A user-programmable __2nd typing keyboard__ for producing unusual glyphs, both 
    a. glyphs used in book publishing, and
    b. alternate scripts like Hindi, Hebrew, or Korean.

After playing with the ___xdotool___ command that popped up as a solution on a web discussion about this, I added a 3rd goal: 

3. provide advance keypress ---> multiple click options for __2 handed mouse control__. 

Accomplishing Goal 3 introduced a squeezed keyboard that I wanted more feature than will fit, so I then added a 4th  and 5th goal:

4. Keys and pads produce intuitive multiple responses based on __touch sensitivity__. 
5. __Key chords__ (pressing multiple keys at once) produce alternate responses. 

Since the inexpensive midi controller purchased for this endeavour includes additional controls (pads, wheels, fader, and dials,) I have played with including these into the code:

6. Intuitive features for __control pads, faders, dials, and wheels__.

After 5ish programming sessions spread across years, I'm approaching successful implementation on Goals 1-4, and goal 6 has some weak setaup, but I have not done any real work toward goal 5. The code base I started from is by design 1 input, and not well formed for key chords.  I'm currently (infrequently) reading about shorthand scripts and stenotype like court recorders use. But this goal 5 at least for now seems like it needs to be separated from the other 4, and probably done by someone who can actually play a piano with multiple fingers.

### Translating MIDI into computer keystrokes on Linux. 

> Re: (https://superuser.com/questions/1170136/translating-midi-input-into-computer-keystrokes-on-linux)
>
> _I have a MIDI controller (Launchkey Mini) that I don't really use for music production anymore, but I would like to use the drum pad buttons or piano keys on it to perform computer keyboard combinations, such as Ctrl + Alt + Delete, with a single press._
>
> _Whenever I try to research this question, I either get pointed to Bome's MIDI Translator, which costs money and isn't on Linux, a horribly outdated plug-in/application, or some random library for coding it myself, which I have no clue how to do since I have virtually no skill in programming things related to audio._
>
> _Is there some way I can do this? I am on Xubuntu 16.04._

____

> This cannot be done without some programming.

> First, test how to detect MIDI events. Go to a terminal, and run aseqdump -l to list the MIDI ports; this outputs something like this:

    $ aseqdump -l
    Port    Client name                      Port name
    0:0     System                           Timer
    0:1     System                           Announce
    14:0    Midi Through                     Midi Through Port-0
    24:0    Xonar D2                         Xonar D2 MIDI
    32:0    Yamaha DS-1E (YMF754)            Yamaha DS-1E (YMF754) MIDI

> Then run it with the client name to check whether events arrive:

    $ aseqdump -p "Xonar D2"
    Waiting for data. Press Ctrl+C to end.
    Source  Event                  Ch  Data
    24:0   Note on                 0, note 64, velocity 86
    24:0   Note on                 0, note 48, velocity 80
    24:0   Note off                0, note 48
    24:0   Note on                 0, note 68, velocity 84
    24:0   Note on                 0, note 52, velocity 88
    24:0   Note off                0, note 64
    24:0   Note off                0, note 52
    24:0   Note off                0, note 68
    ...
    

> Second, to simulate key strokes, you need xdotool. If you do not yet have it installed, run sudo apt-get install xdotool. You can use type to type text, or key to simulate special keys:

    
    xdotool type Hello, World!
    xdotool key ctrl+p
    xdotool mouse click
    xdotool exec espeak 'hello!'
    

> Please note that not all special keys are handled correctly by xdotool. For example, Ctrl+Alt+Del is handled very specially by the kernel and probably does not work when simulated; try running sudo reset instead of xdotool.

> Finally, tie everything together with a script. Put this into a text file, for example,

~/bin/midi-to-keys:

    #!/bin/bash
    aseqdump -p "Xonar D2" | \
    while IFS=" ," read src ev1 ev2 ch label1 data1 label2 data2 rest; do
    case "$ev1 $ev2 $data1" in
    "Note on 64" ) xdotool type hello ;;
    "Note on 48" ) xdotool key ctrl+j ;;
    esac
    done

> Make it executable, 

    $ sudo chmod +x ~/bin/MidiMacros.sh
    
> and run it

    $ ./midimacros.sh

> Now, pressing E-5 or C-4 should have some effect.

> Change or add lines of the form "Note on x" ) command ;; to do whatever you want. _(answered Jan 22 '17 at 9:30 by CL)_



# AV-MidiMacros Information

## Setting up the MIDI keyboard: 

1. Run aseqdump -l and note your keyboard's name. type this into the NAME field (cell C1) on the keyboard tab.
2. This script is defined for 3 levels of sensitivity on the keys and 2 levels on the pads. Due to the case command wanting to deal with string matching and not math logic, the patterns are pretty complex.  See the note below how to modify the sensitivity if needed for your keyboard. 
3. The pads are still a work in progress. My pads don't work well so I've only programmed 2 levels of sensitivity. 
4. The control knobs are listed, but have no actions or output defined. There usefulness is limited without a lot more programming, which makes 'in a user script' not very feasible because it will break too easily.
5. The included settings are for a keyboard with has 32 musical keys and octave shifting.  This provides 3 complete sets of usable keys, and a few others on partially duplicated layouts. Note the Oct(ave) column in the keyboard tab and the colored keynames are intended to help aid programming and referring to the tables. These are intended to be redefined for each keyboard. 

## Defining new actions: 

1. If you have additional Actions, ensure they are included on the actions tab. __MM Name__ is the search text you will include on the "keyboard" tab to setup the key action. __Abbr__ is the text that will appear on the Overlay tab (acts as both a validation, and a cheatsheet for using MidiMacros.) __Bash Command__ is any command you want to run from bash.  Note that my setup is all about pushig keystrokes and macros into text programs like jedit and LibreOffice Writer, but any bash command is valid, noting that I haven't created new processes for each macro press, so If your bash command needs a new shell, to keep both midiMacros and your function working, you need to spawn the new shell in the bash text you enter in this column. __Type__ is a column I use to sort fields if I can't find a key I know I've programmed (triple click? where'd it go?)

2. After adding any Actions to the Actions tab, the table must be sorted alphabetically by the 'MM-Name' field. MidiMacros won't work until you sort the Actions field by the __MM-Name__ column.
3. Copy the Action Description field from the Actions Tab, and paste into the appropriate Actions cell in the Keyboard, Pads, or Controls Tab.
4. Save your configuration file in Calc format. (.ods or .fods)

## Saving your new MidiMacro script: 

1. Switch to the Output tab
2. While the output tab is visible, select Save as 
    a. select the Text (.csv)
    b. ensure edit filter settings is on.
    c. These settings on the Filter page: 
         i. Character set: UTF-8
         ii. Field delimiter: {space}
         iii. String delimiter: [empty] (you must delete the quote that appears.)
         iv. Save cell content as shown is the only box selected.
3. After the file is saved to .csv either close the window or resave it as calc format, or further edits will be lost.
4. Rename the .csv file to .sh file, and change the execute bit (chown).

## Running the Script.

1. The script is run from a bash shell (a standard linux terminal.)

     $ ./MidiMacro.sh 

2. It might be helpful to debug by running aseqdump -p “yourkeyboard” in a 2nd terminal. multiple instances of aseqdump seem to run fine. 
3. The script can be stopped by pressing ctrl-c or closing the terminal window.

## Suggestions, Bugs, Feedback

Please use the issues tab here in this github project to make suggestions, provide feedback and report bugs.  I'm especially interested in alternate script layouts.  I've no clue, but it is my hope that eventually, I'm not the only one using a 32 key midi board as a 2nd multi layout typing board, so keymaps for Korean, Japanese, etc. are accepted. 
   
____

# Appendices. 

## Programming techniques

### Using numeric ranges in case statements.

> re: (https://stackoverflow.com/questions/30000649/using-numeric-ranges-in-a-case-statement)

> I'm facing trouble in getting the cases executed. Every time, it goes to the last case i.e. * characters.

> Here's what I'm using:

    case $used_space in
    [1-84])
        echo "OK - $used_space% of disk space used."
        exit 0
        ;;
    [85])
        echo "WARNING - $used_space% of disk space used."
        exit 1
        ;;
    [86-100]*)
        echo "CRITICAL - $used_space% of disk space used."
        exit 2
        ;;
    *)
        echo "$used_space% of disk space used."
        exit 3
        ;;
    esac

> How can I change my case statement to work with numeric ranges?

____

> With bash and case:

    case $used_space in
    [1-9]|[1-7][0-9]|8[0-4]) # range 1-84
    echo "OK - $used_space% of disk space used."
    exit 0
    ;;
     85)
    echo "WARNING - $used_space% of disk space used."
    exit 1
    ;;
    8[6-9]|9[0-9]|100)        # range 86-100
    echo "CRITICAL - $used_space% of disk space used."
    exit 2
    ;;
    *)
    echo "$used_space% of disk space used."
    exit 3
     ;;
    esac
### Regex Information:

> Regex ranges  look like [ ] (https://javascript.info/regexp-character-sets-and-ranges) .

> Regex alternates look like |  (https://www.regular-expressions.info/alternation.html) .

