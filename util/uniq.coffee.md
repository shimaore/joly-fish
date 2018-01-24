Filter that keeps each value only once
--------------------------------------

Must be applied to very short streams.
Usage: `.filter(uniq())`

    uniq = (set = new Set) -> (id) ->
      if set.has id
        return false
      set.add id
      return true

    module.exports = uniq
