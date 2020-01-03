# Explanation of how to get the app up and running:

[The following](https://github.com/BKSpurgeon/testing_in_sequel) gives you a very basic app which shows you how to get started :

(i) Sequel

(ii) Minitest (not minitest spec)

(iii) Rails, and 

(iv) instructions on how you can deploy it to a service like Heroku.


### Get the App up

* I have used the [sequel_rails gem](https://github.com/TalentBox/sequel-rails) to get started. Follow their instructions.


### Setting up a test with fixtures

* You may create a `test/sequel_test_case.rb` file which contains - and from which you can derive all your test classes. 

```ruby
DB = Sequel.postgres # replace with your database.

class SequelTestCase < ActiveSupport::TestCase # or you can inherit from Minitest::Test
  def run(*args, &block)
    DB.transaction(:rollback=>:always, :auto_savepoint=>true){super} # ensures your database is clean after tests are run.
  end
end
```

But if you are using the fixtures_dependency gem, then you can add the following in your `/test/test_helper.rb` file instead:

```ruby
ENV['RAILS_ENV'] ||= 'test'
require_relative '../config/environment'
require 'rails/test_help'

require 'fixture_dependencies/helper_methods'


class ActiveSupport::TestCase
  # Run tests in parallel with specified workers
  parallelize(workers: :number_of_processors)

  # Add more helper methods to be used by all tests here...
  include FixtureDependencies::HelperMethods

  FixtureDependencies.fixture_path = './test/fixtures'

  def run(*args, &block)
    Sequel::Model.db.transaction(:rollback=>:always){super}
  end
end

```

#### Adding a model / scaffold

* Create an Organisation scaffold: `rails g scaffold Organisation name`.

Don't forget to add a plugin so that it works just like rails:

```ruby
# app/models/organisation.rb
class Organisation < Sequel::Model
  Sequel::Model.plugin :active_model
end

```

The fixture I have created is an `organisation.yml` file located in `text/fixtures/organisation.yml`. 

```yml
one:
  name: MyString

two:
  name: MyString
```

### Adding a test

Create a basic test file: `test/models/organisation_test.rb`


```ruby
require 'test_helper'
require 'sequel_test_case'

require 'fixture_dependencies/test_unit/sequel'

class OrganisationTest < ActiveSupport::TestCase
  def test_organisation_name
    organisation = load(:organisation__one)
    assert_equal(organisation.name, "BadBoys")
  end
end
```

Go to your terminal and run `bin/rake` and the tests should run!

### Heroku instructions

Some of you may want it to work on Heroku. That's simple enough. First create the app in heroku (or on the heroku CLI).

* Go to `/config.ru` file (create one if it doesn't exist), and make sure it looks something like this:

```ruby
# This file is used by Rack-based servers to start the application.

require_relative 'config/environment'
require 'sequel'

Sequel.connect( ENV['DATABASE_URL'])

run Rails.application
```

* Secondly, you need to [provision a database with herou](https://devcenter.heroku.com/articles/heroku-postgresql#provisioning-heroku-postgres). Heroku will supply you with the relevant `DATABASE_URL` string. If you have the heroku CLI installed you can bang the following into your terminal:

`heroku addons:create heroku-postgresql:hobby-dev`

* Now you can run `git push heroku master`.
* Make sure you run an outstanding migrations: `heroku run rake db:migrate`.
* Then you should be able to and check out your site. You can check on the one I created [here](https://testingsequel.herokuapp.com/). 


### Demo app is publicly available:

Here it is: [https://github.com/BKSpurgeon/testing_in_sequel](https://github.com/BKSpurgeon/testing_in_sequel)


