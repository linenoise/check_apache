NAME
    check_apache - Nagios plugin to poll Apache mod_status information

DESCRIPTION
    This script acts as a plugin module for the Nagios IT infrastructure
    monitoring system. It polls an Apache HTTPD server for status
    information through mod_status, parses for the requested value, compares
    that value against warning and critical levels, and responds with a
    status string and appropriate exit status.

    This script handles basic authentication (.htaccess / .htpasswd), SSL
    encryption, and redirection to extract any of the following values:

    *   active_threads
    *   bytes_per_request (measured in bytes)
    *   bytes_per_second (measured in bytes per second)
    *   cpu_load (measured in percent)
    *   cpu_usage_system (measured in percent)
    *   cpu_usage_system_children (measured in percent)
    *   cpu_usage_total (measured in percent)
    *   cpu_usage_user (measured in percent)
    *   cpu_usage_user_children (measured in percent)
    *   idle_threads
    *   parent_server_generation
    *   requests_per_second
    *   server_uptime (does not require thresholds)
    *   server_version (does not require thresholds)
    *   slots_open
    *   slots_total
    *   ssl_average_session_time_remaining (measured in seconds)
    *   ssl_cache_remove_hits (does not require thresholds)
    *   ssl_cache_remove_misses (does not require thresholds)
    *   ssl_cache_retrieve_hits (does not require thresholds)
    *   ssl_cache_retrieve_misses (does not require thresholds)
    *   ssl_cache_type (does not require thresholds)
    *   ssl_cache_usage (measured in percent)
    *   ssl_current_sessions (does not require thresholds)
    *   ssl_expired_sessions (does not require thresholds)
    *   ssl_highest_session_time_remaining (measured in seconds)
    *   ssl_index_usage (measured in percent)
    *   ssl_indexes_per_subcache (does not require thresholds)
    *   ssl_lowest_session_time_remaining (measured in seconds)
    *   ssl_shared_memory (measured in bytes, does not require thresholds)
    *   ssl_subcaches (does not require thresholds)
    *   ssl_total_sessions (does not require thresholds)
    *   ssl_uncached_unexpired_sessions (does not require thresholds)
    *   threads_closing_connection
    *   threads_dns_lookup
    *   threads_gracefully_finishing
    *   threads_idly_cleaning
    *   threads_keepalive
    *   threads_logging
    *   threads_reading_request
    *   threads_sending_reply
    *   threads_starting_up
    *   threads_waiting_for_connection
    *   total_accesses (does not require thresholds)
    *   total_traffic (measured in bytes, does not require thresholds)

        Note: mod_status calculates requests_per_second and bytes_per_second
        as an average of requests and bytes over uptime. These do not
        represent a rolling average.

        Note: mod_status calculates the following SSL-related values as of
        the most recent server restart: ssl_total_sessions,
        ssl_expired_sessions, ssl_uncached_unexpired_sessions,
        ssl_cache_retrieve_hits, ssl_cache_retrieve_misses,
        ssl_cache_remove_hits, ssl_cache_remove_misses.

    This has been tested with Apache/2.2.10 (Unix) and should work with any
    HTTPD 2.x running mod_status. Most Apache HTTPD builds come with
    mod_status pre- compiled: you usually don't have to load it manually.
    One way to configure mod_status on an HTTPD server is to include the
    following in your httpd.conf:

            ExtendedStatus On
            <Location /server-status>
                    SetHandler server-status
                    Order deny,allow
                    Deny from all
                    Allow from 127.0.0.1
                    Allow from 192.168.1.10
            </Location>

    You will ideally want to encrypt this through SSL, protect it behind an
    .htaccess file, or both. At very minimum, it would be good practice to
    white- list any IPs which will need access to status information with
    'Allow from' directives rather than Allowing from all and relying on a
    blacklist.

