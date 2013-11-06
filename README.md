# Cucumber Style Guide
A style guide used by the ControlsInsight team at Rapid7's cucumber suite.

## Table of Contents
1. [Gherkin](#gherkin)
1. [Step Definitions](#step-definitions)
1. [Hooks](#hooks)
1. [Code Portability](#code-portability)
1. [Tags](#tags)
1. [Utilizing DSL Methods](#utilizing-dsl-methods)

## Gherkin
```gherkin
Feature: Login page
  As a controlsinsight user
  I want to visit the login page
  In order to gain access to information on my security controls

  Scenario: Invalid user
    Given I have opened the controlsinsight login page
    When I try to login as "johndoe" with the password "Secret123"
    Then I should see the error:
      """
        Sorry. Those credentials did not work. Please try again.
      """
```

## Step Definitions
```ruby
# features/support/env.rb
require 'rspec'
require 'rspec/expectations'
require 'capybara/cucumber'

# features/step_definitions/login_steps.rb
Given "I have opened the controlsinsight login page" do
  visit '/'
end

When /I try to login as "([^"]+)" with the password "([^"]+)"/ do |username, password|
  fill_in 'username', :with => username
  fill_in 'password', :with => password

  click_link 'Log on'
end

Then 'I should see the error:' do |expected_error|
  expect(find('div.alert').text).to eq(expected_error)
end
```


## Code Portability


## Tags

## Utilizing DSL Methods
