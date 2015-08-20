require 'dotenv/tasks'
require 'ruby-progressbar'

task default: %w(.env)

desc 'Setup necessary files and folders'
task setup: %w(tmp .env)

desc 'Copy .env.example to .env'
file '.env' do |t|
  cp '.env.example', '.env'
  puts 'Environment file created. Please fill in variables in .env'
end

directory 'tmp'

desc 'Download production postgres'
task down: [:dotenv, 'tmp', '.env'] do |t|
  IO.popen(['pg_dump', '--verbose', '--format=c', '--file=tmp/production_postgresql.dump', '--table=countries', ENV['DATABASE_URL']], err: [:child, 1]) do |f|

    reading_system = bar_create(title: 'Reading system')
    finding = bar_create(title: 'Finding columns, types, etc.', total: 50)
    reading_indexes = bar_create(title: 'Reading indexes')
    dumping = bar_create(title: 'Dumping tables', total: 50)
    puts

    while (out = f.gets)
      case out
      when /(reading indexes)|(reading triggers)|(reading foreign)/
        bar_refresh reading_system
        bar_refresh finding
        bar_refresh reading_indexes.increment, refresh: false
        bar_refresh dumping
      when /finding/
        bar_refresh reading_system
        bar_refresh finding.increment, refresh: false
        bar_refresh reading_indexes
        bar_refresh dumping
      when /reading|saving/
        bar_refresh reading_system.increment, refresh: false
        bar_refresh finding
        bar_refresh reading_indexes
        bar_refresh dumping
      when /dumping/
        bar_refresh reading_system
        bar_refresh finding
        bar_refresh reading_indexes
        bar_refresh dumping.increment, refresh: false
      end
      puts
    end

    reading_system.finish
    finding.finish
    reading_indexes.finish
    dumping.finish
    puts
  end
  puts 'Done.'
end

desc 'Load production db dump into local'
task load: [:dotenv, 'tmp', '.env']  do |t|
  cli_args = [
    '-cO',
    "--dbname=#{ENV['PGDATABASE']}",
    "--username=#{ENV['PGUSER']}",
  ]

  # Its possible to connect through socket / user map
  cli_args.merge! "--host=#{ENV['PGHOST']}" unless ENV['PGHOST'].nil? || ENV['PGHOST'] == ''
  cli_args.merge! "--port=#{ENV['PGPORT']}" unless ENV['PGPORT'].nil? || ENV['PGPORT'] == ''
  cli_args.merge! "PGPASSWORD=#{ENV['PGPASSWORD']}" unless ENV['PGPASSWORD'].nil? || ENV['PGPASSWORD'] == ''

  sh 'pg_restore', *cli_args, 'tmp/production_postgresql.dump'
end

def bar_create overrides = {}
    options = {
      total: 30,
      autofinish: false,
      length: 115,
      throttle_rate: 0.0001
    }

    bar = ProgressBar.create options.merge(overrides)
    puts

    bar
end

def bar_refresh progress_bar, options = {refresh: true}
  progress_bar.refresh if options[:refresh]
  puts
end

def refresh_all *bars
  bars.each do |bar|
    bar_refresh bar
  end
end
