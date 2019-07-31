    debug = (require 'debug') 'marked-summer:update-version'
    sleep = require './sleep'

    semver = require 'semver'

Return true if the current version ≥ our version ?

    ok = (current_version,our_version) ->
      return false unless current_version?
      semver.gte (semver.coerce current_version), (semver.coerce our_version)

    maximum_attempts = Infinity
    if process.env.UPDATE_VERSION_MAXIMUM_ATTEMPTS?
      maximum_attempts = parseInt process.env.UPDATE_VERSION_MAXIMUM_ATTEMPTS, 10

    module.exports = update_version = (db,ddoc) ->
      version = null
      timer = 43+Math.random()*319
      attempts = 0
      until attempts > maximum_attempts or ok version, ddoc.version

        {version,_rev} = await db
          .get ddoc._id
          .catch (error) ->
            if error.status in [404,409]
              {}
            else
              debug 'retrieving design document failed', ddoc._id, error
              Promise.reject error

        unless ok version, ddoc.version
          debug 'updating design document', ddoc._id, ddoc.version
          if _rev?
            ddoc._rev = _rev

Introduce a delay in case the user is trying this operation multiple times concurrently.

          await sleep timer

          timer *= 2 + Math.random()

          await db
            .put ddoc
            .catch -> null

        attempts++

      return
