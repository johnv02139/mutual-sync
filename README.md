# mutual-sync

Sync directories without losing data or superfluous copying.

_Important note: this program does not exist!  This README describes a design which has not yet been implemented._

## Description

Take a source directory, and a destination directory, and make the destination exactly the same as the source.

This functions well as a backup.  Many backups compress and combine files in a way that they can be extracted later, but are not usable as such in the backup.  So, this program is more of a sync.  Really the motivation for this program is that you may be editing and creating the files in *both* locations.

In the base case, where the destination directory is empty, it's nothing more than a `cp -Rp`.

If the destination directory is not empty, though, it does something a little different.

One thing it does is, like [rsync](https://rsync.samba.org/), it does not copy files that don't need to be copied (because they already apparently exist in the destination).

But `rsync` and similar programs basically make `destination` be exactly like `source`; any data that was in `destination` but *not* in `source` will be overwritten or deleted.

This program does not lose any data.  If there is content in `destination` that is not in `source`, it is moved out of the way, but not destroyed.

The program also attempts to detect renames.  After matching the files that correspond in both places, it considers the ones with no apparent match.  Again using the same criteria as listed above, if it finds a file in `destination` which appears to have the same contents as one in `source`, the `destination` file will be renamed, and the `source` file will not be copied.

The program also addresses the file modification dates of the files.  The default behavior, again, is to mirror `cp -Rp` and make each corresponding file in `destination` have the same file modification time as the corresponding one in `source`.

Other options would be:
- make each file modification date in the destination be the earlier one of the source and the existing destination file
- make each file modification date in the destination be the later one of the source and the existing destination file
- make each file modification date in the destination _and the source_ be the earlier one of the source and the existing destination file

It allows you to configure where files get put "out of the way", and also how to name them.

## Match criteria

There are various things the program can look at to determine whether a file needs to be copied, or if a copy already exists in the destination.  These include:
- md5 hash of contents
- modification date
- file size
- relative path
- filename extension
- filename basename
- complete comparison of contents
- partial contents of files

The md5 hash is very reliable for detecting exact duplicates.  It is extremely unlikely in practice that two different files would have the same MD5 hash.  If you combine MD5 hash with file modification date, it becomes infinitessimally unlikely that two distinct files would coincidentally appear identical.  So, using those two factors is completly sufficient in practice.

We provide the ability to truly compare the files, byte-by-byte, for a few reasons.  When using certain network filesystems, it might actually be more efficient than trying to run the md5hash over the network.

If this is the concern, we also allow a partial comparison.  For example, if the two files are identical in size, and have identical file paths, and we compare (say) the first 10,000 bytes and find them identical, that might be more then good enough to convince us the files are identical.

But we also provide the ability to consider two files "the same" even if they are not byte-for-byte identical.  The primary case for this is text file with different styles of [newlines](https://en.wikipedia.org/wiki/Newline).  You may want to have Windows-style newlines in one location, and Un*x-style in the other location.  This program can make those comparisons and maintain the status.

It also can detect "pure additions" in text files.  This is primarily of interest for the "never lose data" feature.  If the file in `source` contains everything as the file in `destination`, plus some more, we certainly would want to copy the file from `source` to `destination`.  But do we need to preserve the old file in `destination`?  Probably not.  You have the option of having the program make these sorts of comparisons.


Nevertheless, it is theoretically possible, and it can be accomplished if done intentionally.  

Depending on the criteria being used, it is possible that the backup will not be perfect.   It could seem like the files are identical even if they're not.  But as long as the matching criteria are strong enough, you can be confident you have made an exact backup.

If you are copying from `a` to `b`, and `a/foo/bar` and `b/foo/bar` both exist, the contents of `b/foo/bar` 

It might seem pointless to do a complete comparison; you could just copy the file and not worry about it.  But in order to truly guarantee no data loss, you can use that option.

## No Data Loss

With a typical sync 

## Diffs

If there are already files in the "out of the way" place, it can consider those as well.  If 

pure additions


## Copy specification

Copy converting newlines
