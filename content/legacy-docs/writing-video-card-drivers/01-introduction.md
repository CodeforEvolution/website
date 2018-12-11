+++
type = "article"
title = "Introduction"
date = "2010-05-26T02:06:32.000Z"
tags = []
+++

### 1 - Introduction

The are a lot of programmers in the world but almost none of them know how to write a driver. Drivers are required for any operating system, in order to talk to the hardware. The lack of knowledgeable driver programmers is a serious problem, and one that I hope to rectify with this document.

This document offers practical support on this issue, and is written for people who are interested in knowing basic information about the makeup of a driver and how to write it. The basics of writing a video card driver will be explained here, along with directions to follow, using the Be Operating System (BeOS henceforth) as our working environment.

Because most programmers do not have a lot of hardware knowledge, some explanation about hardware will be provided. Theory will be supported with examples, and the most important issues discussed thoroughly. [Appendix B](/legacy-docs/writing-video-card-drivers/appendix-b-glossary) is a Glossary of common terms used throughout this document, which tries to explain as best as it can basic hardware terminology.

This document describes the various parts of a driver, and how these parts are woven into the BeOS. The layout and relationship of the Application Programming Interface (API henceforth) and the driver interface will be explained. To do this the API "hooks" will be explained for the various driver components. Finally the "flag" level of the driver interface will be explained in detail, as it has an important influence on the operation(s) of the API.

We will try to look from the perspective of a driver writer, so we'll provide more details about driver hooks than information about the API. An excellent reference to the BeOS API is [The BeBook](/legacy-docs/bebook/index.html). This document contains important supplemental information, because the BeBook is not complete enough regarding graphics card drivers. Some things are not described in the BeBook at all, while other things that are need extra explanation.

Modern graphics cards are built more or less on the VGA standard, and still have VGA modes built in. Because the VGA standard is still (patently) in affect, the documentation for old VGA standard cards is invaluable for writing video card drivers. A good reference is the "[Programmer's Guide to the EGA and VGA Cards](https://www.amazon.com/exec/obidos/tg/detail/-/0201624907/)" (any edition). Technical information on PCI and AGP busses, and an example of a 2D accelerator card are described in "[The Indispensable PC Hardware Book](https://www.amazon.com/exec/obidos/tg/detail/-/0201596164/)".

Important to note is that this document does not claim to be 100% complete or accurate. It is impossible to know everything about drivers, hardware, or programming. Luckily it is not required to know everything in order to write a good graphics card driver.

**The structure of this thesis:**  
To explain the creation of graphics card drivers in BeOS as quickly as possible I chose the top-down approach. First some basic concepts will be explained, and then we will gradually go deeper into the more difficult material. The parts of the API that are relevant will be explained and after which the driver that lies deeper in the system will be described in detail.

When a video driver is written for BeOS, about 5% of the time will be spent on the kernel driver, the other 95% of the time on the accelerant. Therefore the major part of this thesis will be related to describing the accelerant.

In addition to the knowledge in the thesis for the reader, we will include a workable method (roadmap) for writing and testing video drivers in BeOS. When someone actually wants to write a video driver this will be a most important part of this document. The roadmap will point the reader to related, and more useful information that exists elsewhere in this document, when possible.

#### 1.1 - Problem Description

It is difficult to write graphics card drivers. There is also not enough information available for BeOS to be able to build a good video driver without extra information. Because I still wanted to learn how to write such a driver, I invested a lot of time in researching all of the interfaces in BeOS needed of it and in the specific things that are important for the software in the hardware of the graphics card.

This took me at least twenty hours a week for two years. The better part was spent on research and testing because manufacturers of graphics cards release very little useful information. There were also a lot of tests needed to determine how the API and hooks work in detail inside of BeOS.

#### 1.2 - Thesis

The knowledge I've collected is pretty specialized and is not known by a lot of other people (in the BeOS Community). The assignment is defined therefore as follows:  
The spreading of the knowledge acquired over the writing of graphics card drivers in the BeOS Community, and making a workable action plan available.

