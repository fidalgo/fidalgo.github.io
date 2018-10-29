---
layout: post
title: Ruby test coverage by feature branch
subtitle: How to get your current feature branch coverage test
tags: ruby tests coverage git branch
---

Having a git workflow based on feature branches, where you create a new branch for every new story you are working on, allows you to have
a strict barrier on what files changed.

But, when you have a global coverage ratio, it's it's hard to know if you're working torwards increasing this ratio or making sure that every line (or critical parts)
are covered by tests.

When coding in Ruby, the usual suspect for coverage reports is [SimpleCov](https://github.com/colszowka/simplecov).

I won't digg in SimpleCov features or in setup, as it's best described in the project page, but how do we get the branch coverage as promised?

# Configure SimpleCov to include a branch coverage report

We start with your  helper file, like the documentation states:

```ruby
require 'simplecov'
SimpleCov.start 'rails'
```

Assuming the _reference_ branch is `master`, we want to report all the files that changed in current branch, against `master` branch.
On way to achieve it, is to create an helper that would run a `git diff` command and gather the output. So we'll end up with a configuration similar to this:

```ruby
require 'simplecov'

class GitBranchFiles
  class << self
    def include?(filename)
      @changed_files ||= git_branch_changed_files
      @changed_files.include?(filename)
    end

    private

    def git_branch_changed_files
      `git diff --name-only master`.split(/\n/).map do |file|
        file_path = Pathname.new(Rails.root + file)
        file_path.to_s if file_path.extname =~ /.rb/
      end.compact
    end
  end
end


SimpleCov.start('rails') do
  add_group 'Current Branch' do |source_file
    GitBranchFiles.include?(source_file.filename)
  end
end


```

After running your tests you'll have a new tab with the changed files of your current branch, where you can check the coverage and improve it if necessary.

























































