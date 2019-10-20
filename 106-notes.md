# 106: User Interfaces and Desktops
## 106.1 Install and Configure X11

### The Basics of X11
#### * The basics of X11 is that this is how we have graphics displayed on Linux systems.
* the X11 protocol was brought about by "X.org"
  * provides graphical rendering for unix like operating systems
  * X11 is the core display server system that provides the protocol service, called X11, for the X Window System
  * provides extra functionality via extensions, such as:
    * RandR: provides dynamic resizing of the root window, refresh rates, mirroring displays
    * GLX: provides rendering of DG OpenGL content within an X11 windows
    * Xinerama: provides the ability to split the desktop display across multiple monitors


#### * What are the building blocks of the basic X11 architecture
* Very simplified rendition
  * we start with the graphics card on the bottom; how we connect the monitor to the computer
  * next is the kernel which by using drivers and modules; will send graphical info to the screen
  * then we have libDRM - direct rendering manager - will communicate with the kernel
    * this library layer sits between the kernel and X-server (the actual
    * the kernel received data from the libDRM; (reminds me of a data bus)
  * the next layer is the X-server
    * the config files for X-server are: /etc/X11/xorg.conf and /etc/X11/xorg.conf.d 
    * the X-server receives "drawing requests" from `Xlib or XCB (KDE)`
  * Xlib / XCB
  * Xlib / XCB receives instructions from the display manager(GNOME, KDE, Xfce, etc.)
  * at last you will see your application

#### * Wayland
  * Replacement for the X Window System
  * uses a simpler rendering protocol
  * constantly being improved; new components are added regularly to bring it to feature parity with X.org
  * provides XWayland, a library that enables an X Window client to render with Wayland
    * this provides backwards compatibility

### Installing X11
### X11 Configurations


### Remote Graphical Connections
The X-server is able to allow you to view remote application windows
* ##### `xhost`
  * Q: does this allow you to run remote commands that will show up on a local computer?
    * A: it give us access to individual windows from an x-server
  * one of the oldest methods to do this is to use the `xhost` command
  * `xhost` is very insecure, you do not want to use this for any operation that has sensitive info
  * by default remote connections are disabled by most distributions
  * `xhost +` can enable the connection  
    * this allows all incoming connections to your x-server regardless of where a user is coming from
  * `xhost -` will disable all incoming connections
  * you can run `xhost + [IP ADDRESS]` to enable a specific host to connect 
    * first get your systems IP address
    * run the command, we now have remote x-connections that are allowed from our local system
  * Next you will ssh into your remote system with a `-Y` (capital Y)
    * eg: `ssh -Y user@193.234.23.03`
    * `-Y` will lets ssh know to forward any X11 requests
  * Once logged into your remote system you will need to set up a display environment variable
    * this display env variable will redirect graphical requests to the local system
    * thereby you execute commands on your remote system and they show up on your local system
    * eg: `export DISPLAY=127.0.0.1:10.0`
      * we use the local host IP of the remote system
      * we also use the port 10 as ssh will use this port first when creating X-windows sessions
      * now you can run a graphical application on your remote system and have it displayed on your local machine
        * eg: `xeyes`

* ##### `xauth`
  * allows a user to edit and view security information that grants a user the ability to control remote X11 client windows
  * `xauth list` shows what active, remote x-sessions we have running currently
  * it will list the hostname and the number of the session (default is 10)
  * MIT-MAGIC-COOKIE-1 acts as a password
    * X11 requests will only be granted to us
    * prevents the hijacking of our system and session and snooping
    * the -Y switch when we sshed into our remote system, started the `xauth` for us

* ##### `VNC`
  * VNC will allow us to access a full, remote graphical desktop environment
  * Virtual Network Computing enables a remote computer to control the graphical display of a remote system
  * This is insecure by default
  * There are many different ways to implement VNC
    * on centos7: `yum install tigervnc-server`
    * next you will need to copy over the vnc template service unit file
    * then you configure the service unit to work with the user's account (replace USER with the account username)
    * since we changed the unit file, we now need to reload all the unit files for our changes to take effect
    * we will need to set up a vnc password on our local machine
      * `vncpasswd` will allow us to set up that local vnc password
    * You can connect in two ways to VNC, secure and insecure    
      * for insecure
    * NETWORKING: a service that accepts incoming connections uses a network port to receive those connections
      * you can think of it as a mailbox to receive messages
      * `ss -tlpn | grep vnc` will give what ports VNC is listening on
      * you will then want to make sure that you open the ports through your firewall
    * Once you secure the network for VNC you will use a remote desktop viewer like "vinagre"
      * VNC is a protocol of sorts
      * you will then select the ip and address the port VNC is listening on
      * then you click connect and enter the vnspasswd you create earlier
    * now you are connected although it is an unsecure connection as our traffic is not encrpyted
    * How do we connect securely?  Through an encrypted ssh tunnel, silly!
      * NEED TO COME BACK TO THIS

* ##### `SPICE`
  * Simple Protocol for Independent Computing Environment
  * this allows for remote desktop viewing; typically virtual machines
  * traffic is encrypted by default using TLS; you do not have to set up a secure tunnel
  * however, there is a performance cost to using SPICE
    * but you can get audio through SPICE
    * you can plug a usb into your local computer and it will be available on the remote system


## 106.2 Graphical Desktops
* Linux has many different flavors of desktop environments
* You can use any desktop environment on any distribution
* A desktop environment is a collection of software like:
  * window manager
  * terminal emulator
  * file manager
  * text editors
  * desktop configuration utilities
  * web browsers
  * multi-media players

* There are two main camps in the desktop environment world
  * one is the GTK or Gimp Tool Kit?
  * the other is Qt
    * ##### GTK+ Based Desktops
      * GTK+ is primarily a C language library
      * GNOME and XFCE are built with this
      * Virtual desktops are known as activated in GNOME
      * Activities overview shows the taskbar and the desktops you can use on the right
      * ctl opt up and down to move through activities
      * XFCE is very lightweight and very customisable
      * you can choose different desktop environments from the login screen

    * ##### Qt Based Desktops
      * Qt is based on a C++ language library used for graphical applications
      * KDE resembles the windows desktop environment; but that is only on the surface
      * KDE plasma was widgets that you can use to decorate your desktop
      * Alt+F2 will give you a search bar to search your system for either an app or a file
      * KDE will use up much more disk space and RAM than GNOME and KDE


## 106.3 Accessibility
* taking a look at assistive technologies on the Linux Desktop
* Assistive technologies are tools and utilities that help computer users that have specific needs
* audio needs and visual needs
* all desktop environments can utilize assistive technologies; GNOME, KDE, XFCE
* GNOME in system tools > settings > universal access
* seeing; hearing; typing; clicking are the different categories of types of assistive technologies
* seeing
  * high contrast - makes everything in black and white
  * large text
  * magnifying
  * text reader - ORCA
    * you can configure ORCA from the terminal
    * many many configurations
* hearing
  * visual cues
* typing
  * on screen keyboard
  * key repeat speed
  * cursor blink settings
  * sticky key
  * bounce keys
* pointing clicking
  * use num pad to simulate mouse
  * hover clicks
  * double click delay
* Nightlight mode
  * blue light
