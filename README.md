# Foreman SystemTap tracing utility

Requirements:

* Foreman or Satellite
* RHEL 7.2 (7.0 should work, 6.x will not work)

Usage:

    yum -y install systemtap systemtap-runtime kernel-devel-`uname -r`

    ./foreman-tracer > trace.log

This generates HUGE log, make sure to start it 10 seconds before the point of
interest and interrupt (Control+C) as soon as possible.
