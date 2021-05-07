dupefinder.py
=============

This is a rewrite of dupefinder.go in Python.
This tool calculates file hashes to find identical files.
These are displayed in (duplicate) groups.



Replacing duplicate files
-------------------------

With the --link-duplicates option, duplicates are automatically
replaced with hardlinks so they won't take up additional space anymore.
When this option is used, each duplicate is automatically
removed from the filesystem and replaced with a hardlink to the original file.
This is obviously only recommended for directory structures which
contain non-volatile content, in other words you must be sure that
none of the duplicates will ever have to be changed.



Map files
---------

A map file contains the map generated by DupeFinder, which contains
metadata for the scanned directory.
It is possible to export a map file, which can be imported later to speed up
another scan of the same directory.
After importing a map file, files that have not changed
(same name, size, mtime) are not hashed again.
This makes it possible to limit a subsequent scan to the changes
(new or modified files) rather than having to scan everything again
which may take hours to finish.

To speed up the process when scanning a directory repeatedly,
a map file should be exported using `--export-map-file files.map`.
This map should then be imported when the directory is scanned again,
so that files that haven't changed since last time aren't hashed again.
This will considerably speed up the process and save time,
especially if this tool is run regularly (as a cron job).



Example
-------

```
$ ./dupefinder.py test/; echo rc=$?
[26bb73556ceb32a5df30b733c5355ee5]
* test/foof/dir1/f1
- test/foof/dir2/filetwo
- test/foof/dir2/filetwotwo

Files (scanned):	14
Total size:		    7340098 B
Duplicate groups:	1
Total wasted space:	30 B
Replaced files:		0
rc=2
```

Return code 2 indicates that it has found duplicates.
If duplicates are removed/replaced, the return code will be 4.

Scan directory and export map file for the next run:
```
# time dupefinder.py --export-map-file /tmp/files.map .
...
Files:                  22719
Total size:             4221404983496 B
Duplicate groups:       6848
Total wasted space:     2378973067188 B
Replaced files:         0

real    1088m56.207s
user    216m4.000s
sys     36m41.507s
```

Second run, import map file:
```
# time dupefinder.py --import-map-file /tmp/files.map --export-map-file /tmp/files.map .
...
Files:                  22837
Total size:         4210329975535 B
Duplicate groups:       6860
Total wasted space:     2364639100866 B
Replaced files:         0

real    4m25.790s
user    1m40.933s
sys     0m19.408s
```

In this example, the first run took more than half a day and the second run
took only a couple of minutes because only new files were hashed.



Author
------

Philip Seeger (philip@philip-seeger.de)



License
-------

Please see the file called LICENSE.



