!SLIDE
    @@@ruby
    $rails server

!SLIDE smaller
    @@@ruby
    #~script/rails

    #!/usr/bin/env ruby
    # This command will automatically be run when you run "rails" with Rails 3 gems installed from the root of your application.

    APP_PATH = File.expand_path('../../config/application',  __FILE__)
    require File.expand_path('../../config/boot',  __FILE__)
    require 'rails/commands'

!SLIDE smaller
    @@@ruby
    #~/rails/commands.rb

    when 'server'
      require 'rails/commands/server'
      Rails::Server.new.tap { |server|
        # We need to require application after the server sets environment,
        # otherwise the --environment option given to the server won't propagate.
        require APP_PATH
        Dir.chdir(Rails.application.root)
        server.start
      }

!SLIDE smaller
    @@@ruby
    #~/rails/commands/server.rb
    module Rails
      class Server < ::Rack::Server
        class Options
          def parse!(args)
          end
        end
        def initialize(*)
          super
          set_environment
        end
      end
    end


!SLIDE smaller
    @@@ ruby
    # ~/railtie/lib/rails/commands.rb
    Rails::Server.new

    def initialize(*)
      super
      set_environment
    end

    def set_environment
      ENV["RAILS_ENV"] ||= options[:environment]
    end

!SLIDE
    @@@ruby
    # ~/railtie/lib/rails/commands.rb
    require 'config/application'

!SLIDE
    @@@ruby
    #config/application.rb
    require "boot"
    require "rails"

!SLIDE
    @@@ruby
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

!SLIDE small
    @@@ruby
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

!SLIDE smaller
    @@@ruby
    initializer "active_record.initialize_timezone" do
      ActiveSupport.on_load(:active_record) do
        self.time_zone_aware_attributes = true
        self.default_timezone = :utc
      end
    end

!SLIDE smaller
    @@@ ruby
    initializer "active_record.logger" do
      ActiveSupport.on_load(:active_record) { self.logger ||= ::Rails.logger }
    end

!SLIDE smaller
    @@@ ruby
    initializer "active_record.set_configs" do |app|
      ActiveSupport.on_load(:active_record) do
        app.config.active_record.each do |k,v|
          send "#{k}=", v
        end
      end
    end

!SLIDE smaller
    @@@ruby
    # This sets the database configuration from Configuration#database_configuration
    # and then establishes the connection.
    initializer "active_record.initialize_database" do |app|
      ActiveSupport.on_load(:active_record) do
        self.configurations = app.config.database_configuration
        establish_connection
      end
    end

!SLIDE smaller
    @@@ruby
    # Expose database runtime to controller for logging.
    initializer "active_record.log_runtime" do |app|
      require "active_record/railties/controller_runtime"
      ActiveSupport.on_load(:action_controller) do
        include ActiveRecord::Railties::ControllerRuntime
      end
    end

!SLIDE smaller
    @@@ruby
    initializer "active_record.set_dispatch_hooks", :before => :set_clear_dependencies_hook do |app|
      unless app.config.cache_classes
        ActiveSupport.on_load(:active_record) do
          ActionDispatch::Callbacks.after do
            ActiveRecord::Base.clear_reloadable_connections!
          end
        end
      end
    end

!SLIDE smaller
    @@@ruby
    config.after_initialize do
      ActiveSupport.on_load(:active_record) do
        instantiate_observers

        ActionDispatch::Callbacks.to_prepare(:activerecord_instantiate_observers) do
          ActiveRecord::Base.instantiate_observers
        end
      end
    end


!SLIDE smaller
    @@@ruby
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

!SLIDE smaller
    @@@ruby
    ActiveSupport.on_load(:action_controller) do
      include app.routes.mounted_helpers
      extend ::AbstractController::Railties::RoutesHelpers.with(app.routes)
      extend ::ActionController::Railties::Paths.with(app)
      options.each { |k,v| send("#{k}=", v) }
    end

!SLIDE smaller
    @@@ruby
    initializer "action_controller.compile_config_methods" do
      ActiveSupport.on_load(:action_controller) do
        config.compile_methods! if config.respond_to?(:compile_methods!)
      end
    end




!SLIDE smaller

    @@@ruby
    Bundler.require(:default, Rails.env) if defined?(Bundler)

!SLIDE smaller
    @@@ ruby
    module AdminData
      class Engine < Rails::Engine
      end
    end

