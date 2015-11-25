check_tor_bandwidth
===================

# Functions
This is a little script rather to monitor a Tor relay but gather performance
data from it.

It retrieves the

* amount of reject rules
* configured relay bandwidth rate
* configured relay bandwidth burst rate
* average read traffic of the last second
* average write traffic of the last second

and prints the result as performance data for your monitoring system (for exampe
your Icinga 1/2 or Nagios).

# Preview

The graphing of the given performance data is done with graphite and grafana.
It can look like this:

![Previewimage](https://blog.veloc1ty.de/bandwidth-small.png)

I think GitHub caches the Image. Periodically updated pictures (every 30 minutes) can be found on my blog:

* Small preview: https://blog.veloc1ty.de/bandwidth-small.png
* Big preview: https://blog.veloc1ty.de/bandwidth-small.png

# How it works

The script connects to Tor's control UNIX socket, fetches and analyzes the
config and then fetch the average bandwith values.

# Setup
## Step 1: Configure Tor

As above mentioned Tor has to provide the control UNIX sockt and currently no authentification is implemented.

Add this values to your torrc file which is usually stored under ``/etc/tor/torrc``:

```
ControlSocket /var/run/tor/control
CookieAuthentication 0
```

**Warning:** This allows unprotected access to the Tor control port which is ok
for me in most of the cases. You have been warned.

Restart Tor afterwards. A UNIX socket should appear under ``/var/run/tor/control``.

## Step 2: Download the script

Clone this repo and copy the script onto your Tor relay server. This is an agent
based check.
The correct place for the script is where the Icinga constant ``PluginDir`` points to.
On my Debian/Ubuntu based systems this is ``/usr/share/nagios/plugins/``.

## Step 3: sudoers entry

Yes, unfortunately this script requires sudo since Tor has a very strict permission checking.
Running with SUID bit doesn't help since that bit is ignored for script files.

Type ``visudo`` in your terminal and add this line:

```
nagios  ALL= NOPASSWD:/usr/lib/nagios/plugins/check_tor_bandwidth
```

**Note:** Please substitue `nagios` with the user your Icinga daemon is running. For me it was ``nagios`` but I saw setups where Icinga runs under ``icinga``.

## Step 4: Create the CheckCommand

Icinga has to know how to call the script. Place this piece of code somewhere:

```
object CheckCommand "tor-bandwidth" {
	import "plugin-check-command"

	command = [ "/usr/bin/sudo",  PluginDir + "/check_tor_bandwidth" ]
}
```

## Step 5: Add the service

Place the following code somewhere:

```
object Service "tor-bandwidth" {
    import "generic-service"

	check_interval = 1m

    display_name = "Tor Bandwidth Average"
    host_name = "mineralwasser"
    command_endpoint = "mineralwasser"

    check_command = "tor-bandwidth"
}
```

**Note:** Substitute ``mineralwasser`` with your values.
