# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.6"

# jekyll-redirect-from lets old post URLs (date-based permalinks) keep
# resolving now that Chirpy's default permalink is /posts/:title/.
gem "jekyll-redirect-from", "~> 0.16"

gem "html-proofer", "~> 5.0", group: :test

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", platforms: [:windows]

gem "webrick", "~> 1.7"
