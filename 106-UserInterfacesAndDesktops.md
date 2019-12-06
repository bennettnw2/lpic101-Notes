# 106: User Interfaces and Desktops
## 106.1 Install and Configure X11

### The Basics of X11
#### * The basics of X11 is that this is how we have graphics displayed on Linux systems.
* the X11 protocol was brought about by "X.org"
  * provides graphical rendering for unix like operating systems
  * X11 is the core display server-system, that provides the protocol service, called X11, for the X Window System
    * the x window system basically just provides 2-D rendering
  * provides extra functionality via extensions, such as:
    * RandR: provides dynamic resizing of the root window, refresh rates, mirroring displays
    * GLX: provides rendering of 3-D OpenGL content within an X11 windows
    * Xinerama: provides the ability to split the desktop display across multiple monitors
  * Each graphical rendering on an X display are clients of the X.org server


#### * What are the building blocks of the basic X11 architecture
* Very simplified rendition
  * we start with the graphics card on the bottom; how we connect the monitor to the computer
  * next is the kernel which by using drivers and modules; will send graphical info to the screen
  * then we have libDRM - Direct Rendering Manager - will communicate with the kernel
    * this library layer sits between the kernel and X-server
    * the kernel received data from the libDRM; (reminds me of a data bus)
  * the next layer is the X-server
    * the config files for X-server are: /etc/X11/xorg.conf and /etc/X11/xorg.conf.d 
    * the X-server receives "drawing requests" from `Xlib or XCB (KDE)`
  * Xlib / XCB
  * Xlib / XCB receives instructions from the display manager(GNOME, KDE, Xfce, etc.)
  * at last you will see your application

#### * Wayland
  * Replacement for the X Window System
  * uses a simpler rendering architecture which has fewer steps than the ones outlined above for X11
  * constantly being improved; new components are added regularly to bring it to feature parity with X.org
  * provides XWayland, a library that enables an X Window client to render with Wayland
    * this provides backwards compatibility

### Installing X11
  * this is done on CentOs 5.11
  * `yum grouplist` will list out all the possible group installs
    * we want to `yum -y groupinstall "X Window System"` to install the X Window System components
  * we can use the telinit command to change to the graphical display (runlevel 5 or graphical.target)
  * `runlevel` will tell us that we are at runlevel 3
  * `telinit 5` or `init 5` will get us there
  * this will then take us to GDM or the Gnome Display Manager
    * the role of the GDM is to handle a user's session and authentication
    * it will also start up the display server (X11) and our desktop
    * you also have the option to shut down or restart your computer
  * try not to ever log into a system as the root user; always go in with a regular, limited user
  * once we get logged in we are greeted by TWM or better known as Tab Window Manager
    * TWM is one of the simplest desktop environments you can use in Linux
    * `iconmanager` is like a taskbar
  * A window manager is what provides the look and feel of a desktop
    * a window manager gives us our title bars, icons, colors, dictates how the mouse behaves when dealing with individual windows
    * we can start the window manager(X11 and TWM) without going through the display manager (GDM)
    * if you run `startx` from runlevel 3(CLI with networking) or 4(custom)
    * this is very unsecure as you bypass the authentication and session management of GDM
  * So how does X11 know how to start GDM rather than just `startx`?
    * first we take a look at the `/etc/inittab` file
    * we will notice this line:
    ```
    # Run xdm in runlevel 5
    x:5:respawn:/etc/X11/prefdm -nodaemon
    ```
    * the `/etc/X11/prefdm` will show a list of display managers and it will go through that list until it finds the one that is installed on the system

  * It is a very similar configuration on modern systems with systemd
    * if you start with your default target unit file `/etc/systemd/system/default.target`
    * we will see `Wants=display-manager.service`
      * next we check out this service unit file located at `/etc/systemd/system/display-manager.service`
    * we will then see `ExecStart=/usr/sbin/gdm`

  * Back to our Centos Machine:
    * note that each window you see is a client of the X windows server
    * there is communication back and forth between the window manager(TWM) and X windows server(X11) regarding:
      * geometry
      * size
      * positioning
    * there is an environment variable that will help us out with things like this `$DISPLAY`
    ```
    echo $DISPLAY
    :0.0
    ```
    * technically there is something to the left of the colon; this is the host that the x server is running on
      * this could be the localhost, loopback address, or blank (as we see here) if the x server is local to the computer 
    * the first 0 indicates which x server is in use, (kind of like TTY numbers?)
    * the second 0 indicates which screen we are using
      * if we had a second monitor this number would be 1, etc

  * Lastly you will want to know that the .xsession-errors is in the home directory
  * this will capture error messages from the display manager to help debug graphical applications
  * this needs to be configured by your display manager

