require 'bundler'

Bundler.require
Dotenv.load

namespace :benchmark do
  desc 'Run the benchmark with URLs from `config/urls.txt`'
  task :run do
    puts "=> Running benchmark via Cache Horse API <#{ENV['CACHEHORSE_API_URI']}>"
  end
end
