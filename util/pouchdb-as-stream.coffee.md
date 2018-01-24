PouchDB as a stream of documents
--------------------------------

Will emit at least once for the initial document.

`db` is a PouchDB instance, ids an array of document IDs

    most = require 'most'

    pouchdb_from_ids_as_stream = (db,ids) ->
      changes = db.changes
        live: true
        include_docs: true
        since: 'now'
        doc_ids: ids

      initial = db.allDocs
        include_docs: true
        keys: ids

      most
      .fromPromise initial
      .chain ({rows}) -> most.from rows
      .continueWith ->
        most
        .fromEvent 'change', changes
        .until most.fromEvent('complete', changes).chain ({results}) -> most.from results
        .until most.fromEvent('error', changes)
        # .merge most.fromEvent('error', changes).map most.throwError
      .map ({doc}) -> doc
      .filter (doc) -> doc?

    module.exports =
      from_ids: pouchdb_from_ids_as_stream
