source 'https://rubygems.org'

gem 'rails', '3.2.6'

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

group :production, :staging do
  gem 'pg'
  gem 'thin'
  gem 'fog'
end

group :development, :test do
  gem 'sqlite3'
end

