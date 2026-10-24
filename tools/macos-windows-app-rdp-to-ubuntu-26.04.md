# Remote Desktop from macOS 13 (Windows App) to Ubuntu 26.04

> Getting Ubuntu 26.04 to accept remote desktop connections from the Windows App on macOS 13, and the errors I hit on the way.

If you get errors when connecting from macOS to Ubuntu 26.04 over remote desktop, this might help. My setup: macOS 13 with the Windows App, connecting to a physical Ubuntu machine that has no display attached.

For the first setup, you need to connect a keyboard, mouse and display to the Ubuntu machine.

Errors I faced:

1. Confusing setup
2. GNOME service not running
3. Credentials not working
4. Total black screen, with no cursor

## Steps to set up

### On Ubuntu

1. Go to Settings > System > Remote Desktop.
2. Open the **Remote Login** tab (not Desktop Sharing). This was confusing for me, but you only need to turn Remote Login on.
3. Set a separate username and password specifically for remote login. Do not use your current login username and password.
4. Reboot.
5. Check that port 3389 is open:

   ```bash
   sudo ss -tulnp | grep :3389
   ```

   OK result:

   ```text
   tcp   LISTEN 0      5                  *:3389            *:*    users:(("gnome-remote-de",pid=1111,fd=9))
   ```

6. Check that gnome-remote-desktop is running:

   ```bash
   systemctl --user status gnome-remote-desktop
   ```

   OK result:

   ```text
   ● gnome-remote-desktop.service - GNOME Remote Desktop
        Loaded: loaded (/usr/lib/systemd/user/gnome-remote-desktop.service; enabled; preset: enabled)
        Active: active (running) since Sat 2026-10-24 21:36:29 +07; 17min ago
    Invocation: 9025....
      Main PID: 3604 (gnome-remote-de)
         Tasks: 10 (limit: 18382)
        Memory: 11.5M (peak: 12.3M)
           CPU: 96ms
        CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/gnome-remote-desktop.service
                └─3604 /usr/libexec/gnome-remote-desktop-daemon

   Oct 24 21:36:29 u... systemd[3371]: Starting gnome-remote-desktop.service - GNOME Remote Desktop...
   Oct 24 21:36:29 u... gnome-remote-desktop-daemon[3604]: [21:36:29:225] [3604:00000e14] [ERROR][com.freerdp.crypto] - [x509_utils_from_pem]: BIO_new failed for certificate
   Oct 24 21:36:29 u... gnome-remote-desktop-daemon[3604]: RDP server certificate is invalid
   Oct 24 21:36:29 u... systemd[3371]: Started gnome-remote-desktop.service - GNOME Remote Desktop.
   ```

7. If it is not running, enable it:

   ```bash
   systemctl --user enable --now gnome-remote-desktop
   ```

8. Reboot.
9. Remember: don't log in after the reboot. If you do, log out or reboot again.
10. Now set up and connect from macOS.

Note: to make sure everything runs well, reboot Ubuntu after each step.

### On macOS

1. In Windows App, add a PC profile for your Ubuntu machine.
2. Export the profile to an `.rdp` file.
3. Open the `.rdp` file in a text editor, find and change the line `use redirection server name:i:0` to `use redirection server name:i:1`.
4. Delete the old profile and import the `.rdp` file back into Windows App.
5. Install [FreeRDP](https://www.freerdp.com/) via Homebrew (`brew install freerdp`) or MacPorts (`sudo port install FreeRDP`). In my case on macOS 13, I had to use MacPorts.

   Note: FreeRDP is a library (plus command-line tools), not a regular app. There's nothing to open after installing it; just install it and keep using the Windows App.
6. Connect to your Ubuntu machine and enjoy.
