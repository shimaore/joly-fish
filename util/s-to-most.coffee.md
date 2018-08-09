Convert a S.js signal into a Most.js stream
-------------------------------------------

    most = require 'most'

    {S} = require './surplus'
    s_to_most = (signal) ->
      h = null
      S.on signal, -> h? value: signal

      most
      .unfold ->
        new Promise (resolve) ->
          h = resolve
      .map S.sample

    module.exports = s_to_most
