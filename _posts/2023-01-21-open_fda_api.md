---
layout: post
title:  "proj: Open FDA API"
date:   2023-01-21
categories: proj
permalink: /proj-open-fda-api/
---

[Open FDA API](https://rubygems.org/gems/open_fda_api) is a Ruby gem to interact with the U.S. Food and Drug Administration's public API.

Here's how to get started:

## Installation

Include the gem into your Gemfile.
```ruby
# Gemfile
gem 'open_fda_api'
```

Run bundler to install the gem.
```shell
bundle install
```

## Usage

First, create a client instance:
```ruby
require 'open_fda_api'

client = OpenFdaApi::Client.new
# or if you have registered an API key
client = OpenFdaApi::Client.new(api_key: ENV['OPEN_FDA_API_KEY'])
```

Then you can use the client to get public data about nouns like `drugs`, `devices`, and `food`, etc:
```ruby
# Animal & Veterinary API
client.animal_and_veterinary.adverse_events(args)

# Drug API
client.drugs.adverse_events(args)
client.drugs.product_labeling(args)
client.drugs.ndc_directory(args)
client.drugs.recall_enforcement_reports(args)
client.drugs.drugs_at_fda(args)

# Device API
client.device.premarket_510ks(args)
client.device.classification(args)
client.device.recall_enforcement_reports(args)
client.device.adverse_events(args)
client.device.premarket_approval(args)
client.device.recalls(args)
client.device.registrations_and_listings(args)
client.device.covid19_serological_tests(args)
client.device.unique_device_identifier(args)

# Food API
client.food.recall_enforcement_reports(args)
client.food.adverse_events(args)

# Other API
client.other.nsde(args)
client.other.substance_data_reports(args)

# Tobacco API
client.tobacco.problem_reports(args)
```

The `args` being passed into these endpoints are the search arguments that you want to filter on.

Here's what the structure of the `args` hash should look like:

```ruby
# First 20 results where (fieldA=foo AND fieldB=bar) OR (fieldC=baz AND fieldA exists)
# Sorted by (fieldD) in descending order
# Skip the first 8 results and limit to 20 first results
args = {
  search: [{"fieldA" => "foo", "fieldB" => "bar"}, {"fieldC" => "baz", "_exists_" => "fieldA"}],
  sort: [{"fieldD" => "desc"}],
  skip: 8,
  limit: 20
}
```

## Examples

Let's go through some examples.

### Scenario 1

Let's search for food recall enforcement reports in Orlando.
```ruby
require 'open_fda_api'

client = OpenFdaApi::Client.new
args   = { search: [{city: 'Orlando'}], limit: 20 }
client.food.recall_enforcement_reports(**args)["results"].map { |result| result["report_date"]}
# => 
# ["20120815",
#  "20121024",
#  "20120926",
#  ...
# ]
```

### Scenario 2

One sample with a positive antibody test under COVID-19 Serological Testing Evaluations.

```ruby
require 'open_fda_api'

client = OpenFdaApi::Client.new
result = client.device.covid19_serological_tests(**{search: [{antibody_truth:"Positive"}], limit: 1}).fetch("results").first
result.fetch("device")

# => "ADVAITE RapCOV Rapid COVID-19 Test"
```

### Scenario 3

Adverse event report with a drug from a certain pharmacologic class.

```ruby
require 'open_fda_api'

client = OpenFdaApi::Client.new
args = {search: [{ "patient.drug.openfda.pharm_class_epc" => "nonsteroidal+anti-inflammatory+drug"}] }
result = client.drugs.adverse_events(**args).fetch("results").first

result.dig("receivedate")
# => "20140228"

```

## Summary

The gem is still pretty new and needs to be iterated on but is mostly usable at the moment.

Some things that could be improved upon are how the search query is created - it needs to be much more user friendly.

The FDA provides a downloadable list of fields that can be queried upon for each endpoint. The gem should use that list to verify that queries are valid.

If anyone is reading this and wants let me know about any issues, please feel free to [open an issue on Github](https://github.com/hebron-george/open_fda_api/issues).

If you're interested in helping to improve the gem, please feel free to [open a Pull Request against the repo](https://github.com/hebron-george/open_fda_api) as well!
