require 'bundler'
require 'uri'
require 'cgi'

Bundler.require
Dotenv.load

USER_AGENT = 'CacheHorseBot/1.0.0 (bot@cache.horse)'.freeze
CH_API = ENV['CACHEHORSE_API_URI']
CH_KEY = ENV['CACHEHORSE_API_KEY']

def safe_uri(uri_string = '') = CGI.escape(uri_string)

urls = File.read('./config/urls.txt').split.map do |raw_uri|
  uri = URI.parse raw_uri

  {
    label: uri.hostname,
    raw_uri: raw_uri,
    cached_uri: "#{CH_API}/get?api_key=#{CH_KEY}&uri=#{safe_uri(raw_uri)}"
  }
end

def request(uri)
  thread = Thread.new do
    connection = Faraday.new(url: uri, headers: { 'User-Agent': USER_AGENT })
    connection.get
  end

  thread.join
end

def benchmark_requests(list)
  list.map do |api|
    raw_realtime = Benchmark.realtime do
      request(api[:raw_uri])
    end

    # The cache will be empty before running the test, so let's fill the cache first.
    request(api[:cached_uri])

    cache_hit_realtime = Benchmark.realtime do
      request(api[:cached_uri])
    end

    { raw_realtime: raw_realtime, cache_realtime: cache_hit_realtime, label: api[:label] }
  end
end

namespace :benchmark do
  desc 'Run the benchmark with URLs from `config/urls.txt`'
  task :run do
    puts "=> Running benchmark via Cache Horse API <#{CH_API}>"

    results = benchmark_requests(urls)

    table = Terminal::Table.new(headings: ['Domain', 'Raw request time', 'Cached request time', 'Time savings']) do |t|
      results.each do |row|
        t << [row[:label], row[:raw_realtime], row[:cache_realtime], row[:raw_realtime] - row[:cache_realtime]]
      end
    end

    puts table
  end
end
