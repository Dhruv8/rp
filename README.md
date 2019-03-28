Steps to create an engine :

1. Generate a new engine :
      rails plugin new ENGINE_NAME --dummy-path=spec/dummy --skip-test-unit --mountable
2. Edit gemspec to add the homepage and summary.
3. Run bundle install.
4. If you are genetating the resources to be used by the engine through scaffold, do the following configuration in  lib/myengine/engine.rb :
      config.generators do |g|
        g.test_framework :rspec, :fixture => false
        g.fixture_replacement :factory_girl, :dir => 'spec/factories'
        g.template_engine :haml
      end

5. Add the following configuration in lib/myengine/engine.rb file to let all of your engine’s migrations automatically get  run in the wrapper Rails app :
      initializer :append_migrations do |app|
        unless app.root.to_s.match "#{root}/"
          config.paths['db/migrate'].expanded.each do |expanded_path|
            app.config.paths['db/migrate'] << expanded_path
          end
        end
      end
6. To add any config variable (which needs to be set in the hosting Rails app), do the following :
      6.1. Add the following in lib/myengine.rb :
                mattr_accessor :my_config_var

                # this function maps the vars from your app into your engine
                def self.setup(&block)
                  yield self
                end                
      The config variable my_config_var can be accessed as Rp.my_config_var in the engine.
7. Build gem from the engine :
      gem build GEMSPEC_FILE
8. Push the gem to rubygems :
      gem push GEM
Steps to use the engine in a Rails app :

1. Add the gem in Gemfile.
2. Run bundle install.
3. Run rake db:migrate to run the migrations of the engine.
4. Mount the engine in routes.rb :
      mount MyEngine::Engine, at: '/myengine'
5. If the engine expects any config variables, then do the following to set them : 
   5.1. Create a file config/initializers/myengine.rb in the hosting Rails app and add the following in it :
          Rp.my_config_var = <value>
