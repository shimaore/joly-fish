Oboe as stream
--------------

    most = require 'most'

    oboe_stream = (e,o) ->
      most
      .fromEvent "node:#{e}", o
      .map ([v]) -> v
      .until most.fromEvent('done',o)
      .until most.fromEvent('fail',o)
      # .until most.fromEvent('fail',o).chain (error) -> most.throwError error

    module.exports = oboe_stream
