# ZeroAuthorization

Functionality to add authorization on Rails model's write operations plus any other set of defined methods.

## Installation

Add this line to your application's Gemfile:

    gem 'zero_authorization'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install zero_authorization

## Usage

1. Specify what any specific role of current root entity(logged_user) can do/can't do in roles_n_privileges.yml

        :role_role_name_one:
          :can_do:
            :account:
              - create
              - save
              - update
            :User:
              - :create
              - :save
              - :update
            :model_crud_history: :nothing
            :Permission: :anything
          :cant_do:
            :Project:
            - :destroy
            :model_crud_history: :anything
        :role_role_name_two:
          :can_do:
            :Project:
              - create
              - save: is_authorized?
              - :update: is_authorized?
              - :destroy: is_authorized?
            :user:
              create: :create_authorized?
              save: :update_authorized?
              update: :update_authorized?
        :role_role_name_three:
          :can_do:
            - :Project
            - :Organization
          :cant_do:
            - :User

2. Restrict models for activity and let rule-set(s) via zero-authorization take control of activity

    1 Add initializer file with content (if all existing models needs to have restrictions. Easy way out).

        # Make a file restrict_models.rb in Rails.root/config/initializers
        Rails.application.eager_load!
        ActiveRecord::Base.descendants.each do |descendant|
          descendant.send(:include, ZeroAuthorization::Engine)
        end

    2 To hand pick some models for restriction

        class ModelName < ActiveRecord::Base
          include ZeroAuthorization::Engine
        end

4. (Re-)boot application.

## Rules for rule-sets execution (Precedence: from top to bottom)

    1. Rule 00: If no rule-sets are available for 'can do' and 'cant do' then is authorized true '(with warning message)'.
    2. Rule 01: If role can't do 'anything' or can do 'nothing' then is authorized false.
    3. Rule 02: If role can't do 'nothing' or can do 'anything' then is authorized true.
    4. Rule 03: If role can't do 'specified' method and given 'evaluate' method returns true then is authorized false.
    5. Rule 04: If role can't do 'specified' method and given 'evaluate' method returns false then is authorized true.
    6. Rule 05: If role can   do 'specified' method and given 'evaluate' method returns true then is authorized true.
    7. Rule 06: If role can   do 'specified' method and given 'evaluate' method returns false then is authorized false.
    8. Rule 07: If role can't do 'specified' method then is authorized false.
    9. Rule 08: If role can   do 'specified' method then is authorized true.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
