------------------------------------------------------------------
What does a segment file in Solr contian and general facts?
------------------------------------------------------------------

```
> A segment consist of multiple files, including the index (for example the term 'friday' appears in documents 1, 4, 5, whatever), docvalues if they are configured, the values of the fields marked as stored, etc. So yes, basically the document index data.
> When you do a hard commit, it closes the current transaction log files and moves them to a segment, which will then be closed and a new one opened.
> Once a segment is written, it is not modified again.
> When you delete or update a document, it will me marked in the next segment. Also for updates. Let's say you add a document.
> That will be stored in a segment in the subsequent hard commit.
> Then you modify the same document. The original segment will be untouched, but the updated document (with a new document version number) will be put into the next segment upon the next commit.
> The query execution is aware of multiple segments and document versions.
> However, this takes up additional space, and the deleted documents will also be taking up space in the original segment.
> That's why we have the so called merge policies which kick in upon commits. In the current solr versions the default one is the tieredmergepolicy.
> The goal is to limit the number of segment files by merging some of the segments together, and also to reclaim disk space occupied by deleted documents
> However, the "tieredmergepolicy" needs to do some balancing on how frequent it does segment merges (too frequent merge operations will slow down the indexing as it will need to constantly copy data back and forth to merge existing segments and delete old ones).
> When the merges are done rarely, we will end up with many segment files, causing the searches to slow down (by having to look through multiple segments)and also waste of disk space
> So, the tieredmergepolicy aims to draw a balance, to keep the number of segments relatively low but trying not to overload the system with frequent copy operations
> This balance can be controlled by the parameters in solrconfig.xml, like the mergefactor.
> This parameter is usually set to 10 (for infra solr AFAIK it is 5)
usually the default values provide a good balance, but rarely, if we think that the indexing operation is slowed down by the too frequent merges, we can adjust this
```
