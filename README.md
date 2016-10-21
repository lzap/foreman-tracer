# Foreman SystemTap tracing utility

Usage:

    yum -y install systemtap systemtap-runtime kernel-devel-`uname -r`

    ./foreman-tracer > trace.log

This generates HUGE log, make sure to start it 10 seconds before the point of
interest and interrupt (Control+C) as soon as possible.
