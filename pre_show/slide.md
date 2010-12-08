
!SLIDE bullets smaller incremental
*    $ rails new 2a
*    Invalid application name 2a. Please give a name which does not start with numbers.

!SLIDE smaller
    @@@ruby
    #~/config/environment.rb

    # Load the rails application
    require File.expand_path('../application', __FILE__)

    # Initialize the rails application
    THowRailsBoots::Application.initialize!

!SLIDE smaller
    @@@ruby
    #~/config/application.rb

    require File.expand_path('../boot', __FILE__)
    require 'rails/all'
    module THowRailsBoots
      class Application < Rails::Application
        config.action_view.javascript_expansions[:defaults] = %w()
        config.encoding = "utf-8"
        config.filter_parameters += [:password]
      end
    end
