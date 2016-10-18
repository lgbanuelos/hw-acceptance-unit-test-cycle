Acceptance-Unit Test Cycle
===

In this assignment you will use a combination of Acceptance and Units tests with the Cucumber and RSpec tools to add a "find movies with same director" feature to RottenPotatoes.


Learning Goals
--------------
After you complete this assignment, you should be able to:
* Create and run simple Cucumber scenarios to test a new feature
* Use RSpec to create unit tests that drive the creation of app code that lets the Cucumber scenario pass
* Understand where to modify a Rails app to implement the various parts of a new feature, since a new feature often touches the database schema, model(s), view(s), and controller(s)


Introduction and Setup
----
To get the initial RottenPotatoes code please clone this repo to your local machine. To that end, execute the following command in your terminal:

```sh
git clone https://github.com/lgbanuelos/hw-acceptance-unit-test-cycle
```

For this homework, you will need to submit most of your application code, including the configuration files. Given that the autograder is running on a Linux-based computer, with different versions of the Ruby, we need to jungle a little bit with all the configuration files. In the following subsections, I will describe the setup procedure for Windows and Unix-based OS. Please follow carefully the instructions.

## WINDOWS

The repository contains the file `Gemfile.windows`, which should be safe to use in your Windows box. Note that I am assuming, as for our lecture/practical sessions, you are using `postgresql`. For the same reason, you will find the file `config/database.yml.postgres` with the corresponding configuration. To complete your setup, in a powershell session, run the commands:

```sh
cp Gemfile.windows Gemfile
cp config/database.yml.postgres config/database.yml
rake db:setup
```

The above should be enough for you.

## UNIX-based OSs (Linux & Mac OS X)

The repository contains the file `Gemfile.unix`. Henceforth, you can run in your terminal the following command:

```sh
cp Gemfile.windows Gemfile
```

I heard that some students are also using `postgresql` as database for Linux. If that is the case, then you can run the following command:

```sh
cp config/database.yml.postgres config/database.yml
rake db:setup
```

If you are using `sqlite3`, you can use the content of `config/database.yml.sqlite3` instead. Run the following command:

```sh
cp config/database.yml.sqlite3 config/database.yml
```

## Completing the setup

Once you have the clone of the repo:

1) Change into the rottenpotatoes directory: `cd hw-acceptance-unit-test-cycle/rottenpotatoes`  
2) Run `bundle install --without production` to make sure all gems are properly installed.    
3) Run `bundle exec rake db:migrate` to apply database migrations.    
4) Run these commands to set up the Cucumber directories (under features/) and RSpec directories (under spec/) if they don't already exist, allowing overwrite of any existing files:

```shell
rails generate cucumber:install capybara
rails generate cucumber_rails_training_wheels:install
rails generate rspec:install
```

5) Create a new file called `rspec.rb` in features/support with the following contents:

```
require 'rspec/core'

RSpec.configure do |config|
  config.mock_with :rspec do |c|
    c.syntax = [:should, :expect]
  end
  config.expect_with :rspec do |c|
    c.syntax = [:should, :expect]
  end
end
```

This prevents RSpec from issuing DEPRECATION warnings when it encounters deprecated syntax in `features/step_definitions/web_steps`.

6) You can double-check if everything was installed by running the tasks `rspec` and `cucumber`. (You will most likely need to use `bundle exec rspec` and `bundle exec cucumber` instead of the plain commands, because at this time you would have multiple versions of cucumber and rspec installed into your computer).

Since presumably you have no features or specs yet, both tasks should execute correctly reporting that there are zero tests to run. Depending on your version of rspec, it may also display a message stating that it was not able to find any _spec.rb files.

**Part 1: add a Director field to Movies**

