- Comprehensive device information and various tips&tricks can be found in [Hardkernel's wiki](http://odroid.com/dokuwiki/doku.php?id=en:odroid-c2). Please be aware that some of the information does not apply to Armbian (eg. we use a different partition table). Schematics can be found [here](http://dn.odroid.com/S905/Schematic/).
- Idle consumption with legacy image in headless mode (`setenv nographics "1"` defined in `/boot/boot.ini`) and only Gigabit Ethernet connected is between ~2300 mW (@500 MHz) and ~2400 mW (@1536 MHz). Hardkernel provides the possibility to [exceed 1536 MHz max cpufreq](http://odroid.com/dokuwiki/doku.php?id=en:c2_set_cpu_freq) but Armbian refrains from doing so. In case you want to change settings please keep in mind that you might have to adjust both `/boot/boot.ini` and `/etc/defaults/cpufrequtils`.
- The legacy kernel we use implements a few different cpufreq governors that show partially strange behaviour (`interactive` most of the times acting like `performance` for example). Since idle consumption differences between different cpufreq governors are negligible choosing even `performance` seems to be ok. At least `conservative` governor that switches between upper and lower clockspeeds (for details see [here](https://github.com/igorpecovnik/lib/issues/499#issuecomment-253481174)) leads to some USB performance drops while not providing significant savings. In case you activate higher clockspeeds please keep in mind that switching then to `performance` governor is needed since otherwise you might end up with a slower system since the added cpufreq operating points will slow down switching to highest clockspeed when needed.
- If you don't need GbE network transfer speeds switching to Fast Ethernet with `ethtool -s eth0 speed 100 duplex full` saves ~230 mW. Completely disabling Ethernet saves an additional 100mW.
- GbE Ethernet speed should reach 935 Mbits/sec in TX direction. In RX direction with defaults you should get 800 Mbits/sec but with some tuning it should be able to exceed 900 Mbits/sec:
  - echo 32768 > /proc/sys/net/core/rps_sock_flow_entries
  - echo 32768 > /sys/class/net/eth0/queues/rx-0/rps_flow_cnt
- You can save at least 170mW by cutting power to the internal USB hub (and also all USB devices connected to any of the type A receptacles) using `/sys/class/gpio/gpio126` (see description [here](http://forum.odroid.com/viewtopic.php?t=22637&p=151969#p151982)). The same way you have full control over power consumption of a connected host powered USB disk: `umount /mnt/usb && echo 0 >/sys/class/gpio/gpio126/value` and `echo 1 >/sys/class/gpio/gpio126/value && sleep 2 && mount /mnt/usb`
- Reducing DRAM clockspeed to reduce consumption doesn't work (difference between default 912 MHz and 408 MHz is just ~100mW less and also [requires a reboot](http://odroid.com/dokuwiki/doku.php?id=en:c2_adjust_ddrclk))
- the red led is a power led while the blue led is custom. Boot stage: as soon as u-boot is loaded the blue led lights solid and when kernel starts this changes to `heartbeat` blinking with default settings. Check `cat /sys/class/leds/*blue*/trigger` for other functionality.