## help

Show the hledger user manual in the terminal, with `info`, `man`, or a pager.
With a TOPIC argument, open it at that topic if possible.
TOPIC can be any heading in the manual, or a heading prefix, case insensitive.
Eg: `commands`, `print`, `forecast`, `journal`, `amount`, `"auto postings"`.

_FLAGS

This command shows the hledger manual built in to your hledger version.
It can be useful when offline, or when you prefer the terminal to a web browser,
or when the appropriate hledger manual or viewing tools are not installed on your system.

By default it chooses the best viewer found in $PATH, trying (in this order):
`info`, `man`, `$PAGER`, `less`, `more`.
You can force the use of info, man, or a pager with the `-i`, `-m`, or `-p` flags,
If no viewer can be found, or the command is run non-interactively, it just prints
the manual to stdout.

If using `info`, note that version 6 or greater is needed for TOPIC lookup. If you are
on mac you will likely have info 4.8, and should consider installing a newer version,
eg with `brew install texinfo` (#1770).

Examples
```shell
$ hledger help --help      # show how the help command works
$ hledger help             # show the hledger manual with info, man or $PAGER
$ hledger help journal     # show the journal topic in the hledger manual
$ hledger help -m journal  # show it with man, even if info is installed
```
