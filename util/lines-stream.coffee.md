Create a unified stream for lines (local-number+endpoint)
---------------------------------------------------------

Provide a list of numbers and endpoints.

    most = require 'most'
    {from_ids} = require './pouchdb-as-stream'
    socketio_source = require './socket.io-source'

    lines_stream = ({db,io,numbers,endpoints}) ->
      numbers_ids = numbers.map (id) -> "number:#{id}"
      endpoint_ids = endpoints.map (id) -> "endpoint:#{id}"

      most.mergeArray [

CouchDB source

        from_ids db, numbers_ids.concat endpoint_ids

Location

        socketio_source io, 'location:response'
        socketio_source io, 'location:update'

Reports

        socketio_source io, 'report'

Queuer

        socketio_source io, 'queuer'

      ]

    module.exports = lines_stream
