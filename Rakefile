task default: %w(setup)

task :setup do |t|
  # Copy .env.example to .env
  cp '.env.example', '.env'
  puts 'Environment file created. Please fill in variables in .env'
end

task down: :setup  do |t|
  # Get production credentials from .env
  # Use pg_backup to get a local copy of the production database
  # Get local credentials from .env
  # Load the production database into local db
end
