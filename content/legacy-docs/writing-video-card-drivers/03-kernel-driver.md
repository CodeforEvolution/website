+++
type = "article"
title = "Kernel Driver"
date = "2010-05-26T02:18:31.000Z"
tags = []
+++

### 3 - Kernel Driver

The part of the graphics driver that exists in the kernel space serves one purpose, to give the accelerant access to the video card. During video driver startup, the kernel driver 'transfers' as much resources as possible to the accelerant in order to allow the accelerant direct access to the video card, without taxing the kernel any further. Resources and information that can not be transferred are instead accessed through the API of the kernel driver component in simple functions.

This split method accomplishes two major things that can not normally be achieved by normal drivers: optimal speed, and stability. Optimal speed is reached because there are barely any (slow) context switches between user and kernel space. Stability is achieved because if the accelerant happens to die, it does not bring the system down since it resides in user space.

The kernel driver has two interfaces

*   the interface to the OS ("system hooks") in kernel space that is responsible for the driver initialization and publicizing of the driver name to the system
*   the interface to the accelerant ("user hooks") in user space for actual use of the driver

The following picture of the startup method of BeOS (in relation to video drivers) shows among other things how the kernel driver is loaded. First the kernel drivers are initialized via the "system hooks". For drivers that succeed (ie: there is supported hardware found) the "user hooks" can then be used.

![3. Basic BeOS boot sequence in relation to video drivers.](/files/images/writing-video-card-drivers/3.kernel.driver.png)

After the kernel driver is loaded all known accelerant's are tried via the user interface of the **first found working kernel driver**. This process is continued until an accelerant reports a `B_OK` status, meaning that it can work with the loaded kernel driver. After this the rest of the loading process continues as usual, ending when BeUserScript script is run in `/boot/home/config/boot/`. At this point, the user is usually moving the mouse and attempting to use the system.

For the rest of this chapter, we will discuss the system and user interfaces of the kernel driver. Information on the accelerant is found in chapter 4 - [Accelerant](/legacy-docs/writing-video-card-drivers/04-accelerant).

#### 3.1 - Interface To The OS

A kernel driver is loaded during the startup of the OS, is started by the OS, and is published to the user space, which can access it via the accelerant using "user hooks".

To do this the kernel driver needs a few standard routines:

*   `init_hardware()`
*   `init_driver()`
*   `uninit_driver()`
*   `find_device()`
*   `publish_devices()`

Kernel drivers are found in two locations in BeOS:

*   `/boot/beos/system/add-ons/kernel/drivers/bin/` (standard with the system supplied drivers)
*   `/boot/home/config/add-ons/kernel/drivers/bin/` (for user add-on drivers)

System kernel drivers are linked to by a symlink, located in `/boot/beos/system/add-ons/kernel/drivers/dev/graphics/`, and user add-on drivers are linked to by a symlink, located in `/boot/home/config/add-ons/kernel/drivers/dev/graphics/`.

If a problem appears with a particular add-on during boot, a configuration screen can be accessed by pressing the spacebar at the **very beginning** of the iconic boot screen. Choose "Safe Mode Options" -> "Disable User Addons", and continue booting. The system will then not attempt to load any add-ons, and allow you to manually remove the offending driver in `/boot/home/config/add-ons/kernel/drivers/bin`. If no working kernel driver is found, BeOS will boot into a black and white failsafe video mode, allowing you to manually check what is wrong.

##### 3.1.1 - init\_hardware()

If all the installation demands are met, the system will attempt to initialize the kernel driver during startup by calling `init_hardware()`. This routine will check if the required bus system (e.g.: ISA and/or PCI) is available, and naturally, if a supported device is present.

##### 3.1.2 - init\_driver()

On success of `init_hardware()`, `init_driver()` will be called. With this some needed variables are initialized, like the name of the present device, and the required bus will be opened for use.

##### 3.1.3 - publish\_devices()

The system will now call the names of the supported hardware via `publish_devices()`. The names will be published as symbolic links in the file system, in the `/dev/` directory. Where this symbolic link points to depends on the corresponding driver location, which was explained in section 3.1 above.

The driver is supposed to generate and publish a "unique" name for the card it is bound to. Usually the driver uses the manufacturer ID, card ID, bus number, slot number, and in some cases a function number, to generate it's "unique" name. Also of relevance is the physical slot the video card is seated in. Therefore creating unique names for two exact cards is not an issue, since they are seated in two different card slots.

The driver will be loaded only once into kernel space, and will support a predefined maximum number of different cards. That number is programmed into the driver itself, when it was created by the driver author.

**Some examples:**

*   A single unified driver has been created to deal with multiple graphics cards by the same vendor, strictly PCI. The driver author has decided to limit the number of cards of this type to thirty (30) instances at one time, for whatever reason. If the computer has 3 PCI cards of this make/model, there will be three entries in `/dev/`.
*   A single unified driver has been created to deal with multiple graphics cards by the same vendor, regardless of bus type. The driver author decides using 30 is still a good limit, and uses that. The computer happens to have one AGP video card, and 4 PCI video cards in it. (Lucky computer user. ;) What will show up in the `/dev/` path will be **five** entries for this driver.

