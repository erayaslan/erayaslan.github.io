__Batch vs Streaming__

Stream good batch bad.  That is the prevailing thought nowadays.  And in a sense yes that is a correct sentiment.  However, streaming data in not inherintly good and batch processing is not inherintly bad.  There are some underlying principles to how data should be handled.  Streaming makes it easier to implement those principles and that is why it is good.  And conversely, batch processing make it harder to implement those principles and that is why it is bad.  And one of those principles is:

__Immutability__

Data should be immutable.  There is a reason functional programming and scala with its immutable variables is used in a distributed system such as spark.  It makes concurrency and therefore scalibility easier to implement.  Atomic operations become very difficult to implement correctly and efficiently when you have hundreds and thousands of machines to coordinate.  Best solution is to design around the problem: just get rid of the need for coordination altogether by making data immutable.

Immutable data is more common than is generally assumed.  Take good old sql databases for example.  They are mutable - or are they?  Let's say we prepare an sql statement `UPDATE xxx SET price=50 WHERE zzz`.  What happens when we send this query to an sql engine is the changes are first written to a log before making changes in data files.  And if there is a crash during the operation, any changes that have not been applied to the data pages can be redone from the log records (and this is called roll-forward recovery).  In other words, the transaction log is our source of truth and the data files (tables, indexes etc) are just a snapshot of this log at some time `t`.  And this is why point-in-time recovery works as well.  We just move back and forth along the transaction log.

Now, I send that query, someone else sends some other query and life goes on.  But please realize that basically we are streaming data into our database.  And the databse uses an immutable (append-only) data structure to mimic changing data.  Internal source of truth is immutable but presentation layer mimics mutability.  That is how our big data systems should work as well.

Almost all modern file systems behave in a similar way.  It is called journal in their case instead of log but the principle is the same.  Log the changes before making them available - filesystems usually log only metadata for the change btw but the principle is the same. 

Another example are the pub/sub systems.  Take Kafka for example. Kafka writes its data to an immutable, append only partition. Should sound familiar by now.

Parquet files are immutable as well and this is one of the reasons they are populer in a distributed environment: You can process them without running into some of the concurrency problems.  Dont get me wrong you still run into concurrency problems, especially when you partition your data and your logical parquet file consists of many physical files but at least you get rid of one type concurrency problem.

Reproducibility: Now this is one of the end goals.  Your ML models and expriments should be reproducible - there are no ifs and buts about this.  Damn this is going to take too long