!SLIDE
    @@@ruby
    server.start

!SLIDE smaller
    @@@ruby
    def start
      puts "=> Booting #{ActiveSupport::Inflector.demodulize(server)}"
      puts "=> Rails #{Rails.version} application starting in #{Rails.env} on http://#{options[:Host]}:#{options[:Port]}"
      puts "=> Call with -d to detach" unless options[:daemonize]
      trap(:INT) { exit }
      puts "=> Ctrl-C to shutdown server" unless options[:daemonize]

      #Create required tmp directories if not found
      %w(cache pids sessions sockets).each do |dir_to_make|
        FileUtils.mkdir_p(Rails.root.join('tmp', dir_to_make))
      end

      super
    end

!SLIDE smaller
    @@@ruby
    #config.ru
    require ::File.expand_path('../config/environment',  __FILE__)
    run THowRailsBoots::Application

!SLIDE smaller
    @@@ruby
    #environment.rb
    # Load the rails application
    require File.expand_path('../application', __FILE__)

    # Initialize the rails application
    THowRailsBoots::Application.initialize!


!SLIDE smaller
    @@@ruby
    # rails/application.rb
    class Application < Engine
      class << self
        def inherited(base)
          raise "You cannot have more than one Rails::Application" if Rails.application
          super
          Rails.application = base.instance
          Rails.application.add_lib_to_load_path!
          ActiveSupport.run_load_hooks(:before_configuration,
                                       base.instance)
        end
      end

!SLIDE
#ActiveSupport.run_load_hooks#

!SLIDE bullets incremental smaller
##ActiveSupport hooks invoked when Rails boots.##

* before_configuration *
* before_initialize *
* before_eager_load
* action_controller
* action_view *
* active_record *
* after_initialize *

!SLIDE smaller
##eager_loading happens only in production mode.##

    @@@ruby
    #~/rails/application/finisher.rb

    initializer :eager_load! do
      if config.cache_classes && !$rails_rake_task
        ActiveSupport.run_load_hooks(:before_eager_load, self)
        eager_load!
      end
    end

!SLIDE smaller
    @@@ruby
    #~rails/engine/configuration.rb
    def paths
      @paths ||= begin
        paths = Rails::Paths::Root.new(@root)
        paths.add "app",                 :eager_load => true, :glob => "*"
        paths.add "app/controllers",     :eager_load => true
        paths.add "app/helpers",         :eager_load => true
        paths.add "app/models",          :eager_load => true
        paths.add "app/mailers",         :eager_load => true
        paths.add "app/views"
        paths.add "lib",                 :load_path => true
        paths.add "lib/tasks",           :glob => "**/*.rake"
        paths.add "config"
        paths.add "config/environments", :glob => "#{Rails.env}.rb"
        paths.add "config/initializers", :glob => "**/*.rb"
        paths.add "config/locales",      :glob => "*.{rb,yml}"
        paths.add "config/routes",       :with => "config/routes.rb"
        paths.add "db"
        paths.add "db/migrate"
        paths.add "db/seeds",            :with => "db/seeds.rb"
        paths.add "public"
        paths.add "public/javascripts"
        paths.add "public/stylesheets"
        paths.add "vendor",              :load_path => true
        paths.add "vendor/plugins"
        paths
      end
    end

!SLIDE smaller
    @@@ruby
    #environment.rb
    # Initialize the rails application
    THowRailsBoots::Application.initialize!

!SLIDE smaller
    @@@ruby
    def run_initializers(*args)
      return if instance_variable_defined?(:@ran)
      initializers.tsort.each do |initializer|
        initializer.run(*args)
      end
      @ran = true
    end

    def initializers
      Bootstrap.initializers_for(self) +
      super +
      Finisher.initializers_for(self)
    end

!SLIDE smaller
    @@@ruby
    #~/rails/application/bootstrap.rb
    module Bootstrap
      initializer :load_environment_hook do .. end
      initializer :load_active_support do .. end

      initializer :preload_frameworks do
        ActiveSupport::Autoload.eager_autoload! if config.preload_frameworks
      end

      # Initialize the logger early in the stack in case we need to log some deprecation.
      initializer :initialize_logger do .. end

      # Initialize cache early in the stack so railties can make use of it.
      initializer :initialize_cache do ..end

      initializer :set_clear_dependencies_hook do .. end

      initializer :initialize_dependency_mechanism do .. end

      initializer :bootstrap_hook do |app| .. end

