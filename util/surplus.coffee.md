Import 'Surplus' and its companions in a way compatible with CommonJS
(aka `require`).

    Surplus = require 'surplus/index.js'
    {S} = Surplus
    SArray = (require 's-array/index.js').default
    data = require 'surplus-mixin-data/index.js'

    module.exports = {Surplus,S,SArray,data}