**Knowledge transition.**  
The thesis will be published on the internet.

#### 1.3 - About The Author

From elementary school onwards I was a real "technician". Once when I was ten, I found a broken transistor radio in the attic and as soon as I experimented with is I became hooked on electronics.

After I completed my studies in MTS and HTS electronics, I found out that my interest was shifting to "information technology", This is a good thing, because as a electronics designer you get to work more and more with micro-controllers and PC's, while in the old days I was used to working with hardwired logic devices. Finally I wanted to learn more about computers and technical "informatica" to improve my knowledge base when working in the field in between electronics engineering and software engineering.

At this moment I am interested in the flip-side of PC's. This means that I want to work with peripherals like USB or PCI cards. But it is difficult for two reasons:

*   The manufacturers of the equipment in effect hamper by giving no information about the products they're making. Even the most basic information is hard to find.
*   The writing of drivers is an area between electronics and "information technology" which requires knowledge of both.

Because this middle area is such a challenge for me I chose to get an education at the Hanzehogeschool Groningen in software engineering (HIO) to enhance my knowledge in the software field.

What are the upsides of education in regards to the electronics and information technology areas?

*   **The technical side.**  
    Especially the theory of the third year: everything surrounding JAVA. Not as a language but as a tool to the explanation of different principles. Multi-threading and the use of semaphores are everywhere in the system: even in drivers. Object oriented programming was also interesting because application software (read: BeOS) is built like this.
*   **The Lowlevel side.**  
    For drivers in general this kind of knowledge is important. An example would be for network card drivers. The Networking and Communication course from the forth year was indispensable. The knowledge of Intel x86 architecture did help, and was especially handy for reverse engineering.

#### 1.4 - About BeOS

BeOS stands for The "Be Operating System", designed by Be Inc., which was housed in Menlo Park, California, USA. Be Inc. no longer exists as they were unable to get enough OS market share, and due to circumstances beyond their control related to Microsoft.

Many components of the BeOS can be found on BeBits, an online file repository for all BeOS software. The company is "kept alive" while the ongoing lawsuit against Microsoft and possible positive financial results take their course, in the best interest of the shareholders.

BeOS is according to a lot of people a schoolbook example of an OS : straight to the point. Free of legacy problems. With a well thought out API. The best pieces of many different OS worlds have been put together into one easy to use operating system, specifically designed for multimedia applications. Server functionality was not a top priority when BeOS was first designed, but with projects such as openBeOS, and Zeta from YellowTab, these issues could be resolved. The one key aspect of the BeOS that most people agree with is that it just works. No fuss no muss. Very consistant.

