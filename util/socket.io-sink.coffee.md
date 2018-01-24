    most = require 'most'

Stream towards socket.io as event
---------------------------------

    socketio_sink = (io,message,stream) ->
      stream
        .forEach (data) -> io.emit message, data
        .catch (error) -> io.emit 'error', error

    module.exports = socketio_sink
