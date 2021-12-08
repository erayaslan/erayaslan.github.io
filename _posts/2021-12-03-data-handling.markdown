__Batch vs Streaming__

Stream good batch bad.  That is the prevailing thought nowadays.  And in a sense yes that is a correct sentiment.  However, processing streaming data is not inherently good and batch processing is not inherently bad.  There are some underlying principles to how data should be handled.  Streaming makes it easier to implement those principles and that is why it is good.  And conversely, batch processing makes it harder to implement those principles and that is why it is bad.  And one of those data handling principles is:

__Immutability__

Data should be immutable.  There is a reason functional programming and scala with its immutable variables is used in a distributed system such as spark.  It makes concurrency and therefore scalability easier to implement.  It is easier to reason about - what you see is what you get. It is easier to test - you worry about the logic and not the state. And it is thread safe, i.e. easier to parallelize.  Atomic operations become very difficult to implement correctly and efficiently when you have hundreds and thousands of machines/processes/threads to coordinate.  Best solution is to design around the problem: just get rid of the need for coordination altogether by making data immutable.

Immutable data is more common than is generally assumed.  Take good old sql databases for example.  They are mutable - or are they?

Let's say we prepare an sql statement `UPDATE xxx SET price=50 WHERE zzz`.  What happens when we send this query to an sql engine is that the changes are first written to a log before changing the data files.  And if there is a crash during the operation, any changes that have not been applied to the data pages can be redone from the log records (and this is called roll-forward recovery btw).  In other words, the transaction log is our source of truth and the data files (tables, indexes etc) are just a snapshot of this log at some time `t`.  And this is why point-in-time recovery works as well.  We just move back and forth along the transaction log.

Now, I send that query, someone else sends some other query and life goes on.  But please realise that basically we are streaming data into our database.  And the database uses an immutable append-only data structure which acts as an event store to mimic changing data.  Internal source of truth is immutable but the presentation layer mimics mutability.  That is how our big data systems should behave as well.

Almost all modern file systems work in a similar way.  It is called journal in their case instead of log but the principle is the same.  Log the changes before making them available for consumption. 

Another example is the pub/sub systems.  Take Kafka for example. Kafka writes its data to an immutable append only partition. Should sound familiar by now.

Parquet files are immutable as well and this is one of the reasons they are popular in a distributed environment: You can process them without running into some of the concurrency problems.  Don't get me wrong you still run into concurrency problems with parquet, especially when you partition your data and your logical parquet file consists of many physical files but at least you get rid of one type concurrency problem.

And streaming is good because the data in streaming is by necessity via immutable append only data structures and event stores.  Hence, it is easier to make the correct choices. The architecture pushes you in the correct direction.  This is doable in a batch processing system as well but requires much discipline and conscious effort.

Also, there is a misconception that streaming data should be used for frequently changing data like clickstream.  But that isn't true at all.  Whether data changes once every second or once every 24 hours does not matter.  Use the same architecture for both streams and keep it simple and clean.

__Reproducibility__: Now this is one of the end goals.  Your data pipeline should be reproducible and, more importantly, your machine learning models and experiments should be reproducible - there are no ifs and buts about this.  And reproducible ML models require reproducible data.  You do not want your underlying data to change while developing, testing or training your ML model.  Fixing your data by copying it around is just not feasible or maintainable with big enough data so having immutable, snapshotable, i.e. time-travel-possible data is a must. Same data source with the same computational logic processed at different time should produce the same output.

__Scalability__: That is another end goal. Designing data pipelines that run the same computational logic across multiple nodes and reproduce the same predictable result every time is difficult. And immutable data helps us achieve this end goal as well.

Achieving the above end goals is (realistically only?) possible via immutable data.

__Moral of the story__: Use immutable data.  Use streaming for ease of implementation. Be wary of batch processing and say no to [lambda architecture][lambda-url]. There should be no reason to use them - not anymore.

Another principle for data handling is idempotence - no side effects, similar to pure functions in functional programming.  But that is a topic for another post. This one even now is longer than I expected.

[lambda-url]: https://en.wikipedia.org/wiki/Lambda_architecture