### What is a Searcher in Solr?
---

Related to the indexing in Solr: the Searcher (actually IndexSearcher as of Solr4) is the Solr / Lucene internal to the indexing and searching backend component, as you call it. The idea is that when you index documents in Solr, they become visible upon commit operation has finished. This is when the Searcher reopens the index and sees the changes that were done since the last commit operation.

Because reopening the Searcher is an expensive operation, there is a new feature in solr4, called soft-commit. It lets you commit as frequent as each second (configurable), for example, and make those committed documents visible immediately to your client app / ui. It is fast, because committing happens in RAM. You still need to do an ordinary hard commit to flush the changes to disk. This is required so that the changes do not get lost and you don't want to run out of RAM.

On a side note, you may want to use soft-commit with updateLog feature, which stores added documents and can be replayed even if Solr instance has crashed due to OutOfMemoryError or physical unplugging the indexing machine.

Also, from a code point of view a Seacher is just an Lucene Class which enables searching across the Lucene Index.

Searcher is an abstract base class that has various overloaded search methods. IndexSearcher is a commonly used subclass that allows searching indices stored in a given directory. The Search method returns an ordered collection of documents ranked by computed scores. Lucene calculates a score for each of the documents that match a given query. IndexSearcher is thread-safe; a single instance can be used by multiple threads concurrently.
