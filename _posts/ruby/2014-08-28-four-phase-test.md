---
layout: post
title:  "Four-Phase Test"
date:   2014-08-28 17:00:00
categories: ruby
tags: testing
author: "Victor"
---

The Four-Phase Test is a testing pattern, applicable to all programming languages and unit tests (not so much integration tests).

It takes the following general form:

```ruby
test do
  setup
  exercise
  verify
  teardown
end
```

There are four distinct phases of the test. They are executed sequentially.

### setup

During setup, the system under test (usually a class, object, or method) is set up.

```ruby
user = User.new(password: 'password')
```

### exercise

During exercise, the system under test is executed.

```ruby
user.save
```

### verify

During verification, the result of the exercise is verified against the developer’s expectations.

```ruby
user.encrypted_password.should_not be_nil
```

### teardown

During teardown, the system under test is reset to its pre-setup state.

This is usually handled implicitly by the language (releasing memory) or test framework (running inside a database transaction).

### all together

The four phases are wrapped into a named test to be referenced individually.

Our style guide advises we “Separate setup, exercise, verification, and teardown phases with newlines.”

```ruby
it 'encrypts the password' do
  user = User.new(password: 'password')

  user.save

  user.encrypted_password.should_not be_nil
end
```

Go forth and test in phases.

## 原文来自

http://robots.thoughtbot.com/four-phase-test