Create and apply a migration that adds the Director field to the movies table. 
The director field should be a string containing the name of the movie’s director. 
HINT: use the [`add_column` method of `ActiveRecord::Migration`](http://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements/add_column) to do this. 

Remember to add `:director` to the list of movie attributes in the `def movie_params` method in `movies_controller.rb`.

Remember that once the migration is applied, you also have to do `rake db:test:prepare` 
to load the new post-migration schema into the test database!

**Part 2: use Acceptance and Unit tests to get new scenarios passing**

We've provided [three Cucumber scenarios](http://pastebin.com/L6FYWyV7) to 
drive creation of the happy path of Search for Movies by Director.
The first lets you add director info to an existing movie, 
and doesn't require creating any new views or controller actions 
(but does require modifying existing views, and will require creating a new step definition and possibly adding a line
or two to `features/support/paths.rb`).

The second lets you click a new link on a movie details page "Find Movies With Same Director", 
and shows all movies that share the same director as the displayed movie.  
For this you'll have to modify the existing Show Movie view, and you'll have to add a route, 
view and controller method for Find With Same Director.  

The third handles the sad path, when the current movie has no director info but we try 
to do "Find with same director" anyway.

Going one Cucumber step at a time, use RSpec to create the appropriate
controller and model specs to drive the creation of the new controller
and model methods.  At the least, you will need to write tests to drive
the creation of: 

+ a RESTful route for Find Similar Movies 
(HINT: use the 'match' syntax for routes as suggested in "Non-Resource-Based Routes" 
in Section 4.1 of ESaaS). You can also use the key :as to specify a name to generate helpers (i.e. search_directors_path) http://guides.rubyonrails.org/routing.html Note: you probably won’t test this directly in rspec, but a line in Cucumber or rspec will fail if the route is not correct.

+ a controller method to receive the click
on "Find With Same Director", and grab the id (for example) of the movie
that is the subject of the match (i.e. the one we're trying to find
movies similar to) 

+ a model method in the Movie model to find movies
whose director matches that of the current movie. Note: This implies that you should write at least 2 specs for your controller: 1) When the specified movie has a director, it should...  2) When the specified movie has no director, it should ... and 2 for your model: 1) it should find movies by the same director and 2) it should not find movies by different directors.

It's up to you to
decide whether you want to handle the sad path of "no director" in the
controller method or in the model method, but you must provide a test
for whichever one you do. Remember to include the line 
`require 'rails_helper'` at the top of your *_spec.rb files.

We want you to report your code coverage as well.

Add `gem 'simplecov', :require => false` to the test group of your gemfile, then run `bundle install --without production`.

Next, add the following lines to the TOP of spec/rails_helper.rb and features/support/env.rb:

```ruby
require 'simplecov'
SimpleCov.start 'rails'
```

Now when you run `rspec` or `cucumber`, SimpleCov will generate a report in a directory named
`coverage/`. Since both RSpec and Cucumber are so widely used, SimpleCov
can intelligently merge the results, so running the tests for Rspec does
not overwrite the coverage results from SimpleCov and vice versa.

To see the results, open /coverage/index.html. You will see the code, but click the Run button at the top. This will spin up a web server with a link in the console you can click to see your coverage report.

Improve your test coverage by adding unit tests for untested or undertested code. Specifically, you can write unit tests for the `index`, `update`, `destroy`, and `create` controller methods.

**Submission:**


IMPORTANT: I must remind you that we replaced some configuration files. Therefore, WE NEED TO RESTORE THE FILES TO KEEP THE AUTOGRADER HAPPY (we don't want to pollute the autograder computers with additional versions of gems). Don't forget to copy the current content of your `Gemfile` somewhere, because we will overwrite its content before the submission. Now, run the following commands:

```sh
cp Gemfile.original Gemfile
cp config/database.yml.original config/database.yml
```

The autograder is expecting from us a zip file. Such zip file must contain the following files and directories of your app:

* app/
* config/
* db/migrate
* features/
* spec/
* Gemfile

If you modified any other files, please include them too. If you are on a *nix based system, navigate to the root directory for this assignment and run

```sh
$ cd ..
$ zip -r hw5.zip rottenpotatoes/app/ rottenpotatoes/config/ rottenpotatoes/db/migrate rottenpotatoes/features/ rottenpotatoes/spec/ rottenpotatoes/Gemfile
```

This will create the file hw5.zip, which you will submit.

If you are using Windows, I propose you to install the GNU Zip application (which is compatible with the command above). The GNU Zip application can be downloaded from: http://gnuwin32.sourceforge.net/packages/zip.htm

IMPORTANT NOTE: Your submission must be zipped inside a rottenpotatoes/ folder so that it looks like so:

```
$ tree
.
└── rottenpotatoes
    ├── Gemfile
    ├── Gemfile.lock
    ├── app
    ...
```
