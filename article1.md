---
layout: post
title: Comments on "*Coding for SSDs*"
---

{{ page.title }}
================

Emmanuel Goossaert posted a series of ["*Coding for SSDs*"](http://codecapsule.com/2014/02/12/coding-for-ssds-part-1-introduction-and-table-of-contents/) based on his research for a key-value store project.
In Part 6, he recommends some access patterns for SSDs.

Here we will go through his recommendations (15-30) one by one.
####15. Never write less than a (NAND) page
This is not necessary.
Inside SSDs, data will be buffered and aggregated to fill up NAND pages.
So a small write will not occupy the whole NAND page.
But, sequential writes outperform random writes significantly because random writes require much more FTL updates.
####16. Align writes (on NAND page size)
Pointless.
First, host data may not be written as is to NAND flash.
Extra data like CRC may be inserted (think of DIF (Data Integrity Field)); data may also be compressed.
Second, actual NAND page size is not aligned to KB boundary at all.
It includes data and spare, which is used for ECC (Error Correction Codes).
Nominal data size recommended by flash vendor is actually aligned to KB but an SSD controller can choose to store fewer data and use more spare for better protection, or vice versa for more space.
####17. Bufferize small writes
Isn't this just a rephrased Item 15?
####18. To improve the read performance, write related data together
This is true.
####19. Separate read and write requests
Generally it is incorrect, but I can imagine it is true for some SSDs.
For a well designed SSD, the IOPS should look like this: 100% read > 50%/50% > 100% write.
If you separate reads and writes, you lose the read/write parallism.
For example, if all requests are writes, the read data path will be idle.
It is true that there may be conflicts/competitions between read and write requests, but it is also true between requests of the same type.
####20. Invalidate obsolete data in batch
No, it will hurt.
If you hold up the TRIMs, the usable space is lowered unnecessarily.
Low usable space will put huge pressure on garbage collection.
It may have to move your obsolete data around to recover empty blocks.
That leads to higher write amplification.
####21. Random writes are not always slower than sequential writes
No, unless your random writes are not random at all.
If your write size is 16 MB each, they are simply not qualified as random.
Each of them is a bunch of sequential writes.
And no, it is not because that is aligned to clustered block size.
The real reason is it is big enough so it requires roughly the same number of FTL updates as sequential writes.
####22. A large single-thread read is better than many small concurrent reads
Misleading.
Sequential reads are better than random reads, period.
####23. A large single-threaded write is better than many small concurrent writes
The same as Item 22.
####24. When the writes are small and cannot be grouped or buffered, multi-threading is beneficial
Yes, it is the purpose of IO queues.
####25. Split cold and hot data
Yes, it will help.
####26. Bufferize hot data
Yes, it is true for any storage.
####27. PCI Express and SAS are faster than SATA
This is true.
But keep in mind a fast interface does not necessarily mean a fast drive.
####28. Over-provisioning is useful for wear leveling and performance
Yes, higher is better.
The best way to over-provision? Keep fewer data on the drive.
See Item 20.
####29. Enable the TRIM command
Yes, see Item 20.
####30. Align the partition (to NAND page size)
Yes, but to 4 KB boundary instead of NAND page size.
It is for FTL to avoid unaligned IOs.
An unaligned write will even become two read-modify-writes.
On the other hand, I am curious why file systems on unaligned partitions don't help to minimize unaligned IOs.

