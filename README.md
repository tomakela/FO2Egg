# FO2Egg
Change the size of the translucent "egg" effect around player in Fallout 2

## Installation
- Get the latest sfall: https://github.com/sfall-team/sfall (tested on 4.3.7)
- Put EGG.frm, EGG2.frm, EGG3.frm, EGG4.frm, EGG5.frm, and INTRFACE.LST into data\art\intrface inside your Fallout 2 installation folder
- Put gl_eggchange.int into your data\scripts
- Enable unsafe sfall script calls in ddraw.ini by setting AllowUnsafeScripting=1

## Usage
Press F:

![img](https://github.com/tomakela/FO2Egg/assets/9822663/2d45885a-9279-4ff5-bde4-61af5ea96fae)

## To note
- Might not be compatible with some mods as the code is dependent on the INTRFACE.LST line numbers
- Could absolutely have some side effects, use with your own discretion

## What is happening?

Fallout2 graphics engine renders the foreground surroundings of the player character transparent. The transparency is complete in the center and gradually changes into opaque. This shape is called "egg" in the data files, and is loaded from data\art\intrface\EGG.FRM and listed third  (i.e. index two when counting from 0) in the INTRFACE.LST. The original EGG.FRM is single channel bitmap with 1 beign transparent, 0 being fully opaque. Transparency is reduced as the pixel value increases from 1 in the middle to 122 at the edges (and zero around it):
<img width="290" alt="image" src="https://github.com/tomakela/FO2Egg/assets/9822663/0f4d4480-d793-4d32-86c7-6fa6c1f1eabf">

In Alex Balatov's excellent reverse engineering of Fallout2 executable (https://github.com/alexbatalov/fallout2-re), one can find the memory address where the code responsible for loading EGG.FRM resides. This is called before the game enters the main menu.
