Socket.io as a source for a new stream
--------------------------------------

    most = require 'most'

    socketio_source = (io,message) ->
      most.fromEvent message, io

    module.exports = socketio_source
