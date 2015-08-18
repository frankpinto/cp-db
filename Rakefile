require 'dotenv/tasks'

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

  sh 'pg_dump', '--format=c', '--file=tmp/production_postgresql.dump', ENV['DATABASE_URL']
end
