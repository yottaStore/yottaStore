# Yottadb


## Drivers

### Key-value

Basic driver, given a record in the form: 
- `account@[driver:]tableName/recordName[/recordRow]`

it allows to find which shard contains the binary data associated
with the record.

Possible operations are:

- Read
- Write
- Append
- Delete

### Columnar

Like key-value, but optimized for storing very long columns.

Possible operations are:

- Read
- Write
- Append
- Delete

### Document

Improvement over the key-value driver, allow to deal with
documents instead of plain binary data. Possible operations are:

- Read document
- Write document
- Update document
- Delete document
- Get collection
- Create collection
- Update collection
- Delete collection

### PubSub

Improvement over the columnar store, to handle queues. Possible
operations are:

- Crud Topic
- Publish
- Crud Subscription
- Consume

### Indexed

An improvement over the document driver, it uses the 
columnar driver to create indexes.

### Graph

Uses key-value store to represent a graph
