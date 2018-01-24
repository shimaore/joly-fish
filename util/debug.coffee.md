A very small debugger, with a minimalist interface reminiscent of `tangible`.

    seem = require 'seem'
    module.exports = (tag) ->
      debug = (args...) ->
        console.log tag, args...

      debug.catch = (msg) ->
        (error) ->
          console.error tag, msg, error

      heal = (p) -> p.catch debug.catch 'Caught'

      hand = (f) ->
        F = seem f
        (args...) -> heal F args...

      {debug,hand,heal}
