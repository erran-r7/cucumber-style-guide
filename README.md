# Cucumber Style Guide
## Table of Contents
1. [Requirements](#requirements-)
1. [References](#references-)
1. [Gherkin](#gherkin-)
1. [Step Definitions](#step-definitions-)
1. [Hooks](#hooks-)
1. [Code Portability](#code-portability-)
1. [Tags](#tags-)

## [Requirements ↩](#table-of-contents)
### Ruby Environment Management
1. [rbenv](https://github.com/sstephenson/rbenv#readme) (Mac install) - Less magic, more control 
    * `brew update`
    * `brew install rbenv ruby-build`
    * `rbenv install 2.0.0-p353`
1. [rbenv](https://github.com/sstephenson/rbenv#readme) (Ubuntu install) 
    * Perform [a basic git checkout](https://github.com/sstephenson/rbenv#basic-github-checkout)
1. [RVM (Ruby Version Manager)](https://rvm.io) - Wide support (use this is you're new to Ruby)
  * `\curl -L https://get.rvm.io | bash -s stable --ruby # Installs the latest stable ruby version`
  * `source ~/.rvm/scripts/rvm`
1. Post install:
  * RVM:
    * `[[ -f ~/.bashrc ]] && echo 'eval "$(rbenv init -)"' >> ~/.bashrc`
    * `[[ -f ~/.bash_profile ]] && echo 'eval "$(rbenv init -)"' >> ~/.bash_profile`
  * rbenv:
    * `[[ -f ~/.bashrc ]] && echo ". ~/.rvm/scripts/rvm" >> ~/.bashrc`
    * `[[ -f ~/.bash_profile ]] && echo ". ~/.rvm/scripts/rvm" >> ~/.bash_profile`
### Cucumber suite
1. Clone your cucumber suite
1. `cd ~/cucumber-suite`
1. `gem install bundler`
1. `bundle install`
1. Set up your environment (`. ./scripts/env.sh`)
1. Use prefixes such as your products name so you can easily grep for your environment variables.
  * `env | grep '^YOUR_PREFIX_'` 
1. Run your first test using: `bundle exec cucumber features/ui/login.feature`

## [References ↩](#table-of-contents)
1. [Learn Ruby ecosystem in Y minutes](http://learnxinyminutes.com/docs/ruby-ecosystem/)
1. [Learn Ruby in Y minutes](http://learnxinyminutes.com/docs/ruby/)
1. [Capybara - The DSL](https://github.com/jnicklas/capybara#the-dsl)
1. [RSpec Expectations - Built in matchers](https://github.com/rspec/rspec-expectations#built-in-matchers)

## [Gherkin ↩](#table-of-contents)
A Gherkin file is a file that describes a set of expectations about product features in plain text.
The Gherkin file extension is `.feature`.

### User Stories
User stories are, just that, a story that details the use case in the user's perspective.
User stories follow the format:

```gherkin
Feature: Trending
    As a controlsinsight customer
    I want to see trending of security controls and trend grades
    In order to see daily, weekly, and monthly improvements
```

### Scenario
#### Anatomy of a Scenario
1. [Steps](#steps)
1. [Examples](#scenariosteps-example)

#### Steps
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
#### Scenario/Steps Example

```gherkin
Scenario: I click random buttons
  Given I have put the system in the known state
    And I have also tweaked a random configuration
  When I compare the last known state with the current state
  Then I should see that the last and current states are different
```

```gherkin
Examples:
  |   user  | password |
  | janedoe | notpass! |
  | jdoe    | sEcrEt34 |
```

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

## [Step Definitions ↩](#table-of-contents)
```ruby
# features/support/env.rb
require 'rspec'
require 'rspec/expectations'
require 'capybara/cucumber'
```
```ruby
# features/step_definitions/login_steps.rb
Given /^I have opened the controlsinsight login page$/ do
  visit '/'
end

When /^I try to login as "([^"]+)" with the password "([^"]+)"$/ do |username, password|
  fill_in 'username', :with => username
  fill_in 'password', :with => password

  click_link 'Log on'
end

Then /^I should see the error:$/ do |expected_error|
  expect(find('div.alert').text).to eq(expected_error)
end
```

## [Hooks ↩](#table-of-contents)
```ruby
# features/support/hooks.rb
require_relative './hooks/capybara/'
```

## [Code Portability ↩](#table-of-contents)
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
Given /^I have logged into controlsinsight$/ do
  login ENV['CONTROLS_USERNAME'], ENV['CONTROLS_PASSWORD']
end
```

## [Tags ↩](#table-of-contents)
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

###### 1. For more about verb tenses see [English verb tenses](https://owl.english.purdue.edu/owl/resource/601/01/) on Purdue OWL. [Return to steps ↩](#steps)
