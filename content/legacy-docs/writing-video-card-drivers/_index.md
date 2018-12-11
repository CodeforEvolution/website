+++
type = "article"
title = "Writing Video Card Drivers In BeOS"
date = "2010-05-26T02:02:46.000Z"
tags = []
+++

A Thesis By:
------------

**Rudolf Cornelissen**  
part-time student  
06 June 2003  

Table Of Contents
-----------------

1.  [Introduction](/legacy-docs/writing-video-card-drivers/01-introduction)
    1.  [Problem Description](/legacy-docs/writing-video-card-drivers/01-introduction#1.1)
    2.  [Thesis](/legacy-docs/writing-video-card-drivers/01-introduction#1.2)
    3.  [About The Author](/legacy-docs/writing-video-card-drivers/01-introduction#1.3)
    4.  [About BeOS](/legacy-docs/writing-video-card-drivers/01-introduction#1.4)
    5.  [About Video Card Drivers](/legacy-docs/writing-video-card-drivers/01-introduction#1.5)
  
2.  [BeOS API Classes For Video Card Drivers](/legacy-docs/writing-video-card-drivers/02-beos-api)
    1.  [BScreen (Interface Kit)](/legacy-docs/writing-video-card-drivers/02-beos-api#2.1)
    2.  [BWindowsScreen (Game Kit)](/legacy-docs/writing-video-card-drivers/02-beos-api#2.2)
    3.  [BDirectWindows (Game Kit)](/legacy-docs/writing-video-card-drivers/02-beos-api#2.3)
    4.  [Classes For Hardware Overlay: BBitmap (Interface Kit)](/legacy-docs/writing-video-card-drivers/02-beos-api#2.4)
    5.  [Classes For Hardware Overlay: BView (Interface Kit)](/legacy-docs/writing-video-card-drivers/02-beos-api#2.5)
    6.  [Conclusion](/legacy-docs/writing-video-card-drivers/02-beos-api#2.6)
  
3.  [Kernel Driver](/legacy-docs/writing-video-card-drivers/03-kernel-driver)
    1.  [Interface To The OS](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.1)
        1.  [init\_hardware()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.1.1)
        2.  [init\_driver()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.1.2)
        3.  [publish\_devices()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.1.3)
        4.  [uninit\_driver()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.1.4)
        5.  [find\_device()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.1.5)
      
    2.  [Interface To The User](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.2)
        1.  [open\_hook()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.2.1)
        2.  [close\_hook()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.2.2)
        3.  [free\_hook()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.2.3)
        4.  [control\_hook()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.2.4)
        5.  [read\_hook()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.2.5)
        6.  [write\_hook()](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.2.6)
      
    3.  [Conclusion](/legacy-docs/writing-video-card-drivers/03-kernel-driver#3.3)
  
4.  [Accelerant](/legacy-docs/writing-video-card-drivers/04-accelerant)
    1.  [The Hooks Of The Accelerant](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1)
        1.  [INIT\_ACCELERANT](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.1)
        2.  [CLONE\_ACCELERANT](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.2)
        3.  [UNINIT\_ACCELERANT](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.3)
        4.  [ACCELERANT\_RETRACE\_SEMAPHORE](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.4)
        5.  [ACCELERANT\_MODE\_COUNT & GET\_MODE\_LIST](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.5)
        6.  [PROPOSE\_DISPLAY\_MODE](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.6)
        7.  [SET\_DISPLAY\_MODE](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.7)
        8.  [GET\_FRAME\_BUFFER\_CONFIG](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.8)
        9.  [GET\_PIXEL\_CLOCK\_LIMITS](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.9)
        10.  [MOVE\_DISPLAY](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.10)
        11.  [SET\_INDEXED\_COLOR](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.11)
        12.  [GET\_TIMING\_CONSTRAINTS](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.12)
        13.  [SET\_CURSOR\_SHAPE](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.13)
        14.  [MOVE\_CURSOR](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.14)
        15.  [2D Accelerant Functions](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.15)
        16.  [Hardware Overlay Functions](/legacy-docs/writing-video-card-drivers/04-accelerant#4.1.16)
      
    2.  [Conclusion](/legacy-docs/writing-video-card-drivers/04-accelerant#4.2)
  
5.  [Flags](/legacy-docs/writing-video-card-drivers/05-flags)
    1.  [Flags For User Overlay](/legacy-docs/writing-video-card-drivers/05-flags#5.1)
        1.  [B\_BITMAP\_WILL\_OVERLAY](/legacy-docs/writing-video-card-drivers/05-flags#5.1.1)
        2.  [B\_BITMAP\_RESERVE\_OVERLAY\_CHANNEL](/legacy-docs/writing-video-card-drivers/05-flags#5.1.2)
        3.  [B\_OVERLAY\_TRANSFER\_CHANNEL](/legacy-docs/writing-video-card-drivers/05-flags#5.1.3)
        4.  [B\_OVERLAY\_MIRROR](/legacy-docs/writing-video-card-drivers/05-flags#5.1.4)
        5.  [B\_OVERLAY\_FILTER\_HORIZONTAL](/legacy-docs/writing-video-card-drivers/05-flags#5.1.5)
        6.  [B\_OVERLAY\_FILTER\_VERTICAL](/legacy-docs/writing-video-card-drivers/05-flags#5.1.6)
      
    2.  [Flags For Mode Setup: Mode Flags](/legacy-docs/writing-video-card-drivers/05-flags#5.2)
        1.  [B\_SUPPORTS\_OVERLAYS](/legacy-docs/writing-video-card-drivers/05-flags#5.2.1)
        2.  [B\_HARDWARE\_CURSOR](/legacy-docs/writing-video-card-drivers/05-flags#5.2.2)
        3.  [B\_IO\_FB\_NA](/legacy-docs/writing-video-card-drivers/05-flags#5.2.3)
        4.  [B\_PARALLEL\_ACCESS](/legacy-docs/writing-video-card-drivers/05-flags#5.2.4)
        5.  [B\_8\_BIT\_DAC](/legacy-docs/writing-video-card-drivers/05-flags#5.2.5)
        6.  [B\_DPMS](/legacy-docs/writing-video-card-drivers/05-flags#5.2.6)
        7.  [B\_SCROLL](/legacy-docs/writing-video-card-drivers/05-flags#5.2.7)
      
    3.  [Flags For Mode Setup: Mode Timing Flags](/legacy-docs/writing-video-card-drivers/05-flags#5.3)
        1.  [B\_BLANK\_PEDESTAL](/legacy-docs/writing-video-card-drivers/05-flags#5.3.1)
        2.  [B\_TIMING\_INTERLACED](/legacy-docs/writing-video-card-drivers/05-flags#5.3.2)
        3.  [B\_SYNC\_ON\_GREEN](/legacy-docs/writing-video-card-drivers/05-flags#5.3.3)
        4.  [B\_POSITIVE\_HSYNC & B\_POSITIVE\_VSYNC](/legacy-docs/writing-video-card-drivers/05-flags#5.3.4)
      
    4.  [Conclusion](/legacy-docs/writing-video-card-drivers/05-flags#5.4)
  
6.  [Writing The Driver](/legacy-docs/writing-video-card-drivers/06-writing-the-driver)
    1.  [Action Plan](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1)
        1.  [Preparations](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.1)
        2.  [Step 1: VBE2 (Vesa Mode) Activation](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.2)
        3.  [Step 2: Non-Active Driver Installation](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.3)
        4.  [Step 3: Hardware Cursor Building](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.4)
        5.  [Step 4: Setting The Frame Buffer Start Address](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.5)
        6.  [Step 5: Setting The Frame Buffer Pitch](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.6)
        7.  [Step 6: Setting The Color Depth](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.7)
        8.  [Step 7: Setting The Color Pallete](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.8)
        9.  [Step 8: DPMS Building](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.9)
        10.  [Step 9: Setting The Refresh Rate](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.10)
        11.  [Step 10: Setting The Monitor Timing](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.11)
        12.  [Step 11: Switching On 'Enhanced Mode'](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.12)
        13.  [Step 12: Setting The Acceleration](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.13)
        14.  [Step 13: Building Hardware Overlay](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.14)
        15.  [Step 14: Cold Start Of The Video Card](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.1.15)
      
    2.  [Testing The Driver](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.2)
        1.  [Kernel Driver](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.2.1)
        2.  [Accelerant](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.2.2)
      
    3.  [Stability](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.3)
    4.  [Conclusion](/legacy-docs/writing-video-card-drivers/06-writing-the-driver#6.4)
  
7.  [Conclusion](/legacy-docs/writing-video-card-drivers/07-conclusion)

Appendix
--------

1.  [Sources Of Information](/legacy-docs/writing-video-card-drivers/appendix-a-sources-of-information)
    1.  [The Manufacturer](/legacy-docs/writing-video-card-drivers/appendix-a-sources-of-information#a.1)
    2.  [Linux](/legacy-docs/writing-video-card-drivers/appendix-a-sources-of-information#a.2)
    3.  [The Internet](/legacy-docs/writing-video-card-drivers/appendix-a-sources-of-information#a.3)
    4.  [Testing For Specifications](/legacy-docs/writing-video-card-drivers/appendix-a-sources-of-information#a.4)
    5.  [Reverse Engineering](/legacy-docs/writing-video-card-drivers/appendix-a-sources-of-information#a.5)
2.  [Glossary](/legacy-docs/writing-video-card-drivers/appendix-b-glossary)
3.  [References](/legacy-docs/writing-video-card-drivers/appendix-c-references)
