# Cucumber Style Guide
A style guide used by the ControlsInsight team at Rapid7's cucumber suite.

## Table of Contents
1. [Gherkin](#gherkin)
1. [Step Definitions](#step-definitions)
1. [Hooks](#hooks)
1. [Code Portability](#code-portability)
1. [Tags](#tags)

## References
1. [Capybara - The DSL](https://github.com/jnicklas/capybara#the-dsl)
1. [RSpec Expectations - Built in matchers](https://github.com/rspec/rspec-expectations#built-in-matchers)

## Gherkin
```gherkin
# features/ui/login.feature
@ui
Feature: Login page
  As a controlsinsight user
  I want to visit the login page
  In order to gain access to information on my security controls

  Scenario: Invalid user and password login
    Given I have opened the controlsinsight login page
    When I try to login as "johndoe" with the password "Secret123"
    Then I should see the error:
      """
        Sorry. Those credentials did not work. Please try again.
      """

  Scenario Outline: Invalid users attempt to login
    Given I have opened the controlsinsight login page
    When I try to login as "<user>" with the password "<password>"
    Then I should see the error:
      """
        Sorry. Those credentials did not work. Please try again.
      """
    
    Examples:
      |   user  | password |
      | janedoe | notpass! |
      | jdoe    | sEcrEt34 |
```

## Step Definitions
```ruby
# features/support/env.rb
require 'rspec'
require 'rspec/expectations'
require 'capybara/cucumber'
```
```ruby
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

## Hooks
```ruby
# features/support/hooks.rb
require_relative './hooks/capybara/'
```

## Code Portability
```ruby
# lib/helpers/ui_helpers.rb
module UIHelpers
  def login(username, password)
    visit '/'
    
    expect(find('.login-wrapper').text).to eq('Welcome to ControlsInsight by Rapid7 LOG ON')

    fill_in 'username', :with => username
    fill_in 'password', :with => password

    click_on 'Log on'
  end
end

World(UIHelpers)
```

```ruby
# features/step_definitions/login_steps.rb
Given "I have logged into controlsinsight" do
  login ENV['CONTROLS_USERNAME'], ENV['CONTROLS_PASSWORD']
end
```

## Tags
### Gherkin
```gherkin
# features/ui/login.feature
  @ui @wip
  Scenario: Session timeout
    Given I have logged into controlsinsight
    When I allow my session to timeout
    Then I should see the error:
      """
        The current session has expired. Please enter your credentials to continue.
      """
```

### Tagged Hooks
```ruby
# features/support/hooks.rb
After '@ui,@wip' do |scenario|
  $stdout.puts "\a\nPaused on '#{scenario.name}'. Press enter/return to continue to the next test."
  $stdin.gets
end
```
