# filter-reply-mails

This perl script was created out of the need to filter and modify mails from an IMAP folder and forward them to the JIRA issue tracker for creating new issues and comments, as JIRA does not support any sane way to deal with HTML mails (especially mails from Outlook) nor remove unnecessary signatures and their including attachments. I am sure that there are other use cases for this script too, so please let me know!

## What does it do?

The perl script connects to an IMAP server and fetches (and by default deletes) all mails from a specific IMAP folder to a temporary file system folder. A set of regexes from two different files and a file for CSS selectors, to query the DOM of HTML mails, trim all fetched mails. One regex file holds regexes for plain text parts of the mails while the other file holds regexes for HTML parts. Each line in these files defines one regex. CSS selectors are used to remove the matching elements. HTML parts can reference images therefore all referenced images of trimmed content are removed from the mail. After a mail has been trimmed it is moved to the destination file system folder.

I tried to use self-explanatory options and arguments for the script. If you need further details please execute the script with the `--help` argument or have a look at the example below.

## Installation

1. Clone the repository

	- `git clone https://github.com/zimmski/filter-reply-mails.git`

2. Install the latest versions of the following Perl modules

	- `cpan Modern::Perl Encode File::Slurp Getopt::Compact IO::File IO::Socket::SSL Mail::IMAPClient MIME::Parser Mojolicious String::Util Try::Tiny`
	- I tested with the following version:
		+ Encode 2.68
		+ File::Slurp 9999.19
		+ Getopt::Compact 0.04
		+ IO::File 1.16
		+ IO::Socket::SSL 2.012
		+ Mail::IMAPClient 3.35
		+ MIME::Parser 5.505
		+ Modern::Perl 1.20150127
		+ Mojolicious 5.77
		+ String::Util 1.24
		+ Try::Tiny 0.22
	- **Note:** Older module versions might work but I do not support them.

## JIRA example use case

1. Go to the "Incoming Mail" section in the administration and add a mail handler with the server "Local Files" and the handler "Add a comment from the non quoted email body". Test and save the handler. This handler looks for mails in the folder /import/mail of your JIRA user home folder.
2. Put the script and the two regex files somewhere where the JIRA user can execute the script.
3. Create a folder where temporary data can be stored by the script.
4. Create a cronjob for the JIRA user with the following command.

	``` bash
	perl /path/to/the/filter-reply-mails.pl --mail-server your.mail.server.domain --mail-user your-mail-user --mail-pwd your-mail-password --folder-tmp /the/tmp/folder --folder-dst /your/jira/home/folder/import/mail --filter-dom /path/to/the/filter-html.dom --filter-html /path/to/the/filter-html.regex --filter-text /path/to/the/filter-text.regex
	```

That's it! If you need a SSL or TLS IMAP connection just use the corresponding `--mail-ssl` or `--mail-tls` flag for the command. If you need to use a different port for the IMAP connection you can define it via the `--mail-server` argument e.g. `--mail-server 'your.mail.server.domain:143'`.

Common settings are:

- No encryption: `--mail-server 'your.mail.server.domain:143'`
- SSL: `--mail-server 'your.mail.server.domain:993' --mail-ssl`
- TLS: `--mail-server 'your.mail.server.domain:143' --mail-tls`

It is also common that a self-signed certificate is declined by the IMAP client. Please use the `--mail-do-not-verify-certificate` flag in that case.

## CLI arguments

```
    --do-not-remove-files              Do not remove mail files
    --filter-dom                       CSS selectors for html parts
    --filter-html                      Regexes for html parts
    --filter-text                      Regexes for text parts
    --folder-tmp                       Temporary mail folder
    --folder-dst                       Destination mail folder
    --in-memory                        Do the MIME work in memory (default is off)
    --file                             Parse MIME file and print the result to STDOUT
    --mail-do-not-connect              Do not open an IMAP connection
    --mail-do-not-delete-mail          Do not delete mails after fetching them
    --mail-do-not-verify-certificate   Do not verify the SSL certificate of the IMAP connection
    --mail-folder                      Folder from where mails will be fetched (default is INBOX)
    --mail-pwd                         Password of the IMAP user
    --mail-server                      Server connection string for the IMAP connection (default port is 993)
    --mail-ssl                         Use SSL for the IMAP connection (default is off)
    --mail-tls                         Use TLS for the IMAP connection (default is off)
    --mail-user                        User for the IMAP connection
    --verbose                          Print what is going on

-h, --help                             This help message
```

# Legal

The source for filter-reply-mails is released under the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.