!SLIDE smaller
    @@@ruby
    #~/rails/application/finisher.rb
    module Finisher

      initializer :add_generator_templates do .. end

      initializer :ensure_autoload_once_paths_as_subset do .. end

      initializer :add_to_prepare_blocks do .. end

      initializer :add_builtin_route do |app| .. end

      initializer :build_middleware_stack do .. end

      initializer :eager_load! do .. end

      initializer :finisher_hook do .. end

!SLIDE smaller
    @@@ruby
    initializer :add_to_prepare_blocks do
      config.to_prepare_blocks.each do |block|
        ActionDispatch::Callbacks.to_prepare(&block)
      end
    end

!SLIDE smaller

#to_prepare is a hook that is called every single time before a request in development mode. Only once in production mode#

!SLIDE smaller
    @@@ruby
    puts 'last initializer done'

    ActionDispatch::Callbacks.to_prepare do
      puts Time.now.to_s(:db)
    end

!SLIDE smaller
    @@@ruby
    initializer :add_builtin_route do |app|
      if Rails.env.development?
        app.routes.append do
          match '/rails/info/properties' => "rails/info#properties"
        end
      end
    end

#http://localhost:3000/rails/info/properties#

!SLIDE smaller
    Ruby version	1.8.7 (i686-darwin10.4.0)
    RubyGems version	1.3.7
    Rack version	1.2
    Rails version	3.1.0.beta
    Active Record version	3.1.0.beta
    Action Pack version	3.1.0.beta
    Active Resource version	3.1.0.beta
    Action Mailer version	3.1.0.beta
    Active Support version	3.1.0.beta
    Middleware
    ActionDispatch::Static
    Rack::Lock
    ActiveSupport::Cache::Strategy::LocalCache
    Rack::Runtime
    Rails::Rack::Logger
    ActionDispatch::ShowExceptions
    ActionDispatch::RemoteIp
    Rack::Sendfile
    ActionDispatch::Callbacks
    ActiveRecord::ConnectionAdapters::ConnectionManagement
    ActiveRecord::QueryCache
    ActionDispatch::Cookies
    ActionDispatch::Session::CookieStore
    ActionDispatch::Flash
    ActionDispatch::ParamsParser
    Rack::MethodOverride
    ActionDispatch::Head
    Rack::ConditionalGet
    Rack::ETag
    ActionDispatch::BestStandardsSupport
    Application root	/Users/nsingh/dev/rails_lighthouse/tickets/t_how_rails_boots
    Environment	development
    Database adapter	sqlite3
    Database schema version	20101207012417

!SLIDE smaller
    @@@ruby
    initializer :build_middleware_stack do
      build_middleware_stack
    end

!SLIDE smaller
    @@@ruby
    middleware.use
    ::Rack::Cache, rack_cache  if rack_cache
    ::ActionDispatch::Static, config.static_asset_paths if config.serve_static_assets
    ::Rack::Lock unless config.allow_concurrency
    ::Rack::Runtime
    ::Rails::Rack::Logger
    ::ActionDispatch::ShowExceptions, config.consider_all_requests_local if config.action_dispatch.show_exceptions
    ::ActionDispatch::RemoteIp, config.action_dispatch.ip_spoofing_check, config.action_dispatch.trusted_proxies
    ::Rack::Sendfile, config.action_dispatch.x_sendfile_header
    ::ActionDispatch::Callbacks, !config.cache_classes
    ::ActionDispatch::Cookies
    ::ActionDispatch::Flash
    ::ActionDispatch::ParamsParser
    ::Rack::MethodOverride
    ::ActionDispatch::Head
    ::Rack::ConditionalGet
    ::Rack::ETag, "no-cache"
    ::ActionDispatch::BestStandardsSupport, config.action_dispatch.best_standards_support if config.action_dispatch.best_standards_support

!SLIDE smaller
    @@@ruby
    initializer :eager_load! do
      if config.cache_classes && !$rails_rake_task
        ActiveSupport.run_load_hooks(:before_eager_load, self)
        eager_load!
      end
    end

!SLIDE smaller
    @@@ruby
    initializer :finisher_hook do
      ActiveSupport.run_load_hooks(:after_initialize, self)
    end