SYNOPSIS
  Command Line Interface
    Poll status on a server with no encryption or authentication:

            check_apache -H your_host.com -w 160 -c 200 \
            -m active_threads

    Poll status on a server with encryption but no authentication:

            check_apache -S -P 443 -H your_host.com -w 160 \
            -c 200 -m active_threads

    Poll status on a server with encryption and authentication:

            check_apache -S -P 443 -H your_host.com -w 160 \
            -c 200 -m active_threads -u username -p password

    Poll a server with command line options stored in a configuration file:

            check_apache --extra-opts=my_config.ini

  Running within Nagios
    In objects/commands.cfg:

            define command{
            command_name    check_apache
            command_line    $USER1$/check_apache -H $HOSTNAME$ -m \
                                                    $ARG1$ -c $ARG2$ -w $ARG3$
            }
            
        define command{
            command_name    check_apache_ssl
            command_line    $USER1$/check_apache -H $HOSTNAME$ -m \
                                                    $ARG1$ -c $ARG2$ -w $ARG3$ -P 443 -S
            }

            define command{
            command_name    check_apache_ssl_auth
            command_line    $USER1$/check_apache -H $HOSTNAME$ -m \
                                                    $ARG1$ -c $ARG2$ -w $ARG3$ -P 443 -S \
                                                    -u $ARG4$ -p $ARG5$
            }

            define command{
            command_name    check_apache_auth
            command_line    $USER1$/check_apache -H $HOSTNAME$ -m \
                                                    $ARG1$ -c $ARG2$ -w $ARG3$ -u $ARG4$ \
                                                    -p $ARG5$
            }

    In the configuration file for your host with apache running on it:

            define service{
            use                   local-service
            host_name             your.hostname.com
            service_description   Apache Active Threads
            check_command         check_apache!active_threads200!300
            }

            define service{
                    use                   local-service
                    host_name             your.encrypted.hostname.com
                    service_description   Apache Active Threads
                    check_command         check_apache_ssl!active_threads200!300
            }

            define service{
            use                   local-service
            host_name             your.cool.hostname.com
            service_description   Apache Active Threads
            check_command         check_apache_ssl_auth!active_threads200!300!un!pw
            }

            define service{
                    use                   local-service
                    host_name             your.passworded.hostname.com
                    service_description   Apache Active Threads
            check_command         check_apache_auth!active_threads200!300!un!pw
            }

FREQUENTLY ASKED QUESTION
    Q: I've checked all of my command line arguments thrice, but am still
    getting "APACHE UNKNOWN - Host returned HTTP code 500." on an otherwise
    working server. I've even run it with -v and verified that the URI it
    generates if correct: I can copy and paste it into Firefox and
    everything and it works there! What gives?

    A: If you've using SSL, you need to install Crypt::SSLeay. Also, if
    you're not polling port 80, don't forget to specify your port with the
    -P option.

SEE ALSO
    If using an external configuration file, it should be structured
    according to the specification at
    <http://nagiosplugins.org/extra-opts/>.

    If you intend on processing requests over SSL, you will need the
    Crypt::SSLeay module installed.

    Thresholds given to this script should be in the format specified at
    <http://nagiosplug.sourceforge.net/developer-guidelines.html>.

    This module is built upon Nagios::Plugin by the Nagios Plugin
    Development Team. Further reading on Nagios, NPRE, and Nagios Plugins is
    available at <http://nagios.com/>.

AUTHOR
    This script is written and maintained by Dann Stayskal
    <dann@stayskal.com> and is available on his website, at
    <http://fragmentedzen.com/software/check_apache/>.

LICENSE
    Copyright (C) 2009 by Dann Stayskal.

    This program is free software; you can redistribute it and/or modify it
    under the terms of the GNU General Public License as published by the
    Free Software Foundation; either version 2 of the License, or (at your
    option) any later version.

    This program is distributed in the hope that it will be useful, but
    WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General
    Public License for more details.

    You should have received a copy of the GNU General Public License along
    with this program; if not, write to the Free Software Foundation, Inc.,
    59 Temple Place, Suite 330, Boston, MA 02111-1307 USA

    Nagios, the Nagios logo, and Nagios graphics are the servicemarks,
    trademarks, or registered trademarks owned by Nagios Enterprises. All
    other servicemarks and trademarks are the property of their respective
    owner.

