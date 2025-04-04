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

In Alex Batalov's excellent reverse engineering of Fallout2 executable (https://github.com/alexbatalov/fallout2-re), one can find the memory address where the code responsible for loading EGG.FRM resides. This is called before the game enters the main menu, which we can all agree is good and fine.

```
game/object.c:
// 0x488780
int obj_init(unsigned char* buf, int width, int height, int pitch)
```
But we are especially intereseted in the final part:
```
    eggFid = art_id(OBJ_TYPE_INTERFACE, 2, 0, 0, 0);
    obj_new(&obj_egg, eggFid, -1);
    obj_egg->flags |= OBJECT_FLAG_0x400;
    obj_egg->flags |= OBJECT_TEMPORARY;
    obj_egg->flags |= OBJECT_HIDDEN;
    obj_egg->flags |= OBJECT_LIGHT_THRU;

    objInitialized = true;

    return 0;
```
This matches the following in good old OllyDbg (OBJ_TYPE_INTERFACE = 6):

<img width="371" alt="image" src="https://github.com/tomakela/FO2Egg/assets/9822663/1cd3ae73-beb8-4d2c-81ed-29f9ff441785">

The number 2 in art_id is the index of EGG.FRM which can be changed by overwriting the memory address. We are also in luck: the obj_egg-pointer is a global variable, the function ends at this point, simple jump to the "eggFid = " line is enough to reload the EGG.FRM. Double-luckily sfall provides memory writes and calls when using unsafe scripting. Let's add EGG2-5.frms to the end of INTRFCE.LST, being the indices 469-472. A quick botch of a code that checks the memory address and toggles between the five 512x512 FRMs (i.e. 0002, 01D5-01D8) I created with different extent of transparency: 
```
if (keyDX == DIK_F) then begin // F is the key
  if (read_byte(0x488964) == 0x02) then begin
     write_byte(0x488964, 0xD5);
     write_byte(0x488965, 0x01);
     write_byte(0x4889a4, 0xC3);
  end else begin
     dummy = read_byte(0x488964);
     if (dummy == 0xD8) then begin
        write_byte(0x488964, 0x02);
        write_byte(0x488965, 0x00);
     end else begin
        write_byte(0x488964, dummy+1);
     end
  end
  dummy=call_offset_r0(0x488961);
```

This would work in principle, but we have two issues:

1) Graphics are not updated with this call. We need something to trigger a dummy graphics update. If I got this right, one way without too many side effects is to register a entry to animation queue, change the player graphics to the current graphics, and no delay (for the last one: I think -1 was the magic I was looking for all this time). I am sure there is a smarter way, but this seems to work:
```
if (call_offset_r1(0x413AF4,2) == 0) then begin // register_begin()
   dummy=call_offset_r3(0x41518C, dude_obj, obj_art_fid(dude_obj), -1); // register_object_change_fid. dummy "animation" with instant redraw
   dummy=call_offset_r0(0x413CCC); // register_end()
end
```

2) The original EGG.FRM needs to be replaced with 512x512-version because otherwise the graphics don't get updated when moving from the largest to the original size. The version I made _should_ match the original

There. The hardest parts were to find the correct memory addresses to call (not that hard), the creation of the larger "eggs" so that the center points are still sensible and look relatively good (okay, this is also not so hard), and understanding the FRM format (not hard thanks to https://fallout.fandom.com/wiki/FRM_file).
