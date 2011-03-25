namespace :sass do
  desc "Compile Sass into CSS"
  task :compile do
    %x{ sass --update sass }
  end
end

namespace :jekyll do
  desc "Generate site"
  task :generate => :'sass:compile' do
    require 'jekyll'

    options = Jekyll.configuration({})
    site = Jekyll::Site.new(options)
    site.process
  end

  desc "Starts a local server"
  task :server do
    %x{ sass --watch sass:css & }
    %x{ jekyll --server --auto }
  end
end
