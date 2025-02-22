## import

Read new transactions added to each FILE provided as arguments since
last run, and add them to the journal.
Or with --dry-run, just print the transactions that would be added.
Or with --catchup, just mark all of the FILEs' current transactions 
as imported, without importing them.

_FLAGS

This command may append new transactions to the main journal file (which should be in journal format).
Existing transactions are not changed.
This is one of the few hledger commands that writes to the journal file (see also `add`).

Unlike other hledger commands, with `import` the journal file is an output file,
and will be modified, though only by appending (existing data will not be changed).
The input files are specified as arguments, so to import one or more
CSV files to your main journal, you will run `hledger import bank.csv`
or perhaps `hledger import *.csv`.

Note you can import from any file format, though CSV files are the
most common import source, and these docs focus on that case.

### Deduplication

`import` does *time-based deduplication*, to detect only the new
transactions since the last successful import.
(This does not mean "ignore transactions that look the same",
but rather "ignore transactions that have been seen before".)
This is intended for when you are periodically importing downloaded data,
which may overlap with previous downloads.
Eg if every week (or every day) you download a bank's last three months of CSV data,
you can safely run `hledger import thebank.csv` each time and only new transactions will be imported.

Since the items being read (CSV records, eg) often do not come with
unique identifiers, hledger detects new transactions by date, assuming
that:

1. new items always have the newest dates
2. item dates do not change across reads
3. and items with the same date remain in the same relative order across reads.

These are often true of CSV files representing transactions, or true
enough so that it works pretty well in practice. 1 is important, but
violations of 2 and 3 amongst the old transactions won't matter (and
if you import often, the new transactions will be few, so less likely
to be the ones affected).

hledger remembers the latest date processed in each input file by
saving a hidden ".latest.FILE" file in FILE's directory
(after a succesful import).

Eg when reading `finance/bank.csv`, it will look for and update the
`finance/.latest.bank.csv` state file.
The format is simple: one or more lines containing the
same ISO-format date (YYYY-MM-DD), meaning "I have processed
transactions up to this date, and this many of them on that date."
Normally you won't see or manipulate these state files yourself.
But if needed, you can delete them to reset the state (making all
transactions "new"), or you can construct them to "catch up" to a
certain date. 

Note deduplication (and updating of state files) can also be done by
[`print --new`](#print), but this is less often used.

Related: [CSV > Working with CSV > Deduplicating, importing](#deduplicating-importing).


### Import testing

With `--dry-run`, the transactions that will be imported are printed
to the terminal, without updating your journal or state files.
The output is valid journal format, like the print command, so you can re-parse it.
Eg, to see any importable transactions which CSV rules have not categorised:

```shell
$ hledger import --dry bank.csv | hledger -f- -I print unknown
```

or (live updating):

```shell
$ ls bank.csv* | entr bash -c 'echo ====; hledger import --dry bank.csv | hledger -f- -I print unknown'
```

Note: when importing from multiple files at once, it's currently possible for
some .latest files to be updated successfully, while the actual import fails
because of a problem in one of the files, leaving them out of sync (and causing
some transactions to be missed).
To prevent this, do a --dry-run first and fix any problems before the real import.

### Importing balance assignments

Entries added by import will have their posting amounts made explicit (like `hledger print -x`).
This means that any [balance assignments](https://hledger.org/hledger.html#balance-assignments) in imported files must be evaluated;
but, imported files don't get to see the main file's account balances.
As a result, importing entries with balance assignments
(eg from an institution that provides only balances and not posting amounts)
will probably generate incorrect posting amounts.
To avoid this problem, use print instead of import:

```shell
$ hledger print IMPORTFILE [--new] >> $LEDGER_FILE
```

(If you think import should leave amounts implicit like print does,
please test it and send a pull request.)

### Commodity display styles

Imported amounts will be formatted according to the canonical [commodity styles](hledger.html#commodity-display-style)
(declared or inferred) in the main journal file.