Every "filename" in the `/dev/` hierarchy ressembles a supported device that can be used by the system, using the unique name assigned to it by the driver. If however `init_hardware()` was unsuccessful then no names will be published in the `/dev/` hierarchy at all.

When a second accelerant tries to control the same video card, the kernel driver will keep a "times opened" counter, and report that the driver and video card were already initialized.

##### 3.1.4 - uninit\_driver()

When a kernel driver is no longer required, and the accelerant has performed the final close routine for the driver, the operating system will call `uninit_driver()`. This routine releases all used resources and closes any open busses used by the driver.

##### 3.1.5 - find\_device()

The `find_device()` routine returns all routine names associated with the "unique name", and makes it ready to be used by "driver hooks". These "driver hooks" are the interface to the user, used to access the driver in the kernel.

#### 3.2 - Interface To The User

Once the kernel driver is initialized by the operating system, it can be accessed by using the following hook routines:

*   `open_hook()`
*   `close_hook()`
*   `free_hook()`
*   `control_hook()`
*   `read_hook()`
*   `write_hook()`

With this a driver can be opened and closed, can be written and read, and controlled dierctly. The hooks are based on a concept similar to file access: a driver can open or close the driver-filename in the `/dev/` hierarchy.

**Applications of a kernel driver as a part of a video driver.**

For a video driver only the following routines truly work:

*   `open_hook()`
*   `free_hook()`
*   `control_hook()`

The remaining routines are simply not used, according to the BeOS Documentation.

For the video driver the accelerant portion is considered the "user", unlike other driver types where an application can use the remaining hook routines directly, such as `write_hook()`. An example of a driver that can use all hook functions would be a mass storage driver.

The `control_hook()` routine determines when an accelerant trying to access the kernel driver if it is allowed to do so. Input/Output commands can be given for ISA or PCI operations too.

The kernel driver must recognize the supported card, map it into system memory, assign ISA and PCI "configuration space", assess what Input/Output commands are available, and be prepared to deal with the accelerant's use of "user\_hooks". One more thing to be aware of, is that IRQ's can be accessed in kernel space too, more of which will be explained later.

Resources such as PCI Input/Output addresses are mapped into system memory, and in some cases, the entire video memory can be allocated in system memory, further speeding up video operations. By allocating these resources into system memory, they become available to the accelerant, without having to go through the (slow) kernel driver repeatedly. Any registers that the video card has may optionally be placed into memory, depending on if the programmer chooses to do so, the video card make/model, and if it is a PCI or AGP video card. More of these concerns are covered later.

##### 3.2.1 - open\_hook()

The `open_hook()` routine makes a "shared info" `struct` that can be accessed by all accelerants, and contains the card's ID, the manufacturer, card specifications (DAC speed for each color depth, memory and core speed), and display settings available by it.

If possible a semaphore is created that later can be used to synchronize with the vertical retrace of the video card, using an interrupt handler that allows for capturing and release of the semaphore. The BeOS makes sure (since the PR2 days) that the IRQ handler is installed in such a way that in the event of a shared interrupt line (a common occurrence with PCI cards), all routines called by all affected drivers are "daisy-chained". A status return type is used that indicates how the interrupt was handled, and if intended for a particular card or routine.

##### 3.2.2 - close\_hook()

The `close_hook()` routine closes the driver in principle, but according to the BeOS documentation, it does not work for video drivers, instead returning an error status.

##### 3.2.3 - free\_hook()

The `free_hook()` routine closes the kernel driver. The number of times that the driver has been opened or closed is logged by the operating system and when the kernel driver is closed for the last time, the `free_hook()` routine is called, and the driver is actually closed for real. This means that everything that was done from `open_hook()` onwards is done in reverse order, finally ending in `free_hook()`: The interrupts are cleared, the outstanding interrupts are deleted, the interrupt routine is deinstalled, the retrace-semaphore deleted, the video card memory "unmapped" and the "shared info" `struct` destroyed.

##### 3.2.4 - control\_hook()

As mentioned before the `control_hook()` routine is called whenever any accelerant attempts to access it's matching kernel driver portion. The only public function that the accelerant can use is the `B_GET_ACCELERANT_SIGNATURE` constant, which is not a real function, but selected with a switch() statement.

Other functions in the control\_hook() are for instance execution of IO commands for ISA or PCI busses (such as PCI configuration space I/O commands), and it is possible for the accelerant to call for a private struct ("shared\_info") to get all information about the card that is controlled by the driver (and other instances of the accelerant). It is also possible to activate and deactivate interrupts via the control\_hook() routine if implemented.

##### 3.2.5 - read\_hook()

The `read_hook()` routine is not used, returning instead a `B_NOT_ALLOWED` status.

##### 3.2.6 - write\_hook()

The `write_hook()` routine is not used, returning instead a `B_NOT_ALLOWED` status.

#### 3.3 - Conclusion

The kernel driver interface is well thought out. This is not really surprising given how long it has existed in BeOS. Part of the structural concepts were borrowed from the linux world, in which it has been around even longer, and has over the years developed into a solid implementation of how drivers should behave.

The construction of a kernel driver portion for a video driver is not very exciting, because most of the actual functionality exists in the accelerant. It is important to pay attention however when no separation needed programming the kernel driver, because one small mistake can crash the entire system. The kernel is special indeed. The lockups etc. are much more common in the accelerant ;-)
