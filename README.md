# Shiba

Shiba is a tool that helps catch poorly performing queries before they cause problems in production. By default, it will detect queries that miss indexes. As it's fed more information, it warns about advanced problems, such as queries that use indexes but are still very expensive.

To help find such queries, Shiba monitors test runs for ActiveRecord queries. A warning and report are then generated. Shiba is further capable of only warning on changes that occured on a particular git branch/pull request to allow for CI integration.

## Installation

Install using bundler. Note: this gem is not designed to be run on production.

```
gem 'shiba', :group => :test, :require => true
```

## Usage

```ruby
# Install
bundle

# Run some tests using to generate a SQL report
rake test:functional
rails test test/controllers/users_controller_test.rb

# 1 problematic query detected
# Report available at /tmp/shiba-explain.log-1550099512
```

## Going beyond table scans

For smarter analysis, Shiba requires general statistics about production data, such as the number of rows in a table and how unique columns are.

This information can be obtained by running the bin/dump_stats command in production.

```
production_host: git clone https://github.com/burrito-brothers/shiba.git
production_host: cd shiba ; bundle
production_host: bin/dump_stats DATABASE_NAME [MYSQLOPTS] > ~/shiba_index.yml
local: scp production_host:~/shiba_index.yml MYPROJECT/config
```

The stats file will look similar to the following:

```
users:
  count: 10000
  indexes:
    PRIMARY:
      name: PRIMARY
      columns:
      - column: id
        rows_per: 1
      unique: true
    index_users_on_login:
      name: index_users_on_login
      columns:
      - column: login
        rows_per: 1
      unique: true
    index_users_on_created_by_id:
      name: index_users_on_created_by_id
      columns:
      - column: created_by_id
        rows_per: 3
      unique: false
```