[![Build Status](https://travis-ci.org/donv/capybara-screenshot-diff.svg?branch=master)](https://travis-ci.org/donv/capybara-screenshot-diff)

# Capybara::Screenshot::Diff

Ever wondered what your project looked like two years ago?  To answer that, you
start taking screen shots during your tests.  Capybara provides the
`save_screenshot` method for this.  Very good.

Ever introduced a graphical change unintended?  Never want it to happen again?
Then this gem is for you!  Use this gem to detect changes in your pages by
taking screen shots and comparing them to the previous revision.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'capybara-screenshot-diff'
```

Then include to your tests `Capybara::Screenshot::Diff`:

```ruby
describe 'Some feature', :type => :feature, :js => true do
  include Capybara::Screenshot::Diff
  ...
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install capybara-screenshot-diff

## Usage

Add `screenshot '<my_feature>'` to your tests.  The screenshot will be saved in
the `test/screenshots` directory.

Change your existing `save_screenshot` calls to `screenshot`

```ruby
test 'my useful feature' do
  visit '/'
  screenshot 'welcome_index'
  click_button 'Useful feature'
  screenshot 'feature_index'
  click_button 'Perform action'
  screenshot 'action_performed'
end
```

This will produce a sequence of images like this

```
test
  screenshots
    action_performed
    feature_index
    welcome_index
```

To store the screen shot history, add the `test/screenshots` directory to your
 version control system (git, svn, etc).

### Screenshot groups

Commonly it is useful to group screenshots around a feature, and record them as
a sequence.  To do this, add a `screenshot_group` call to the start of your
test.

```ruby
test 'my useful feature' do
  screenshot_group 'useful_feature'
  visit '/'
  screenshot 'welcome_index'
  click_button 'Useful feature'
  screenshot 'feature_index'
  click_button 'Perform action'
  screenshot 'action_performed'
end
```

This will produce a sequence of images like this

```
test
  screenshots
    useful_feature
      00-welcome_index
      01-feature_index
      02-action_performed
```

**All files in the screenshot group directory will be deleted when
`screenshot_group` is called.**


#### Screenshot sections

You can introduce another level above the screenshot group called a
`screenshot_section`.  The section name is inserted just before the group name
in the save path.  If called in the setup of the test, all screenshots in
that test will get the same prefix:

```ruby
setup do
  screenshot_section 'my_feature'
end

test 'my subfeature' do
  screenshot_group 'subfeature'
  visit '/feature'
  click_button 'Interresting button'
  screenshot 'subfeature_index'
  click_button 'Perform action'
  screenshot 'action_performed'
end
```

This will produce a sequence of images like this

```
test
  screenshots
    my_feature
      subfeature
        00-subfeature_index
        01-action_performed
```



### Multiple Capybara drivers

Often it is useful to test your app using different browsers.  To avoid the
screenshots for different Capybara drivers to overwrite each other, set

```ruby
Capybara::Screenshot.add_driver_path = true
```

The example above will then save your screenshots like this
(for poltergeist and selenium):

```
test
  screenshots
    poltergeist
      useful_feature
        00-welcome_index
        01-feature_index
        02-action_performed
    selenium
      useful_feature
        00-welcome_index
        01-feature_index
        02-action_performed
```

### Multiple OSs

If you run your tests on multiple operating systems, you will most likely find
the screen shots differ.  To avoid the screenshots for different OSs to
overwrite each other, set

```ruby
Capybara::Screenshot.add_os_path = true
```

The example above will then save your screenshots like this
(for Linux and Windows):

```
test
  screenshots
    linux
      useful_feature
        00-welcome_index
        01-feature_index
        02-action_performed
    windows
      useful_feature
        00-welcome_index
        01-feature_index
        02-action_performed
```

If you combine this config with the `add_driver_path` config, the driver will be
put in front of the OS name.

### Screen size

You can specify the desired screen size using

```ruby
Capybara::Screenshot.window_size = [1024, 768]
```

This will force the screen shots to the given size, and skip taking screen shots
unless the desired window size can be achieved.

### Disabling screen shots

If you want to skip taking screen shots, set

```ruby
Capybara::Screenshot.enabled = false
```

You can of course set this by an environment variable

```ruby
Capybara::Screenshot.enabled = ENV['TAKE_SCREENSHOTS']
```

### Disabling diff

If you want to skip the assertion for change in the screen shot, set

```ruby
Capybara::Screenshot::Diff.enabled = false
```

Using an environment variable

```ruby
Capybara::Screenshot::Diff.enabled = ENV['COMPARE_SCREENSHOTS']
```

### Screen shot save path

If you would like the screen shots to be saved in a different location set

```ruby
Capybara::Screenshot.save_path = "spec/screenshots"
```

### Screen shot stability

To ensure that animations are finished before saving a screen shot, you can add
a stability time limit.  If the stability time limit is set, a second screen
shot will be taken and compared to the first.  This is repeated until two
subsequent screen shots are identical.

```ruby
Capybara::Screenshot.stability_time_limit = 0.5
```



### Removing focus from the active element

In Chrome the screenshot includes the blinking input cursor.  This can make it impossible to get a
stable screenshot.  To get around this you can set the `blur_active_element` option:

```ruby
Capybara::Screenshot.blur_active_element = true
```

This will remove the focus from the active element, removing the blinking cursor.



### Allowed color distance

Sometimes you want to allow small differences in the images.  For example, Chrome renders the same
page slightly differently sometimes.  You can set set the color difference threshold for the
comparison using the `color_distance_limit` option to the `screenshot` method:

```ruby
test 'color threshold' do
  visit '/'
  screenshot 'index', color_distance_limit: 30
end
```

The difference is calculated as the eucledian distance.  You can also set this globally:

```ruby
Capybara::Screenshot::Diff.color_distance_limit = 42
```


### Allowed difference size

You can set set a threshold for the differing area size for the comparison
using the `area_size_limit` option to the `screenshot` method:

```ruby
test 'area threshold' do
  visit '/'
  screenshot 'index', area_size_limit: 17
end
```

The difference is calculated as `width * height`.  You can also set this globally:

```ruby
Capybara::Screenshot::Diff.area_size_limit = 42
```


## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake test` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`.

To release a new version, update the version number in `lib/capybara/screenshot/diff/version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/donv/capybara-screenshot-diff. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

