    describe 'The components', ->
      ['agent'].forEach (component) ->
        it "should load #{component}", -> require "../component/#{component}"

    describe 'The utilities', ->
      [
        'lines-stream'
        's-to-most'
        'surplus'
        'uniq'
        'debug'
        'pouchdb-as-stream'
        'oboe-as-stream'
      ].forEach (util) ->
        it "should load #{util}", -> require "../util/#{util}"
