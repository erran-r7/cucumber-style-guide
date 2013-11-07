# Cucumber Style Guide
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
### Steps
The following table describes which step keywords to use and when to use them.<sup><a href="#1-for-more-about-verb-tenses-see-english-verb-tenses-on-purdue-owl">1</a></sup>

<table border="1">
    <tr>
        <th>Step (keyword)</th>
        <th>Purpose</th>
        <th>Verb tense</th>
    </tr>
    <tr>
        <td><b>Given</b></td>
        <td>To ensure that a precondition is met.</td>
        <td>Past imperfect</td>
    </tr>
    <tr>
        <td><b>When</b></td>
        <td>To get into a desired state (to be used to make assertions).</td>
        <td>Present</td>
    </tr>
    <tr>
        <td><b>Then</b></td>
        <td>To make assertions about the current state.</td>
        <td>Future</td>
    </tr>
    <tr>
        <td><b>And</b></td>
        <td>To prevent conjunction steps.</td>
        <td>Dependent on which step keyword is being replaced.</td>
    </tr>
    <tr>
        <td><b>But</b></td>
        <td>To prevent conjunction steps.</td>
        <td>Dependent on which step keyword is being replaced.</td>
    </tr>
</table>

### User Stories
### Gherkin Example
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
### Gherkin Example
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

---

###### 1. For more about verb tenses see [English verb tenses](https://owl.english.purdue.edu/owl/resource/601/01/) on Purdue OWL.[↩](#steps).
