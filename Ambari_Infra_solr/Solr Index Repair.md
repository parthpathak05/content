o In case of index corruption, please follow the below procedure:
-----------------------------------------------------------------

1. Stop the Solr node.

2. Backup the index directory if data is of paramount importance.

3. Goto ```<Path_to_SolrInstall-X.Y.Z>/server/solr-webapp/webapp/WEB-INF/lib```

You should see lucene-core-X.Y.Z.jar here:

### Run the following command:

Syntax:
```
$ java -cp lucene-core-6.3.0.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex <path_to_index_folder> -exorcise
```

Example:
```
$ java -cp lucene-core-6.3.0.jar -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex /Users/Rohit/Documents/SolrInstall/solr-6.3.0/example/cloud/node1/solr/gettingstarted_shard1_replica1/data/index -exorcise
```

> This command will remove the corrupt segments from the index (so there might be some data loss as the corrupt segments are purged)
 
You will see an output similar to below:
----------------------------------------
```
Opening index @ /Users/Rohit/Documents/SolrInstall/solr-6.3.0/example/cloud/node1/solr/gettingstarted_shard1_replica1/data/index

Segments file=segments_1 numSegments=0 id=1gbklfuh4fgecaa913367pgxy

No problems were detected with this index.

Took 0.077 sec total.
```

o Also read the below information:
----------------------------------

> -exorcise: actually write a new segments_N file, removing any problematic segments. *LOSES DATA*
> -segment X: only check the specified segment(s). This can be specified multiple times, to check more than one segment, eg -segment _2 -segment _a. You can't use this with the -exorcise option.

*** WARNING: -exorcise should only be used on an emergency basis as it will cause documents (perhaps many) to be permanently removed from the index. Always make a backup copy of your index before running this! Do not run this tool on an index that is actively being written to. You have been warned!

Run without -exorcise, this tool will open the index, report version information and report any exceptions it hits and what action it would take if -exorcise were specified. With -exorcise, this tool will remove any segments that have issues and write a new segments_N file. This means all documents contained in the affected segments will be removed.

This tool exits with exit code 1 if the index cannot be opened or has any corruption, else 0.