### X11 Configurations
  * this is where we talk about the `xorg.conf` file which is the file we use to configure the xorg server
  * this file is located at `/etc/X11/xorg.conf`
    * There are sections that indicate what to do with things
    * each section is responsible for a function of the xserver
    * xorg.conf and xorg.conf.d are composed of a number of sections which may be present in any order or omitted to use default configuration values.
    * Each section has the form:
    ```
    Secton "SectionName"
        SectionEntry
        ...
    EndSection
    ```
    * The section names are:
    ```
       Files          File pathnames
       ServerFlags    Server flags
       Module         Dynamic module loading
       Extensions     Extension enabling
       InputDevice    Input device description
       InputClass     Input class description
       OutputClass    Output class description
       Device         Graphics device description
       VideoAdaptor   Xv video adaptor description
       Monitor        Monitor description
       Modes          Video modes descriptions
       Screen         Screen configuration
       ServerLayout   Overall layout
       DRI            DRI-specific configuration
       Vendor         Vendor-specific configuration
    ```
  
  * `X -configure` or `xorg -configure` will generate a new `xorg.conf` file in our root's home directory
    * you can test the new config file with `X -config /root/xorg.conf.new`
    * you will be greeted by a blank screen with an "X" shaped cursor
    * this is because no applications have sent any instructions to the x server to draw anything
    * but we know it works!  You can exit out of it with ctl-alt-backspace
    * then you can copy the new config file to its proper place of `/etc/X11/xorg.conf`

  * ##### `xdpyinfo`
    * this will display information about the current X session and X server instance

  * you can make changes to `/etc/X11/xorg.conf.d`
  * reboot the system for the changes to take effect

  * modern xorg servers do not use the `xorg.conf` configuration file any more
  * this is considered old school
  * instead we will have a `/etc/X11/xorg.conf.d/` directory that contains individual configuration files
  * these files will start with double digits as the x server will load them in numerical order
  * the default amount of these files will be small as newer versions of x server are much better at auto detecting hardware
  * if you did have a `xorg.conf` file, it would be loaded last after the ones in the directory

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
    * first get your system's IP address
    * run the command, we now have remote x-connections that are allowed from our local system
  * Next you will ssh into your remote system with a `-Y` (capital Y)
    * eg: `ssh -Y user@193.234.23.03`
    * `-Y` will lets ssh know to forward any X11 requests
  * Once logged into your remote system you will need to set up a display environment variable
    * this display environment variable will redirect graphical requests to the local system
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
  * a desktop environment is typically a collection of software 
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
      * Virtual desktops are known as "Activities" in GNOME
      * Activities overview shows the taskbar and the desktops you can use on the right
      * Ctl+Alt and the up and down arrow keys to move through activities
      * XFCE is very lightweight and very customisable
      * you can choose any desktop environment from the login screen or more technically known as the display manager
      * Note: there are many display managers you are able to use
        * GDM (Gnome Display Manager)
        * SDDM (Simple Desktop Display Manager)
        * LXDM (LXDE Display Manager)
        * LightDM
        * KDM (KDE Display Manager)
        * XDM (X Display Manager)

    * ##### Qt Based Desktops
      * Qt is based on a C++ language library used for graphical applications
      * KDE resembles the windows desktop environment; but that is only on the surface
      * KDE plasma was widgets that you can use to decorate your desktop
      * Alt+F2 will give you a search bar to search your system for either an app or a file
      * KDE will use up much more disk space and RAM than GNOME and XFCE


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
