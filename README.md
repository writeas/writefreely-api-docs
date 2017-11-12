# Write.as API Docs

View the [documentation](https://developer.write.as/docs/api/).

## Getting Set Up

Initialize and start Slate. You can either do this locally, or with Vagrant:

```shell
# Upgrade Ruby to 2.0 potentially
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt-get install ruby2.2

# Make sure json and other things install
sudo apt-get install ruby2.2-dev

# Install bundler
sudo gem install bundler

# either run this to run locally
bundle install
bundle exec middleman server

# OR run this to run with vagrant
vagrant up
```

View at http://localhost:4567

## Run
`bundle exec middleman server`

## Deploy
`./deploy.sh`

## Update
```bash
git remote set upstream git@github.com:lord/slate.git
git fetch upstream
git merge upstream/master
```
