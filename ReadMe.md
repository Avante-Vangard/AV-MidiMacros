## Disclaimers. 

Read through this entire document before attempting to use AV-MidiMacros. 

Proceed at your own risk. Running any program carries risk. This file and the resulting script may or may  not act appropriately.  No warranties of any kind are provided. following these instructions may provide  usable code, but the resposibility for that code rests on the one following this guide.

## Background (or why am I here?) 

### Translating MIDI into computer keystrokes on Linux. 
Re: [Translating MIDI input into computer keystrokes](https://superuser.com/questions/1170136/translating-midi-input-into-computer-keystrokes-on-linux) 

I have a MIDI controller (Launchkey Mini) that I don't really use for music production anymore, but I would like to use the drum pad buttons or piano keys on it to perform computer keyboard combinations, such as Ctrl + Alt + Delete, with a single press.

Whenever I try to research this question, I either get pointed to Bome's MIDI Translator, which costs money and isn't on Linux, a horribly outdated plug-in/application, or some random library for coding it myself, which I have no clue how to do since I have virtually no skill in programming things related to audio.

Is there some way I can do this? I am on Xubuntu 16.04.


This cannot be done without some programming.

First, test how to detect MIDI events. Go to a terminal, and run aseqdump -l to list the MIDI ports; this outputs something like this:

`$ aseqdump -l
 Port    Client name                      Port name
  0:0    System                           Timer
  0:1    System                           Announce
 14:0    Midi Through                     Midi Through Port-0
 24:0    Xonar D2                         Xonar D2 MIDI
 32:0    Yamaha DS-1E (YMF754)            Yamaha DS-1E (YMF754) MIDI`

Then run it with the client name to check whether events arrive:

{
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
}

Second, to simulate key strokes, you need xdotool. If you do not yet have it installed, run sudo apt-get install xdotool. You can use type to type text, or key to simulate special keys:

{
xdotool type Hello, World!
xdotool key ctrl+p
}

Please note that not all special keys are handled correctly by xdotool. And Ctrl+Alt+Del is handled very specially by the kernel and probably does not work when simulated; try running sudo reset instead of xdotool.

Finally, tie everything together with a script. Put this into a text file, for example, ~/bin/midi-to-keys:

#!/bin/bash
aseqdump -p "Xonar D2" | \
while IFS=" ," read src ev1 ev2 ch label1 data1 label2 data2 rest; do
    case "$ev1 $ev2 $data1" in
Note on 64" ) xdotool type hello ;;
        "Note on 48" ) xdotool key ctrl+j ;;
    esac
done

Make it executable (chmod +x ~/bin/midi-to-keys), and run it (~/bin/midi-to-keys). Now, pressing E-5 or C-4 should have some effect.

Change or add lines of the form "Note on x" ) command ;; to do whatever you want.

answered Jan 22 '17 at 9:30 by CL



# AV-MidiMacros Information

## Setting up the MIDI keyboard: 

   1. Run aseqdump -l and note your keyboard's name. type this into the NAME field (cell C1) on the keyboard tab.
   2. This script is defined for 3 levels of sensitivity on the keys and 2 levels on the pads. Due to the case command wanting to deal with string matching and not math logic, the patterns are pretty complex.  See the note below how to modify the sensitivity if needed for your keyboard. 
   3. The pads are still a work in progress. My pads don't work well so I've only programmed 2 levels of sensitivity. 
   4. The control knobs are listed, but have no actions or output defined. There usefulness is limited without a lot more programming, which makes 'in a user script' not very feasible because it will break too easily.
   5. The included settings are for a keyboard with has 32 musical keys and octave shifting.  This provides 3 complete sets of usable keys, and a few others on partially duplicated layouts. Note the Oct(ave) column in the keyboard tab and the colored keynames are intended to help aid programming and referring to the tables. These are intended to be redefined for each keyboard. 

## Defining new actions: 

   1. If you have additional Actions, ensure they are included on the actions tab.
   2. After adding any Actions to the Actions tab, the table must be sorted alphabetically by the description field. 
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
   4. Rename the .csv file to .bsh file, and change the execute bit (chown). 

## Running the Script: 

   1. The script is run from a bash shell (a standard linux terminal.) $ ./MidiMacro.bsh 
   2. It might be helpful to debug by running aseqdump -p “yourkeyboard” in a 2nd terminal. multiple instances of aseqdump seems to run fine. 
   3. The script can be stopped by pressing ctrl-c or closing the terminal window. 
   


## Using numeric ranges in case statements.
re: [Numeric Ranged in case statements](https://stackoverflow.com/questions/30000649/using-numeric-ranges-in-a-case-statement)

I'm facing trouble in getting the cases executed. Every time, it goes to the last case i.e. * characters.

Here's what I'm using:

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

How can I change my case statement to work with numeric ranges?


With bash and case:

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

More information: 
[Regex ranges](https://javascript.info/regexp-character-sets-and-ranges) look like [ ]
[Regex alternates](https://www.regular-expressions.info/alternation.html) look like |
