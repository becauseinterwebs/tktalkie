# TK Talkie
Code for Teensy 3.2 (or arduino compatible) and an audio shield to automatically add comm static when talking...or more precisely...when someone STOPS talking.

This code is based on examples that come with the Teensy board from www.pjrc.com and Arduino (www.arduino.cc) and have been modified for a particular purpose and is distrubuted under the GNU General License.

In particular, the equipment that this was written for (but should work with other boards with minimal modification):

-  Teensy 3.2 available at http://www.pjrc.com/store/teensy32.html.  For a few more dollars, you can get one that already has the pins soldered on it (a HUGE timesaver!) http://www.pjrc.com/store/teensy32_pins.html
-  Teensy Audio Board available at http://www.pjrc.com/store/teensy3_audio.html
-  (Optional...but makes life easier!) 14 pin header (http://www.pjrc.com/store/header_14x1.html) and socket (http://www.pjrc.com/store/socket_14x1.html) for mounting the audio shield to the Teensy (instructions on Teensy website)
-  (Optional) Micro volume pot (http://www.pjrc.com/store/pot_thumb_25k.html)
-  Micro SD Card (at least 8 Gigs...I think that's the smallest you can get right now anyway ;)  )

There is a great online GUI that allows you to create audio component configurations as well as allows you to setup connections and find out more about how they work.  This is available at:

-  http://www.pjrc.com/teensy/gui

You should be able to take the configuration generated by this GUI and import it back in if you wish to modify it.

## Requirements

You will need:

-  The boards listed above (or compatibles)
-  Arduino software (available at www.arduino.cc)
-  Teensyduino loader software (avaliable at http://www.pjrc.com/teensy/td_download.html)

There are a wealth of examples on the Teensy website as well in the Arduino software once you have installed Teensyduino.

## Getting Started

Once you have installed the Arduino and Teensyduino software packages, you should familiarize yourself with the tutorials. 

- Download the example .WAV files (SDCard.zip) and unzip them to your micro SD card.  The file named "STARTUP.WAV" is a special file that is played when the board is turned on.
- You can then load the TK Talkie sketch and compile and upload it to your Teensy.

If everything is working correctly, you should hear the STARTUP.WAV file play when your board resets.

## WAV Files

You can [download a zip file with WAV files here.] (http://www.becauseinterwebs.com/downloads/TKTalkieSounds.zip)

Note that the SD Card reader is expecting filenames in all uppercase, 8.3 format (file name can be up to 8 characters, extension is 3 characters.)  The way the sketch is currently written, it will load all WAV files it finds that start with "TKT_" into an array and randomly select one to play whenever you stop talking.  You can, of course, change this.

If you want to make your own wav files, make sure they are at least 16-bit and sampled at 44100Hz.  It doesn't matter if they are mono or stereo.

## Customizing the Software

The sketch is pretty well noted throughout, so it should be pretty easy to understand what is going on at any place in the application.  Again, for specifics on how any of the audio components work, please reference the Teensy example and information.

## Development Notes/How It Works

The loop() function is the main function of any 'duino sketch.  In this loop, the input (microphone or line-in) is checked to see if there is an audio signal.  If there is, it will wait until the signal goes away (i.e. stopped talking) and then randomly select a sound to play from the card.

You will see some defined constants:

    const int MIN_SILENCE_TIME = 350;

This is the amount of time to wait (in milliseconds) after no more input is detected before playing a sound.  You don't want to make this wait too long or there will be a long silence after you stop talking.  Also, you don't want to make it too short so that if you pause briefly while speaking it doesn't trigger the sound.  Around 250 to 350 milliseconds should work just fine.

    const float VOL_THRESHOLD_TRIGGER  = 0.15;
    const float VOL_THRESHOLD_MIN      = 0.01;

These are the input (amplitude) levels that trigger the sound.  The VOL_THRESHOLD_TRIGGER is the level that needs to be hit in order to let the software know you've started talking.  The VOL_THRESHOLD_MIN is the level that the input needs to fall below after you've started talking to know it's time to wait and see if you are done and then play a random sound.  These two values let you tweak the triggers that work best for your voice.  This allows you to start talking and get a little quieter without triggering the sound effects accidentally.  The trigger number helps to prevent false positives that can be set off just by breathing...it's not perfect, but the default settings should work pretty well.  If not, just tweak them until you find a combination that works for you.

    String wavFiles[99];

This array will hold a list of valid WAV files found on the SD card.  The number 99 is abitrary.  Ideally you don't want to set it to more than the number of files you will have on your card, but setting it to a higher number will let you add more files to the SD card later without having to recompile the sketch.

## A Note About WAV Files and Sound Levels

Since the recorded levels of the WAV files you use can vary greatly, if you are making your own try to keep the levels as close as possible (to avoid things like really loud effects vs. quiet effects.)  You can adjust the microphone gain, the SD Card Player output levels, etc.  There are several places in the setup() function where you can make these adjustments.  They are noted with comments such as "adjust as needed."

## Conclusion

I built this project because I wanted to have full control over the sound effects that were played, plus it helps that it only costs around $40.00 if you buy all the parts listed above (except for the SD card as the price on those varies greatly...)  

It's a fun project and adds that little something extra to your TK build.  Have fun!

## Troubleshooting

Here are some of the more common problems you may experience.

### WAV Files Are Not Playing

Make sure the WAV file is in 16-bit (mono or stereo) at 44100Hz.  This seems to be the ideal setting.

### Sound Effects Play Before I Stop Speaking / Sound Effects Do Not Play

Most likely this is because the **VOL_THRESHOLD_MIN** is set too low, or your microphone puts off a constant signal **above** what you have the **VOL_THRESHOLD_MIN** set to.  To check this, make sure the line:

    Serial.begin(9600);
    
is uncommented, then add the following to the **loop()** function right after the line where the microphone input is checked.  It should look like this:

    float val = rms1.read();
    Serial.println(val);
    
If your microphone is putting off a constant signal (over 0.00) then make sure your **VOL_THRESHOLD_MIN** is set above this signal.  For example, if it puts off a constant signal of 0.02, set the threshold level to 0.03 or 0.04.

### My Microphone Is Not Working

First, test and make sure the microphone works on something else.  If it does, check if it has a stereo jack (two stripes on the jack) or if it is mono (just one stripe.)  If it's mono and you used a stereo input jack on the audio shield, it's probably not connecting with one of the channels.  Try pulling it out just a tad and see if you get signal. If you do, you'll need to either switch out the input jack to a mono jack, or disconnect one of the input leads on the input jack and just use one (should be left channel...but maybe the right.)
