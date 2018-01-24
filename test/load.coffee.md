    describe 'The components', ->
      ['agent'].forEach (component) ->
        it "should load #{component}", -> require "../component/#{component}"

    describe 'The utilities', ->
      ['lines-stream','s-to-most','surplus','uniq','utils'].forEach (util) ->
        it "should load #{util}", -> require "../util/#{util}"
