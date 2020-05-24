gokrazy - a pure-Go userland for your Raspberry Pi 3 appliances

# gokrazy is a pure-Go userland for your Raspberry Pi 3 appliances

For a long time, we were unhappy with having to care about security issues and Linux distribution maintenance on our various Raspberry Pis.

Then, we had a crazy idea: what if we got rid of memory-unsafe languages and all software we don’t strictly need?

Turns out this is feasible. gokrazy is the result.

#### Your app(s) + only 4 moving parts

1. the [Linux kernel](https://github.com/gokrazy/kernel)

2. the [Raspberry Pi firmware files](https://github.com/gokrazy/firmware)

3. the [Go](https://golang.org/) compiler and standard library

4. the gokrazy userland

All are updated using the same command.

#### Web status interface

On a regular Linux distribution, we’d largely use systemctl’s start, stop, restart and status verbs to manage our applications. gokrazy comes with a [convenient web interface](https://gokrazy.org/overview.png) for seeing process status and stopping/restarting processes.

#### Debugging

Sometimes, an interactive `busybox` session or a quick `tcpdump` run are invaluable. [breakglass](https://github.com/gokrazy/breakglass) allows you to temporarily enable SSH/SCP-based authenticated remote code execution: scp your statically compiled binary, then run it interactively via ssh.

#### Constraints

Due to no C runtime environment being present, your code must compile with the environment variable `CGO_ENABLED=0`. To cross-compile for the Raspberry Pi 3, use `GOARCH=arm64`. If your program still builds, you’re good to go!

#### Network updates

After building a new gokrazy image on your computer, you can easily update an existing gokrazy installation in-place thanks to the A/B partitioning scheme we use. Just specify the `-update`flag when building your new image.

#### Minimal state and configuration

A tiny amount of configuration is built into the images (e.g. hostname, password, serial console behavior). In general, we prefer auto-configuration (e.g. DHCP) over config files. If you need more configurability, you may need to replace some of our programs.

© 2017 The gokrazy authors