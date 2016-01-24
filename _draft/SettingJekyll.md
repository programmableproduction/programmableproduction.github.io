
1. Installing Ruby
    sudo pacman -S ruby

2. Installing Bundler
    gem install bundler

3. Update your path to include ~/.gem/ruby/2.3.0/bin
    Open the file ~/.profile
    write in PATH=$PATH:~/gem/ruby/2.3.0/bin

3. Create a Gemfile with
    source 'https://rubygems.org'
    gem 'github-pages'

5. Run 
    bundle install --path ~/.gem

6. run jekyll new .

7. run jenkyll
    bundle exec jekyll serve




