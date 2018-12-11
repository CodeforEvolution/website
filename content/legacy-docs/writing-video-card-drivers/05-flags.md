+++
type = "article"
title = "Flags"
date = "2010-05-26T02:24:07.000Z"
tags = []
+++
### 5 - Flags

![Flags Chart](/files/images/writing-video-card-drivers/5.flags.png)

**Chart Legend**

*   **Name** - The name as defined in the BeOS header files.
*   **API Construct** - The construct used in classes and functions.
*   **C** - The command, from API of appserver or accelerant.
*   **S** - Status, from accelerant to API.
*   **P** - The app\_server is target.
*   **A** - The accelerant is source or target.

#### 5.1 - Flags For User Overlay

##### 5.1.1 - B\_BITMAP\_WILL\_OVERLAY

##### 5.1.2 - B\_BITMAP\_RESERVE\_OVERLAY\_CHANNEL

##### 5.1.3 - B\_OVERLAY\_TRANSFER\_CHANNEL

##### 5.1.4 - B\_OVERLAY\_MIRROR

##### 5.1.5 - B\_OVERLAY\_FILTER\_HORIZONTAL

##### 5.1.6 - B\_OVERLAY\_FILTER\_VERTICAL

#### 5.2 - Flags For Mode Setup: Mode Flags

##### 5.2.1 - B\_SUPPORTS\_OVERLAYS

##### 5.2.2 - B\_HARDWARE\_CURSOR

##### 5.2.3 - B\_IO\_FB\_NA

##### 5.2.4 - B\_PARALLEL\_ACCESS

##### 5.2.5 - B\_8\_BIT\_DAC

##### 5.2.6 - B\_DPMS

##### 5.2.7 - B\_SCROLL

#### 5.3 - Flags For Mode Setup: Mode Timing Flags

##### 5.3.1 - B\_BLANK\_PEDESTAL

##### 5.3.2 - B\_TIMING\_INTERLACED

##### 5.3.3 - B\_SYNC\_ON\_GREEN

##### 5.3.4 - B\_POSITIVE\_HSYNC & B\_POSITIVE\_VSYNC

#### 5.4 - Conclusion
