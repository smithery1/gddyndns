# gddyndns

Client program to manage dynamic DNS for a Google Domains domain.

# Dynamic DNS and Google Domains

Google Domains allows you to configure dynamic DNS for a domain registered
with them. See [here](https://support.google.com/domains/answer/6147083?hl=en)
for more information.

The gddyndns script is a client program that keeps the registered IP address up
to date by periodically polling your ISP and domain IP addresses and updating
the domain one if it differs.  It follows the Google documentation at the link
above.  Google reserves the right to block updates from non-conforming API
users, and the script tries to be careful and avoid abuse.  It only calls the
API to update when the address changes and prevents updates for a time when an
error occurs.

# Install

First set up Dynamic DNS synthetic record for your Google Domains domain as
described in the link above.

Then clone the git repo.

`git clone https://github.com/smithery1/gddyndns`

Copy the sample.config file to a `.gddyndns` in the home directory of the user
that will run the script.  Do not run as root.  Modify the contents to match
your settings.

All the parameters in the file can also be specified on the command line, and
will override settings in the config file.  See `gddyndns help` for more
information.

Run the script to test that it worked.  If it returns an error, fix the
configuration and try again.

`gddyndns update`

Finally, add the script to cron or another scheduler to run periodically. Here
is a sample cron line.

`0,15,30,45 * * * * /home/asmith/gddyndns/gddyndns update`

When the script's stdout and stderr are not a terminal, it will log via syslog
with its name and a daemon priority.

# Usage

## Update

Running `gddyndns update` will update your dynamic DNS IP address if
necessary.  It performs the following steps.

1. If a previous run failed and we are still within the lockout time, log a
message and exit.

2. Obtain the current ISP assigned IP address from
`https://domains.google.com/checkip`

3. Obtain the current domain IP address using `dig`.

4. If the two address match, then exit.

5. Otherwise attempt to update the dynamic DNS address.

6. If the update fails, record a lockout time to prevent further attempts
for a period.  If the failure code is `911` this time is five minutes.
Otherwise it is the value of the `LOCKOUT_TIME` setting (see `sample.config`).

## Status

Running `gddyndns status` will output the status of the most recent update
attempt and the current values of the IP addresses.