BeOS has a small but dedicated Community of people who hold dear the values that make up the OS. When Be Inc. sold the Intellectual Property of BeOS to Palm Inc in 2001, members of the BeOS Community decided to recreate an open source version of BeOS R5 Pro. The openBeOS project was born (after a time in finding of a new (licenced free) project name openBeOS was renamed to [Haiku](https://www.haiku-os.org)).

The group did not have to start from scratch however, as some parts of the OS like Desktop and Tracker were previously open sourced by Be Inc. Separate from the openBeOS project are other commercial attempts to continue BeOS. YellowTab Inc. is working on Zeta, a version of BeOS that would have been how BeOS R6 would have looked. Before the Be Inc. - Palm Inc. deal, YellowTab has arranged through a contract deal to license various parts of the operating system, and continue developing it. This frees them of any legalese from Palm Inc., and allows the ideas and concepts inherent in BeOS to live on commercially.

New drivers and applications must be developed. Most developers know how to build applications, but there are not enough that know how to create drivers. The holes have to be closed somehow. This is where a document like this might help.

#### 1.5 - About Video Card Drivers

**In general**  
A driver is used by applications, and the OS itself, to allow access to the hardware. Drivers also sometimes incorporate complete functions for setup up (parts of) the hardware. Because drivers are sort of a part of the OS, reliability is a big factor for the stability of the system as a whole.

Drivers are for example used for:

*   network cards
*   sound cards
*   modems
*   keyboard and mouse
*   graphics cards

Because video works with enormous amounts of data in very short timespans (e.g. displaying moving pictures), graphics card drivers are a special breed of driver.

The underlying hardware is also kind of special: in this case among the add-on cards used in the computer. The PCI bus is a bottleneck as far as the video card is concerned, as it has low bandwidth available, hence the creation of the AGP bus. The single AGP slot is kind of an extended PCI slot. AGP has a higher bus speed, it can supply more power and more bandwidth to the card, is closer to the processor, has extra operating features, and has a dedicated controller. PCI slots usually have to share one single PCI controller which therefore limits the bandwidth available even further.

**BeOS Video Drivers:**  
Most drivers in BeOS consist of only one single file: the kernel driver. The video driver however consists of two parts: the kernel driver and the accelerant.

The kernel driver however is used to just give the accelerant access to the video card. On startup the accelerant gets as much resources from the kernel driver as is possible so that the accererant can use the many video features without actually using the kernel driver. The rest of the resources can easily be accessed through API functions of the kernel driver. This method has two upsides: speed, and stability because the kernel driver is not used very much.

Drivers for BeOS are largely written in the C language, as it is very close to the hardware, and is fast. Applications are written in C++ because object oriented programming and multithreading can be applied easily.

BeOS R5 supports the following display features:

*   setting display modes
*   hardware based scrolling and panning in virtual screens
*   hardware based cursors
*   hardware based 2d acceleration
*   hardware overlay (motion video)

Hardware 3D acceleration is not suported in BeOS R5, only software emulation of [openGL](https://www.opengl.org/) is. (There are two commonly used 3D acceleration standards in the world: openGL is a standard that is used on different "platforms", DirectX is a closed standard owned by Microsoft and used for 3D acceleration, etc. DirectX is only used in windows at this time.) Hardware acceleration with OpenGL was planned for BeOS R5 Pro/PE, but was delayed and eventually abandoned.

As a guideline for testing and determining how graphicscard drivers work, the BeOS R4 Graphics Driver Kit is used as a reference. Its the youngest known version of a description of graphicscard drivers by Be Inc.

**Access To BeOS Graphics Card Drivers**  
There are many ways to get access to (parts) of the graphics card in BeOS. This has been done to ensure the best speed and simplicity was reached. The picture below illustrates all of the different ways possible to access the hardware.

![1.5 Access To BeOS Graphics Card Drives](/files/images/writing-video-card-drivers/1.5.access.png)

The usual way to gain access to graphics card hardware for applications or the application server is through the primary accelerant and the kernel driver.

For special applications there are different ways possible:

*   An application can get direct access to the video memory via the classes `BWindowScreen` or `BDirectWindow`.
*   An application can get direct access to the accelerant for 2D acceleration via `BWindowScreen`. (The application server shields some other functions however.)
*   The application server (app\_server) can get direct access to the video memory via the `frame_buffer_config` struct. This struct can be obtained with the hook `GET_FRAME_BUFFER_CONFIG`.
*   The accelerant has direct access to the video memory and all mapped registers. For the accelerant this is the most commonly used method for direct access to the hardware. The kernel driver gives the accelerant the needed information for direct acces via the struct `shared_info`. This struct will be obtained by the accelerant when initialization is done via the kernel driver user hook "control"
*   In addition to the above mentioned access to video memory and mapped registers the kernel driver also has access to I/O commands. With this the kernel driver can also access registers that can not be mapped. The possibilities to map certain registers is determined in the hardware.

A different way an application can get access to the graphics card is via direct load of a "private" accelerant. When an accelerant for that card is already loaded, the application is provided with a "clone" accelerant. This clone has the same information as the original accelerant, therefore they can work next to each other using the same kernel driver.

