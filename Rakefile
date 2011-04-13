namespace :sass do
  desc "Compile Sass into CSS"
  task :compile do
    %x{ sass --update --style compressed css }
  end
end

namespace :jekyll do
  desc "Generate site"
  task :generate => :'sass:compile' do
    %x{ jekyll }
  end
end

namespace :blog do
  desc "Starts a local server"
  task :server do
    sass = Process.fork do
      trap('INT') {}
      %x{ sass --style compressed --watch css:css > /dev/null }
    end
    jekyll = Process.fork do
      trap('INT') {}
      %x{ jekyll --server --auto > /dev/null }
    end

    run = true

    trap('INT') { run = false }

    while run
      sleep 0.5
    end

    Process.kill 'INT', sass, jekyll
  end

  desc "Publish blog to GitHub"
  task :publish => [ :'sass:compile', :'jekyll:generate' ] do
    # %x{ git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s - %C(yellow)%aN%Creset %Cgreen(%cr)%Creset' --abbrev-commit --date=relative github/master.. }

    %x{ git push github master }
  end
end
