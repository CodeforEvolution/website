+++
type = "article"
title = "Accelerant"
date = "2010-05-26T02:22:03.000Z"
tags = []
+++

### 4 - Accelerant

The accelerant portion of the video card driver resides in user space. The accelerant makes available the necessary functions needed by the oprating system (app\_server), and any applications, to control the video card. These functions could be merely informative in nature, or could be used to set refresh rate, display size, and so forth.

The separation of the video card driver into two components, a kernel driver component, and a user space accelerant, is done for a few different reasons:

*   _Speed_. With system memory holding mappings to the Input/Output of the video card, we can use pointers to handle any I/O operations, without having to go through kernel space to do so each time. Context-switching (twice per I/O) back and forth between kernel space and user space costs a lot of time, especially if tens or even hundreds of registers need to be accessed.  
    We can speed up operations by providing lists of registers from system memory, and therefore limit the the number of context switches required.
*   _Stability_. A user space program can not crash the system, but a crash in kernel space will spell trouble. If the accelerant component crashes, the app\_server will no longer be responsive and will usually die, but system services such as telnet and ftp (run as daemons, really.) will still accept incoming connections, and allow you to "log in" and still interact with the system. There would be at this point still be a modicum of control allowing the user to tell the system to reboot gracefully, transfer files, etc.
*   _Debugging_. By separating the code into two parts, we can more easily add features to the driver over time, and see what impact our changes have on the system more easily. If something goes wrong, it's far easier to narrow down the offending piece of code, then wading through one gigantic set of code. Plus, it further refines the coding style, as it were, in case other programmers take a peek at how things were done.

![Theoretical possible BeOS setup with multiple graphics cards.](/files/images/writing-video-card-drivers/4.accelerant.png)

#### 4.1 - The Hooks Of The Accelerant

Hooks which are not implemented are not exported. The accelerant will return a nullpointer instead. Beware: except where pointed out, the hooks won't be requested again after a modeswitch!

When either of the two hooks `B_INIT_ACCELERANT` and `B_CLONE_ACCELERANT` are requested by the app\_server, they are executed before any other hook can be requested. All other hooks are declared and assigned as per appointed variables during the accelerant initialization process and of course the availability of that function.

The BeOS R5 supported hooks are:  
**Initialization:**

*   `INIT_ACCELERANT`
*   `CLONE_ACCELERANT`
*   `ACCELERANT_CLONE_INFO_SIZE`
*   `GET_ACCELERANT_CLONE_INFO`
*   `UNINIT_ACCELERANT`
*   `GET_ACCELERANT_DEVICE_INFO`
*   `ACCELERANT_RETRACE_SEMAPHORE`

  
**Mode configuration:**

*   `ACCELERANT_MODE_COUNT`
*   `GET_MODE_LIST`
*   `PROPOSE_DISPLAY_MODE`
*   `SET_DISPLAY_MODE`
*   `GET_DISPLAY_MODE`
*   `GET_FRAME_BUFFER_CONFIG`
*   `GET_PIXEL_CLOCK_LIMITS`
*   `MOVE_DISPLAY`
*   `SET_INDEXED_COLORS`
*   `GET_TIMING_CONSTRAINTS`

  
**Powersave functions:**

*   `DPMS_CAPABILITIES`
*   `DPMS_MODE`
*   `SET_DPMS_MODE`

  
**Cursor management:**  
_The cursor management hooks are only exported when the accelerant and the card support a hardware cursor._

*   `SET_CURSOR_SHAPE`
*   `MOVE_CURSOR`
*   `SHOW_CURSOR`

  
**Synchronization of the acceleration engine:**

*   `ACCELERANT_ENGINE_COUNT`
*   `ACQUIRE_ENGINE`
*   `RELEASE_ENGINE`
*   `WAIT_ENGINE_IDLE`
*   `GET_SYNC_TOKEN`
*   `SYNC_TO_TOKEN`

  
**2D acceleration:**  
_The first four 2D hooks are used by the app\_server. All hooks can be used by applications, such as BwindowScreen, for example._

The 2D hooks are requested by the app\_server after every `display_mode` switch, and applications should do the same if they use them directly. This way, a different routine can be used to execute the acceleration function, depending on the architecture of the engine.

It's possible that in another colourdepth a different setup is required. It's also possible NOT to execute the function by not returning a hook. It is expected of the app\_server or application to do it by itself.

*   `SCREEN_TO_SCREEN_BLIT`
*   `FILL_RECTANGLE`
*   `INVERT_RECTANGLE`
*   `FILL_SPAN`
*   `SCREEN_TO_SCREEN_TRANSPARENT_BLIT`
*   `SCREEN_TO_SCREEN_SCALED_FILTERED_BLIT`

  
**Hardware overlay:**  
_The following hooks are used for hardware overlay. Depending on the architecture of the engine different functions can be used in different colordepths. The overlay hooks below can be requested again after every modeswitch!_

