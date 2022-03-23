### Scratch Tips

The scratch storage is on a large storage node with 2 raid arrays with 50TB of storage each.  Your scratch will always be available at ```/nfs/scratch/<username>```.

Your scratch storage could be on scratch or scratch2, to find out run ```vuw-quota```.

Each user has a quota of 5TB on scratch - you can ask [us](../support.md) to increase it if needed.  While each user has a quota of 5TB, we don't actually have enough storage for each user to fill 5TB of storage!  This is a shared resource and we will occationally ask on the [slack channel](uwrc.slack.com/) for users to clean up their storage to make space for others.

To check how much space is free on the scratch storage for all users, on Rāpoi: 
```
df -h | grep scratch  #df -h is disk free with human units, | pipes the output to grep, which shows lines which contain the word scratch
```
On the [slack channel](uwrc.slack.com/) in any DM or channel type
```
/df-scratch 
```
The output should only be visible to you


This storage is **not backed up** at all.  It is on a raid array so if a hard drive fails your data is safe.  However in the event of a more dramatic hardware failure, earthquakes or fire - your data is gone forever.  If you accidentally delete something, it's gone forever. If an Admin misconfigures something, your data is gone (we try not to do this!).

It is **your responsiblilty to backup your data** here - a good place is to use Digital Solutions Research Storage.

Scratch is also not a place for your old data to live forever, please clean up datasets you're no longer using!