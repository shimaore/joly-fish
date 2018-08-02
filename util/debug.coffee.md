A very small debugger, with a minimalist interface reminiscent of `tangible`.

    module.exports = (tag) ->
      debug = (args...) ->
        console.log tag, args...

      debug.catch = (msg) ->
        (error) ->
          console.error tag, msg, error

      heal = (p) -> p.catch debug.catch 'Caught'

      {debug,heal}