It's permitted to either export the hooks or not to export them. In this way the accelerant can tell you if hardware overlay is supported in that mode or on the card.

*   `OVERLAY_COUNT`
*   `OVERLAY_SUPPORTED_SPACES`
*   `OVERLAY_SUPPORTED_FEATURES`
*   `ALLOCATE_OVERLAY_BUFFER`
*   `RELEASE_OVERLAY_BUFFER`
*   `GET_OVERLAY_CONSTRAINTS`
*   `ALLOCATE_OVERLAY`
*   `RELEASE_OVERLAY`
*   `CONFIGURE_OVERLAY`

The most important hooks will be discussed in the next section.

##### 4.1.1 - INIT\_ACCELERANT

This hook is called first. This hook will request the `shared_info` from the already loaded kernel driver and make a clone of it for its own use. If the kernel driver doesn't belong to the accelerant, it will be reported as such and stopped. If successful, the card will be initialized, and either a 'cold start' or 'hot start' will be executed. The video RAM location and visible framebuffer and cursor data is determined next. The needed semaphores are made, and all variables are initialized.

The accelerant creates a mode list of the available modes that the card supports. This list is made accessible through the `shared_info` structure. The maximum possible modes listed is integrated in the code of the accelerant, and is validated through the `PROPOSE_DISPLAY_MODE`. That means that a model is created for the currently controlled card. As such, the maximum DAC speed and the ammount of video RAM are of big influence for the supporting modes.

The app\_server in BeOS requires that for each supported resolution in the BeOS screen preferences panel at least one mode is in the list. If the resolutions are not provided they will be greyed out. It's possible to have multiple modes per resolution, and they can have different colordepths, refresh rates, and monitor timings (different relative place and length of the horizontal and vertical sync signals).

The mode list can later be requested by a hook that will use it as a reference. In this way the app\_server also can obtain the list.

**Remarks:**  
As stated in the R4 graphics driver kit, during the creation of the `display_mode` list in the internal function `create_mode_list()` two things have to be done different:

*   Implicitly point out that some hardware needs a bigger pitch (distance between two lines) to make a mode. The author says that it needs to be done with `virtual_width`. This is NOT the case. The so called 'slopspace' as what they refer to has to be done with `frame_buffer_config.bytes_per_row`. Only in this way will the app\_server correctly use the 'slopspace'. In other cases a virtual screen is created which can be panned.
*   Calling the `PROPOSE_DISPLAY_MODE` as pointed out isn't correct. While in the correct way a margin is given to `PROPOSE_DISPLAY_MODE` for the asked for pixel clock, it is not considered if the mode can be made within the given limits. Instead of taking over the mode if `PROPOSE_DISPLAY_MODE` doesn't return a `B_ERROR`, the mode only can be taken over if the function returns a `B_OK` status.

##### 4.1.2 - CLONE\_ACCELERANT

When the driver and accelerant is running and another application wants to load the accelerant, it must be done through the `CLONE_ACCELERANT` hook. Because the card is already initialised there's not much to be done.

First, the hook will open the kernel driver (for the 2nd or 3rd, etc). After this `SHARED_INFO` is requested and cloned. After that the previously made list will be cloned as well.

##### 4.1.3 - UNINIT\_ACCELERANT

When the current accelerant is the original accelerant this function destroys the semaphores that the accelerant uses, and also the cloned `shared_info` and `mode_list`. When it's a clone accelerant, the cloned `shared_info` and `mode_list` will be destroyed and the kernel driver will be closed.

##### 4.1.4 - ACCELERANT\_RETRACE\_SEMAPHORE

This hook only returns the semaphore that the kernel driver made for synching with the vertical retrace, which is important for interference free displaying of video (as example). If there's a need for synchronisation by an application, a semaphore lock is required. This will not succeed until the kernel driver releases it during the retrace interupt routine.

##### 4.1.5 - ACCELERANT\_MODE\_COUNT & GET\_MODE\_LIST

`ACCELERANT_MODE_COUNT` returns the number of modes that are in the `MODE_LIST` (made during the 'original' accelerant initialization). The application (or app\_server) which wants requests the `MODE_LIST` has to allocate enough space for the accelerant to make a copy of the list which is returned via `GET_MODE_LIST`.

##### 4.1.6 - PROPOSE\_DISPLAY\_MODE

##### 4.1.7 - SET\_DISPLAY\_MODE

##### 4.1.8 - GET\_FRAME\_BUFFER\_CONFIG

##### 4.1.9 - GET\_PIXEL\_CLOCK\_LIMITS

##### 4.1.10 - MOVE\_DISPLAY

##### 4.1.11 - SET\_INDEXED\_COLOR

##### 4.1.12 - GET\_TIMING\_CONSTRAINTS

##### 4.1.13 - SET\_CURSOR\_SHAPE

##### 4.1.14 - MOVE\_CURSOR

##### 4.1.15 - 2D Accelerant Functions

##### 4.1.16 - Hardware Overlay Functions

#### 4.2 - Conclusion
