### BeeGFS Tips

The BeeGFS storage is spread across 3 nodes with SSD disks.  The aggregate storage is 100TB.  We don't enforce quotas here as some projects have large storage needs.  However, you do need to be a good HPC citizen and respect the rights of others.  Don't needlessly fill up this storage.

We regularly **delete all data** here every 3-6 months.  We only post warnings on the [slack channel](uwrc.slack.com/).  If afteer 3 months the storage is not full and still performning well, we delay the wipe another 3 months.

This storage is **not backed up** at all.  It is on a raid array so if a hard drive fails your data is safe.  However in the event of a more dramatic hardware failure, earthquakes or fire - your data is gone forever.  If you accidentally delete something, it's gone forever. If an Admin misconfigures something, your data is gone (we try not to do this!).

It is **your responsiblilty to backup your data** here - a good place is to use Digital Solutions Research Storage.

BeeGFS should have better IO performance than the scratch storage - however it does depend what other users are doing.  No filesystem likes small files, BeeGFS likes small files even less than scratch.  Filesizes over 1MB are best.   If you have a large amount of files you can improve performance by splitting your files accross many directories - the load balancing accross the metadata and storage servers is by directory, not file.