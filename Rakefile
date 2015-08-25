require 'dotenv/tasks'
require 'ruby-progressbar'
require 'uri'

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
  cli_args = [
    '--verbose',
    '--format=c',
    '--file=tmp/production_postgresql.dump',
  ]

  if tables = ENV['TABLE'] || ENV['TABLES']
    tables = tables.split ','
    tables.each do |table|
      cli_args << '--table=' + table
    end
  end

  IO.popen(['pg_dump', *cli_args, ENV['DATABASE_URL']], err: [:child, 1]) do |f|

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
  cli_args <<  "--host=#{ENV['PGHOST']}" unless ENV['PGHOST'].nil? || ENV['PGHOST'] == ''
  cli_args << "--port=#{ENV['PGPORT']}" unless ENV['PGPORT'].nil? || ENV['PGPORT'] == ''

  command = "pg_restore #{cli_args.join(' ')} tmp/production_postgresql.dump"
  command = "PGPASSWORD=#{ENV['PGPASSWORD']} " + command unless ENV['PGPASSWORD'].nil? || ENV['PGPASSWORD'] == ''

  sh command
end

desc 'Download production mongodb'
task mongo_down: [:dotenv, 'tmp', '.env'] do |t|
  parts = URI.parse ENV['MONGODB_URL']
  db = parts.path[1..-1]

  cli_args = [
    "--host=#{parts.host}",
    "--port=#{parts.port}",
    "--db=#{db}",
    "--username=#{parts.user}",
    "--password=#{parts.password}",
    "--out=tmp"
  ]

  command = "mongodump"

  if collections = ENV['COLLECTION'] || ENV['COLLECTIONS']
    collections = collections.split ','
    collections.each do |collection|
      sh command, *cli_args, "--collection=#{collection}"
    end
  else
    sh command, *cli_args
  end
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
