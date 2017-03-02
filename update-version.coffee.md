    seem = require 'seem'
    debug = (require 'debug') 'marked-summer:update-version'
    sleep = require './sleep'

    module.exports = update_version = seem (db,ddoc) ->
      version = null
      timer = 43+Math.random()*319
      until version is ddoc.version

        {version,_rev} = yield db
          .get ddoc._id
          .catch (error) ->
            if error.status is 409
              {}
            else
              Promise.reject error

        unless version is ddoc.version
          debug 'updating design document', ddoc._id, ddoc.version
          if _rev?
            ddoc._rev = _rev

Introduce a delay in case the user is trying this operation multiple times concurrently.

          yield sleep timer

          timer *= 2 + Math.random()

          yield db
            .put ddoc
            .catch -> null
