Completeness-Fu [![Build Status](https://secure.travis-ci.org/timrwilliams/completeness-fu.png?branch=master)](http://travis-ci.org/timrwilliams/completeness-fu)
===============

In short, completeness-fu for ActiveModel allows you cleanly define the way a model instance is scored for completeness, similar to LinkedIn user profiles.


When should I use it?
---------------------

When you want your objects/models/instances to be of a certain standard, if it be data presence, format or what values are allowed and not allowed,
before an object can be saved or updated then you use validations. But when you want to allow more information to be entered, thus relaxing some
of the validation rules and prompt the user to enrich the data, then completeness-fu is your cup of tea.

Take an events web site for example, you may want to import information from various sources which might have varied data quality.
If your validations are too strict then the information won't import, but if you relax your validations too much then you risk
having quantity but not quality. What you can do is relax the validations and add some quality checks which calculate a score
which is then used to determine if the event information is shown on the public site or is listed in the admin panel prompting
staff to enrich the data before it is made public.


What does it work with?
-----------------------

Completeness-Fu used to be for ActiveRecord only, but this has changed with the latest version with only ActiveModel needed as a gem dependency (and a little ActiveSupport). This means that the latest version works with the following ORMs :

  - ActiveRecord

  - Mongoid

  - Cassandra Object

The `define_completeness_scoring` method is mixed into the above mentioned  ORMs by default, but if you want to use Completeness-Fu in an ActiveModel model which isn't one of the above ORMs, make sure you include `ActiveModel::Naming`. If you want to cache the completeness score make sure you also include `ActiveModel::Validations` and `ActiveModel::Validations::Callbacks`.


How do I use it?
----------------

If you want your model to only be regarded as complete if it has a title, description, and picture, you can do the following:

    class Example < ActiveRecord::Base
      define_completeness_scoring do
        check :title,       lambda { |per| per.title.present? },        :high   # => defaults to 60
        check :description, lambda { |per| per.description.present? },  :medium # => defaults to 40
        check :main_image,  lambda { |per| per.main_image? },           :low    # => defaults to 20
      end
    end

Now, you can access several methods to find out how far along a model is : _passed\_checks_, _failed\_checks_, _completeness\_score_, _percent\_complete_.

    @example = Example.new

    @example.passed_checks.size # => 0
    @example.failed_checks.size # => 3

    @example.completeness_score # => 0
    @example.percent_complete   # => 0

    @example.title = "An Awesome Example"
    @example.passed_checks.size # => 1
    @example.failed_checks.size # => 2

    @example.completeness_score # => 60
    @example.percent_complete   # => 50


Options
-------

You can add the following to an initializer to set some defaults:

    CompletenessFu.common_weights = { :low => 30, :medium => 50, :high => 70 }

    CompletenessFu.default_weighting = :medium

`weights` and `default_weighting` can be changed per model, and you can use symbols for the checks instead of lambdas, thus allowing you to place check logic as methods in your class (public or private).

    define_completeness_scoring do
      weights :low => 30, :super_high => 60
      default_weighting :medium

      check :title,       lambda { |per| per.title.present? }, :super_high
      check :description, :description_present?
      check :main_image,  :main_image_or_pretty_picture?, :low
    end

And if you want to cache the score to a field so you can use it in database searches you just have to add the following:

    define_completeness_scoring do
      cache_score :absolute # the default is :relative

      check :title,       lambda { |per| per.title.present? }, :super_high
      check :description, :description_present?
    end

Requirements
------------

If you are not using one of the above ORMs, make sure `ActiveModel::Validations` and `ActiveModel::Validations::Callbacks` are included.


Passed checks, Failed checks and i18n
----------------------------------------

Both the _passed\_checks_ and _failed\_checks_ methods return an array of hashes with extended information about the check.
The check hash includes a translated title, description and extra information which is based on the name of the check.
The translation structure is as such:

    en:
      completeness_scoring:
        models:
          my_model_name:
            name_of_check:
              title: 'Title'
              description: 'The Check Description'
              extra: 'Extra Info'


Up and coming features
----------------------

- enhance caching so filter type can be changed and field to save score to can be customized
- ability to 'share' common lambdas
- better docs
- increase test coverage (error edge cases, ORM inclusion)


Change Log
----------

4 Sep 10

- added bundler for development and testing
- make the plugin ORM agnostic, only ActiveModel is required

15 Oct 09

- bug fix for failed checks when check is a symbol to a method
- bug fix for grading (percent complete needed to be rounded)
- change of CompletenessFu.default\_weightings to CompletenessFu.default\_weighting
- change of CompletenessFu.default\_grading to CompletenessFu.default\_gradings

29 Sep 09

- added a 'grading' method which returns either :poor, :low, :medium or :high based on its percentage complete
- added ability to customize the default translations namespace using CompletenessFu.default\_i18n\_namespace

28 Sep 09

- move the scoring check builder into its own class so that it works within a clean room

24 Sep 09

- options to save the score to a field (caching) - good for searching on
- define methods on the class to use in the checks



Contributors
------------

- Peter (pero-ict)
- Andrew Brown (omenking)
- Paul Campbell (paulca)
