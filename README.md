# Foreman SystemTap tracing utility

## Requirements

* Foreman or Satellite
* RHEL 7 (6.x will not work)

## Usage

    yum -y install systemtap systemtap-runtime kernel-devel-`uname -r`

Start tracing of the foreman Ruby on Rails (Passenger) application:

    foreman-tracer

This is same as:

    foreman-tracer rails

Note this generates a lot of lines and it can overload the server when
connected via SSH, it's recommended to redirect output to a file:

    foreman-tracer > trace.log

It is good idea to start it 10 seconds before the point of interest and
interrupt (Control+C) as soon as possible to minimize the file size. The log
can be compressed with bzip2/xz with high compression ratio, consider that
before sending it to Foreman or Satellite support engineers.

### Tracing foreman-proxy

To trace Foreman Proxy (also known as Smart Proxy or Capsule) do this:

    foreman-tracer proxy > trace.log

### Tracing cron

It is possible to trace arbitrary Ruby applications as well (e.g. rake tasks
created from cron). In that case, provide PID of the process:

    foreman-tracer 13513 > trace.log

Note that some classes are filtered out for visibility (e.g. Logger etc). Also
only sources from /usr/share/foreman* are taken into account (all the rest
like rubygems and other dependencies are ignored from all tracing).

### Top method calls

To see top method calls "live" do:

    foreman-tracer rails calls

This works with proxy option or arbitrary Ruby PID as well. Screen refreshes
every second and all call counters reset every five minutes. Use Ctrl+C to
stop the live view.

Keep in mind that all "live" probes do only show counters from files in
/usr/share/foreman* therefore when going after a memory leak or performance
issue, it can be in Foreman dependency which is not shown here. It is possible
to easily modify the script to ignore the prefix.

### Top object allocations

    foreman-tracer rails objects-total

### Top object allocations per line

    foreman-tracer rails objects

### Top array or hash allocations per line

    foreman-tracer rails arrays

This probe does not filter irrelevant classes like Logger.

### Top string allocations per line

    foreman-tracer rails strings

This probe does not filter irrelevant classes like Logger.

## Exceptions

All exceptions are traced with `EXCEPTION` prefix so they can be easily
searched. Ruby source filepath and line is also added to the output.

## Example sessions

The following output demonstrates usage of Foreman Proxy 1.13 handling request
to list it's features (GET `/features`).

    # foreman-tracer proxy
    Tracing started at Tue Oct 25 08:52:49 EDT 2016
    Proxy::Plugins.instance
    Proxy::Plugins.select
    Proxy::Plugins.loaded
    JSON::Ext::Generator::GeneratorMethods::Array.to_json
    JSON::Ext::Generator::State.initialize_copy
    Struct.initialize
    ^CTracing ended at Tue Oct 25 08:54:23 EDT 2016

The following example shows "top" utility tracing Foreman Rails process after
few clicks in the WebUI (hosts list, host details, dashboard).

    # foreman-tracer rails calls
                                                                   FILENAME   LINE                    METHOD  CALLS
         /usr/share/foreman/app/services/config_report_status_calculator.rb     23                    status    780
         /usr/share/foreman/app/services/config_report_status_calculator.rb     34                 status_of    780
         /usr/share/foreman/app/services/config_report_status_calculator.rb     11                 calculate    764
         /usr/share/foreman/app/services/config_report_status_calculator.rb      5                initialize    764
                             /usr/share/foreman/app/models/config_report.rb     87                calculator    744
                             /usr/share/foreman/app/models/config_report.rb     84                 status_of    744
                             /usr/share/foreman/app/models/config_report.rb     31      report_status_column    744
                       /usr/share/foreman/app/helpers/application_helper.rb      6                   link_to    128
                  /usr/share/foreman/app/models/concerns/parameterizable.rb     43                  to_param    114
                                      /usr/share/foreman/app/models/user.rb    137                    admin?    113
                                   /usr/share/foreman/app/models/setting.rb    218                     cache    108
                                   /usr/share/foreman/app/models/setting.rb     72                        []    105
                       /usr/share/foreman/app/helpers/application_helper.rb     92            authorized_for     72
                                      /usr/share/foreman/app/models/user.rb    319               allowed_to?     58
                               /usr/share/foreman/app/services/menu/item.rb     29                  url_hash     40

The following example shows "top" mode for Foreman Rails app when bare-metal
host was created and then deleted:

    # foreman-tracer rails objects-total
                                            OBJECT    COUNT
                      ConfigReportStatusCalculator     2101
                                               ERB       97
                                     Safemode::Box       97
                          ActionView::OutputBuffer       53
                           ActionDispatch::Request       24
                                        Authorizer       23
          ActiveSupport::HashWithIndifferentAccess       23
                                        SafeRender       12
                                            IPAddr       11
                                OpenSSL::PKey::RSA       10
                        OpenSSL::X509::Certificate       10
                                HostStatus::Global        8
                       Classification::GlobalParam        8
                               Orchestration::Task        7
                                     ProxyAPI::DNS        7

## Cleanup

When SystemTap is no longer necessary, it is good idea to remove it as it
installs GNU C/C++ compiler which can be considered as a security risk:

    yum remove systemtap* kernel-devel-`uname -r` gcc cpp

## Troubleshooting

Ruby SCL is hardcoded in the script, some older version of Foreman (or
Satellite) have different Foreman/Ruby SCL. This can be determined with:

# grep Ruby /etc/httpd/conf.d/05-foreman.conf
PassengerRuby /usr/bin/tfm-ruby

## TODO

* Strip "/usr/share/foreman" from FILENAME column to make it narrower
* Automatically detect Ruby SCL

Have fun!
