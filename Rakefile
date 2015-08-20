require 'dotenv/tasks'
require 'ruby-progressbar'

task default: %w(.env)

task setup: %w(.env)

desc 'Copy .env.example to .env'
file '.env' do |t|
  cp '.env.example', '.env'
  puts 'Environment file created. Please fill in variables in .env'
end

directory 'tmp'

desc 'Download production postgres'
task down: [:dotenv, 'tmp', '.env']  do |t|
  # Get production credentials from .env
  # Use pg_backup to get a local copy of the production database
  # Get local credentials from .env
  # Load the production database into local db


  IO.popen(['pg_dump', '--verbose', '--format=c', '--file=tmp/production_postgresql.dump', '--table=countries', ENV['DATABASE_URL']], err: [:child, 1]) do |f|
    options = {
      total: 30,
      autofinish: false,
      length: 115,
      throttle_rate: 0.0001
    }

    reading_system = ProgressBar.create options.merge(title: 'Reading system')
    puts
    finding = ProgressBar.create options.merge(title: 'Finding columns, types, etc.', total: 50)
    puts
    reading_indexes = ProgressBar.create options.merge(title: 'Reading indexes')
    puts
    dumping = ProgressBar.create options.merge(title: 'Dumping tables', total: 50)
    puts
    puts

    while (out = f.gets)
      case out
      when /(reading indexes)|(reading triggers)|(reading foreign)/
        reading_system.refresh
        puts
        finding.refresh
        puts
        reading_indexes.increment
        puts
        dumping.refresh
        puts
        puts
      when /finding/
        reading_system.refresh
        puts
        finding.increment
        puts
        reading_indexes.refresh
        puts
        dumping.refresh
        puts
        puts
      when /reading|saving/
        reading_system.increment
        puts
        finding.refresh
        puts
        reading_indexes.refresh
        puts
        dumping.refresh
        puts
        puts
      when /dumping/
        reading_system.refresh
        puts
        finding.refresh
        puts
        reading_indexes.refresh
        puts
        dumping.increment
        puts
        puts
      end
    end

    reading_system.finish
    finding.finish
    reading_indexes.finish
    dumping.finish
    puts
  end
  puts 'Done.'
end
