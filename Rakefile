# Dhruv Sahni <dsahni@chef.io>
# Version 0.1.3

task :default => 'help'
CKBK_NAME=`cat metadata.rb | grep '^name' | tr -s ' ' | cut -d ' ' -f 2 | tr -d "'"`

desc "Prints this help display"
task :help do
  sh 'chef exec rake -T'
end

desc "Verify Pipeline Stage (Syntax + Lint tests)"
task :verify => [:check_version, 'test:syntax', 'test:lint', 'test:unit']

desc "Build Pipeline Stage (Integration tests)"
task :build => 'test:integration'

desc "Deliver Pipeline Stage (Upload to Supermarket + Chef Server)"
task :deliver => [:supermarket_share, :berks_upload]

namespace :test do

  desc "Runs all syntax, lint, unit and integration tests on a cookbook"
  task :syntax do
    sh "chef exec knife cookbook test -o ../ #{CKBK_NAME}"
  end

  desc "Runs foodcritic checks, failing on checks labelled 'correctness'"
  task :foodcritic do
    sh 'chef exec foodcritic . -f correctness'
  end

  desc "Runs cookstyle checks"
  task :cookstyle do
    sh 'chef exec cookstyle'
  end

  desc "Runs lint tests (foodcritic + cookstyle)"
  task :lint => [:foodcritic, :cookstyle]

  desc "Runs chefspec tests"
  task :unit do
    sh 'chef exec rspec -c'
  end

  desc "Runs Test Kitchen + ServerSpec tests"
  task :integration do
    sh 'chef exec kitchen test -d always centos-7'
  end
end

desc "Performs a 'berks upload' (Warning! Expects a berks config.json)"
task :berks_upload do
  sh 'if [ -f Berksfile.lock ];then chef exec berks update; else chef exec berks install; fi'
  sh 'berks upload -c ~/.chef/berks.json'
end

desc "Shares cookbook on to private Supermarket (Warning! Expects a supermarket_url in knife.rb)"
task :supermarket_share do
  sh "chef exec knife supermarket share -o ../ #{CKBK_NAME}"
end

desc "Checks if cookbook version has been bumped compared to a Chef Server (Warning! Expects a knife.rb)"
task :check_version do
  sh "chef exec knife spork check -o ../ --fail #{CKBK_NAME}"
end

desc "Validates all json files are correct using jsonlint"
task :validate_json do
  sh "for file in $(find . -name '*.json' -exec echo {} \\;); do  chef exec jsonlint $file; done"
end

desc "Uploads environments, roles and data_bags to the Chef Server (Warning! Expects a knife.rb)"
task :chef_repo_upload do
  sh "chef exec knife upload  --chef-repo-path . data_bags environments roles"
end
