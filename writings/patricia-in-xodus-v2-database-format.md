# Xodus Version 2 Database Format and Patricia Tree Improvements

It's been over a year since [Xodus](https://github.com/JetBrains/xodus) version 1.3.232
was released. A lot of new things are done, and the next release of Xodus will be 2.0.0.
It will bring the new database format which would be possible to forcibly turn
off by an `EnvironmentConfig` setting (`exodus.useVersion1Format`).

For existing databases and applications using
[Environments API](https://github.com/JetBrains/xodus/wiki/Environments), no migration
would require. Database format would be changed automatically in background: new and
modified data as well as the data moved by database GC would be written in the new format.

Among two types of search tree (BTree & Patricia) only Patricia can be saved in the new
format. Let me remind you that Patricia is a search tree for [stores](https://github.com/JetBrains/xodus/wiki/Environments#stores)
created with the `StoreConfig.WITHOUT_DUPLICATES_WITH_PREFIXING` or
`StoreConfig.WITH_DUPLICATES_WITH_PREFIXING` configs (BTW, I encourage everyone to use
those instead of `StoreConfig.WITHOUT_DUPLICATES` and `StoreConfig.WITH_DUPLICATES`).

I wrote a benchmark that reads and writes the first 10 million non-negative integers in a
random order using both database formats. Writing is done in several transactions
each committing 10 thousand integers. Each method is measured in seconds, here are the results:

```text
Benchmark                                              Mode  Cnt  Score   Error  Units
JMHEnvWithPrefixingRandomAccessReadBenchmarkV1.read      ss    5  5.493 ± 1.929   s/op
JMHEnvWithPrefixingRandomAccessReadBenchmarkV2.read      ss    5  3.455 ± 1.981   s/op
JMHEnvWithPrefixingRandomAccessWriteBenchmarkV1.write    ss    5  6.385 ± 1.755   s/op
JMHEnvWithPrefixingRandomAccessWriteBenchmarkV2.write    ss    5  5.328 ± 2.141   s/op
```

Writes and especially reads are faster when using the new database format. Usage of
disk space is also less.

Results could be different if I used some other input, not the first 10 million non-negative
integers. However, this is the case actual for [YouTrack](https://www.jetbrains.com/youtrack)
databases which we improve first. Also, the case looks similar to use of some kind of
autoincrement as a key (primary key) for retrieving.