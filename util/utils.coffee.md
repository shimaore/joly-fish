    most = require 'most'

PouchDB as a stream of documents
--------------------------------

Will emit at least once for the initial document.

`db` is a PouchDB instance, ids an array of document IDs

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

Oboe as stream
--------------

    oboe_stream = (e,o) ->
      most
      .fromEvent "node:#{e}", o
      .map ([v]) -> v
      .until most.fromEvent('done',o)
      .until most.fromEvent('fail',o)
      # .until most.fromEvent('fail',o).chain (error) -> most.throwError error

Stream towards socket.io as event
---------------------------------

    socketio_sink = (io,message,stream) ->
      stream
        .forEach (data) -> io.emit message, data
        .catch (error) -> io.emit 'error', error

Socket.io as a source for a new stream
--------------------------------------

    socketio_source = (io,message) ->
      most.fromEvent message, io

    module.exports = {
      pouchdb_from_ids_as_stream
      oboe_stream
      socketio_sink
      socketio_source
    }
