# ~/railtie/lib/rails/commands.rb
Rails::Server.new

    def initialize(*)
      super
      set_environment
    end


   def set_environment
      ENV["RAILS_ENV"] ||= options[:environment]
    end

# ~/railtie/lib/rails/commands.rb
require 'config/application'

#config/application.rb
require "rails"

%w(
  active_record
  action_controller
  action_mailer
  active_resource
  rails/test_unit
).each do |framework|
  begin
    require "#{framework}/railtie"
  rescue LoadError
  end
end

#~/active_record/railtie.rb
module ActiveRecord
  # = Active Record Railtie
  class Railtie < Rails::Railtie
    config.active_record = ActiveSupport::OrderedOptions.new

    config.app_generators.orm :active_record, :migration => true,
                                              :timestamps => true

    config.app_middleware.insert_after "::ActionDispatch::Callbacks",
      "ActiveRecord::QueryCache"

    config.app_middleware.insert_after "::ActionDispatch::Callbacks",
      "ActiveRecord::ConnectionAdapters::ConnectionManagement"

    rake_tasks do
      load "active_record/railties/databases.rake"
    end

    # When loading console, force ActiveRecord to be loaded to avoid cross
    # references when loading a constant for the first time.
    console do
      ActiveRecord::Base
    end

!SLIDE
    initializer "active_record.initialize_timezone" do
      ActiveSupport.on_load(:active_record) do
        self.time_zone_aware_attributes = true
        self.default_timezone = :utc
      end
    end

!SLIDE
    initializer "active_record.logger" do
      ActiveSupport.on_load(:active_record) { self.logger ||= ::Rails.logger }
    end

!SLIDE
    initializer "active_record.set_configs" do |app|
      ActiveSupport.on_load(:active_record) do
        app.config.active_record.each do |k,v|
          send "#{k}=", v
        end
      end
    end

!SLIDE
    # This sets the database configuration from Configuration#database_configuration
    # and then establishes the connection.
    initializer "active_record.initialize_database" do |app|
      ActiveSupport.on_load(:active_record) do
        self.configurations = app.config.database_configuration
        establish_connection
      end
    end

!SLIDE
    # Expose database runtime to controller for logging.
    initializer "active_record.log_runtime" do |app|
      require "active_record/railties/controller_runtime"
      ActiveSupport.on_load(:action_controller) do
        include ActiveRecord::Railties::ControllerRuntime
      end
    end

!SLIDE
    initializer "active_record.set_dispatch_hooks", :before => :set_clear_dependencies_hook do |app|
      unless app.config.cache_classes
        ActiveSupport.on_load(:active_record) do
          ActionDispatch::Callbacks.after do
            ActiveRecord::Base.clear_reloadable_connections!
          end
        end
      end
    end

!SLIDE
    config.after_initialize do
      ActiveSupport.on_load(:active_record) do
        instantiate_observers

        ActionDispatch::Callbacks.to_prepare(:activerecord_instantiate_observers) do
          ActiveRecord::Base.instantiate_observers
        end
      end
    end
  end
end


!SLIDE
    initializer "action_controller.logger" do
      ActiveSupport.on_load(:action_controller) { self.logger ||= Rails.logger }
    end

    initializer "action_controller.initialize_framework_caches" do
      ActiveSupport.on_load(:action_controller) { self.cache_store ||= RAILS_CACHE }
    end

    initializer "action_controller.set_configs" do |app|
      paths   = app.config.paths
      options = app.config.action_controller

      options.assets_dir           ||= paths["public"].first
      options.javascripts_dir      ||= paths["public/javascripts"].first
      options.stylesheets_dir      ||= paths["public/stylesheets"].first
      options.page_cache_directory ||= paths["public"].first

      # make sure readers methods get compiled
      options.asset_path           ||= nil
      options.asset_host           ||= nil

!SLIDE
      ActiveSupport.on_load(:action_controller) do
        include app.routes.mounted_helpers
        extend ::AbstractController::Railties::RoutesHelpers.with(app.routes)
        extend ::ActionController::Railties::Paths.with(app)
        options.each { |k,v| send("#{k}=", v) }
      end
    end

!SLIDE
    initializer "action_controller.compile_config_methods" do
      ActiveSupport.on_load(:action_controller) do
        config.compile_methods! if config.respond_to?(:compile_methods!)
      end
    end
  end
end

!SLIDE
server.start


