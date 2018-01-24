    {Surplus,S,SArray} = require '../util/surplus'

Note that the `number` is static: if there is need for a different number, the enclosing element will add/remove the tag as needed.

    module.exports = ({number,endpoint,selector},{source,sink}) ->

View
----

      visible = (v) ->
        if v then '' else 'hidden'

      join = (a...) ->
        a.join ' '

      view = ->
        <li class={join 'list-group-item', line.state(), line.queuer_state(), visible line.visible()}>
          <span class="media-object pull-left" width="32" height="32">{if line.registered() then '☎' else '☏'}</span>
          <div class="media-body">
            <strong class="poste">
              {line.display_number} {line.display_name}
              <span class="remote">
                {line.from} {line.queuer_from}
              </span>
              <span class="agent" onclick={-> logout line}>
                <span class="queuer-count" class={visible line.queuer_count()}> — En cours: {line.queuer_count}</span>
                <span class="queuer-missed" class={visible line.queuer_missed()}> — Manqués: {line.queuer_missed}</span>
                <span class="queuer-state"> — {line.queuer_state}</span>
              </span>
            </strong>
            <p>
              <span class="appel">
                {line.state} {line.direction} {line.offhook}
              </span>
              <span class="tags">
                {line.tags.map (tag) ->
                  <span class="tag">{tag}</span>
                }
              </span>
            </p>
          </div>
        </li>

Model
-----

      Line = (t) ->

Keep the match values so that we can re-evaluate whether we need to be visible when our content changes.

        match_v:        S.data ''
        match_r:        S.data new RegExp ''

Data collected

        registered:     S.data t.registered ? false
        display_number: S.data t.display_number ? number.split('@')[0]
        display_name:   S.data t.display_name ? ''
        tags:           SArray t.tags ? []
        from:           S.data ''
        state:          S.data ''
        direction:      S.data ''
        queuer_state:   S.data ''
        queuer_missed:  S.data 0
        queuer_count:   S.data 0
        queuer_from:    S.data ''
        offhook:        S.data ''

      line = Line {}

      compare_options =
        usage:'search'
        sensitivity:'base'
        ignorePunctuation:true
      selector_visible = (v,r) ->
        return true if number.substr(0,v.length) is v
        st = line.display_name().substr(0,v.length)
        return true if 0 is st.localeCompare v, [], compare_options
        return r?.test line.search()

FIXME: This should be moved above but I still haven't figured out how to do it properly.

      S.root ->
        line.search = S ->
          tags = line.tags()
          [line.display_name(),line.state(),line.queuer_state(),tags...]
            .map (w) -> w ? ''
            .join ' '

        line.visible = S ->
          selector_visible line.match_v(), line.match_r()

Behavior
--------

      number_id = "number:#{number}"
      endpoint_id = "endpoint:#{endpoint}"
      this_id = ({_id}) -> _id is number_id or _id is endpoint_id
      this_number = ({_in}) -> _in? and number_id in _in
      this_aor = ({aor}) -> aor? and aor is endpoint

Process documents.

      source
      .filter this_id
      .filter ({_rev}) -> _rev?
      .observe (doc) ->
        line.display_number doc.display_number if doc.display_number?
        line.display_name doc.display_name if doc.display_name?

Process reports (calls in huge-play).

      source
      .filter this_number
      .filter ({type}) -> type is 'report'
      .observe (notification) ->
        switch notification.state
          when 'ingress-call'
            line.from notification.source
            line.state notification.state
            line.direction notification.direction
          when 'egress'
            line.from notification.destination
            line.state notification.state
            line.direction notification.direction
          when 'end'
            switch notification.direction
              # Ignore centrex-redirect
              when 'transferred'
                return
              # Required for off-hook queuer
              when 'queuer-offhook'
                line.state 'queuer'
              else
                line.state notification.state
            line.from null
            line.direction notification.direction

Process calls in the queuer (black-metal events, sent by huge-play/middleware/client/queuer).

      source
      .filter this_number
      .filter ({_queuer}) -> _queuer
      .observe (event) -> S.root ->
        line.queuer_state event.state if event.state?
        line.queuer_missed event.missed if event.missed?
        line.queuer_count event.count if event.count?
        line.queuer_from event.remote_number if event.remote_number?
        line.tags event.tags if event.tags?
        line.offhook event.offhook if event.offhook?
        return

Query queuer state.

      setTimeout ->
        sink.emit 'join', number_id, (ok) ->
          sink.emit 'queuer:get-agent-state', number
      , 2000+5000*Math.random()

Process responses from ccnq4-opensips.

      source
      .filter ({aor}) -> aor?
      .filter this_aor
      .filter ({_missing,contact}) -> contact? and not _missing
      .map ({_deleted}) -> not _deleted
      .observe line.registered

Query location state.

      setTimeout ->
        sink.emit 'join', endpoint_id, (ok) ->
          sink.emit 'location', endpoint
      , 8500+10000*Math.random()

      selector
      .observe ({r,v}) ->
        line.match_r r
        line.match_v v

      logout = (line) ->
        return unless confirm "Si l'agent est en cours d'appel cette opération causera des problèmes."
        sink.emit 'queuer:log-agent-out', number
        return

The module _must_ return the view.

      return view
