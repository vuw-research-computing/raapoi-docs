### Scratch Tips

The scratch storage is provided by a dedicated ZFS storage system and is available to all Raapoi users at ```/nfs/scratch/<username>```.

Scratch storage is intended for actively used research data and temporary working files. It is not backed up, so important data should not be stored exclusively on scratch storage. While the storage system includes RAIDZ2 disk redundancy and redundant power supplies, it remains a single storage system and cannot protect against all hardware failures.

Each user is allocated a default quota of 5TB of scratch space. You can check your quota and current usage by running vuw-quota. If your research requires additional storage, please contact [the support team](../support.md).

Scratch storage is a shared resource. Although users are allocated generous quotas, the total storage available is shared across all users. Researchers are encouraged to regularly remove data that is no longer required and archive completed work elsewhere. When storage usage becomes high, the Research Computing team may contact users with large allocations and ask them to review or clean up their data.

To check how much space is free on the scratch storage for all users, on Rāpoi: 
```
df -h | grep scratch  #df -h is disk free with human units, | pipes the output to grep, which shows lines which contain the word scratch
```

This storage is **not backed up** at all.  It is on a raid array so if a hard drive fails your data is safe.  However in the event of a more dramatic hardware failure, earthquakes or fire - your data is gone forever.  If you accidentally delete something, it's gone forever. If an Admin misconfigures something, your data is gone (we try not to do this!).

It is **your responsiblilty to backup your data** here - a good place is to use Digital Solutions Research Storage (see [Connecting to SoLAR](external/solar_vuw.md)).

Scratch is also not a place for your old data to live forever, please clean up datasets you're no longer using!
