Set breakpoints at 4 places and determined the file short names at these places.

1. [`OnlineFileSelectionStrategy.java:1029`](https://github.com/snowflakedb/snowflake/blob/c3e85eb3da53ead41a1ed7e543c31b14ed242dde/GlobalServices/src/main/java/com/snowflake/sql/compiler/fileselection/OnlineFileSelectionStrategy.java#L1029): Looking at `batchSet`
2. [`PartitionScanSet.java:1205`](https://github.com/snowflakedb/snowflake/blob/c3e85eb3da53ead41a1ed7e543c31b14ed242dde/GlobalServices/src/main/java/com/snowflake/core/scanset/PartitionScanSet.java#L1205): Looking at `scanFiles`
    * Hits once with `shouldUnregisterScanFiles` as `false` then a second time with `true`
3. [`V2QueryCoordinator.java`](https://github.com/snowflakedb/snowflake/blob/c3e85eb3da53ead41a1ed7e543c31b14ed242dde/GlobalServices/src/main/java/com/snowflake/core/V2QueryCoordinator.java#L4085): Looking at `toUnregister`
4. [`TableCtx.java:4543`](https://github.com/snowflakedb/snowflake/blob/c3e85eb3da53ead41a1ed7e543c31b14ed242dde/GlobalServices/src/main/java/com/snowflake/core/TableCtx.java#L4543): Looking at `filesUnregisterData`

With
```java
  public boolean shouldUnregisterScanFiles() {
    switch (insertScanMode) {
      ...
      case CLUSTERING_SERVICE_GROUPED_EXECUTION:
        return checkIsDmlSource();
    }
  }
```

The short names for the files are exactly the same at every point.

At file selection, we find the files that are selected for reclustering. At PartitionScanSet.java, we hit the breakpoint twice but only set the files to unregistered once. Then for the rest of the pipeline, we only hit the breakpoints once with exactly the same file short names as from file selection.

With
```java
  public boolean shouldUnregisterScanFiles() {
    switch (insertScanMode) {
      ...
      case CLUSTERING_SERVICE_GROUPED_EXECUTION:
        return true;
    }
  }
```

In `PartitionScanSet`, `V2QueryCoordinator` and `TableCtx`, we have the same set of files that we selected in file selection appear twice.
