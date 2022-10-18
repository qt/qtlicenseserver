// Copyright (C) 2022 The Qt Company Ltd.
//

Qt Licenser daemon

- Tested in Ubuntu 16.04, Mint 20.3, Macbook Pro M1 (Mac from console only, not as a daemon)
- This README does not include Windows version, because it's not supported yet

Prerequisites:
    cmake, curl, build-essentials

To build, run the build script:  (from project root)

    ./build.sh


----------  Install it as a service: ----------

cd to deploy/ directory (created by build.sh):
    cd deploy

At this point, you might want to edit the licd.ini file, where most important is the License Server address to be able to connect to it. If you do this later, you need to edit /opt/licd.ini as root. See licd.ini in deploy directory.

Run installation script as root:
    sudo ./installer.sh

(will now start automatically after reboot, too)

To get MOC wrapper working, too, please note: Installation script does not know your Qt installation location (Something like /home/user/Qt/6.3.1/gcc_64/libexec/). That's why you need to set MOC wrapper working by doing:
    1. cd [your Qt framework install dir]
    2. mv moc orig_moc   <--- Note: has to be exactly 'orig_moc'
    3. ln -s /opt/licd/mocwrapper moc
    4. Try it by building any Qt-enabled app wit any IDE or from command line

In case youre using Qt5, you need to enable legacy support for its qmake licensing system by replacing licheck binary with the provided one:
    (Still in the Qt installation location):
    1. cp licheck licheck.bak   <--- This is just to back up the original, you can name it freely
    2. ln -s /opt/licd/licheck licheck


Check if the service running:
    sudo systemctl status licd

To stop it:
    sudo systemctl stop licd

to disable it:
    sudo systemctl disable licd

To view logs:
        journalctl -u licd -n 100
        (Displays last 100 lines. If -n is not given, it will show all the logs)

To run manually (easier to see what happens while testing, as logs will print out directly in the console):
    - sudo systemctl stop licd   <--- Don't forget to stop all other licd instances as well
    - cd <project root>/deploy
    - OR:
    - cd /opt/licd  <-- Settings will always be taken from /opt/licd/licd.ini
    - sudo ./licd     <--- 'sudo' because licd needs to have write access to its directory

To stop it (manually):
    killall licd  <-- Not that this does not work if running as a system service

To completely uninstall, run 'uninstaller.sh' from deploy directory (as root, of course)
    In addition, you need to remove symlinks from Qt installation dir and restore original binaries there:
    - cd <your Qt installation dir>
    - mv orig_moc moc
    - mv licheck.bak licheck    <-- or whatever you named your backup


----------     Test it:   ----------

Start License server on some Windows machine

- If you didn't edit settings after building: Check the ip address of Win box, then copy it  to the line stating 'server_addr' in the settings file (licd.ini). This will be user as the default site license server address. After installation, it must be edited (as root) in /opt/licd/licd.ini

Note that user settings file is not there yet. If you want, you can create it now by running MOC from your Qt installation dir:
    - <your Qt installation dir>/moc   <--- Use the absolute path here!!!
MOC saves the settings file in <your home dir>/.local/share/Qt/qtlicenseservice.ini and advises you to review/edit it with any text editor. Settings file is divided into installation-specific sections named after Qt installation path. At first run, there is only [default] section - Edit the settings there to suit your needs. On the next run, your installation-specific section is created and contents is copied from [default] section. Then, if you do not use any other licenses, three is no need to edit them again.

After getting those settings edited, you can try to compile any Qt-enabled app, and it should go through. If you did't start MOC "manually" in the previous step thus the settings file is missing, MOC will give you the similar response now, asking you to review and edit the settings. Build again when done with it.

- To test long term licenses, say:
        qtlongterm <add/remove> -u <username> -i <license id> -d <daemon address> [-l <license server address>]
          - And note that the license server address (-l) is optional
    - For example, adding a longterm reservation:
        qtlongterm add -u sami -i 12345678 -d localhost:60000
    - To release it, use 'remove instead of 'add'
    - To get help:
        qtlongterm --help

- Test MOC and plugin interfaces without actually using MOC or Qt Creator (plugin):

Open telnet (you may need to install it first: sudo apt install telnet)
    telnet localhost 60000 (in linux)

After connecting, issue a command like:
    license -u foobar -a QtCreator -v 6.3 -i 123456
        Where: -u = username, -a = app in use, -v app version and -i Qt license ID
        Optionally, you can also give -l switch followed by license server address, like: -l 10.0.10.234:8080

Reply is then info txt about having/not having a valid license, or "Bad request" for malformed message.
