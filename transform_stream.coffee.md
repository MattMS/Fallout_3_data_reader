# Read file stream

## Library imports

	bunyan = require 'bunyan'

	{isEmpty, last, pick} = require 'ramda'

	through2 = require 'through2'


## Relative imports

	get_initial_state = require './parser/get_initial_state'

	parse_file_data = require './parser/main'


## Logging

	log = bunyan.createLogger
		name: 'transform_stream'
		serializers:
			state: pick([
				'buffer'
				'group_number'
				'group_stop_byte'
				'record_number'
				'record_stop_byte'
				'stop_byte'
			])


## Exports

	module.exports = ->
		state = get_initial_state()

> Note that `through2.obj(fn)` is a convenience wrapper around `through2({objectMode: true}, fn)`.

		through2.obj (chunk, encoding, done)->
			if not Buffer.isBuffer(chunk)
				log.error('Chunk is not a Buffer.')

			else
				state.buffer = Buffer.concat([state.buffer, chunk])

				results = parse_file_data(state)

				if not isEmpty(results)
					state = last(results)


### Output values

Loop over the results returned and push each out to the Stream consumer.

				results.forEach (item)=>
					@push item


### End

			done()
