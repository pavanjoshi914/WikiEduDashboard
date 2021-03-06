## Upgrading Ruby ##

### Prepare the code
Update the code where the specific Ruby version is specified:
* `Gemfile`
* `.travis.yml`
* `docs/setup.md`

Make sure the tests pass on travis with the new Ruby.

### Prepare for deployment

* Stop updates: `cap production sake task=batch:pause`
* Stop Sidekiq: `cap production deploy:sidekiq:stop`

### Prepare Ruby and Passenger on the server

On the server where the dashboard is already running, in `/var/www/dashboard/current`:
* `rvm get head` (to ensure the new ruby is available)
* `rvm install ruby-x.x.x`
* `gem install bundler`
* `gem install passenger`
* `cd ~`
* `rvm --default use x.x.x`
* `rvmsudo passenger-install-apache2-module`

Passenger should now be ready. Copy the output for updating the apache config.
It will be something like this:
```
LoadModule passenger_module /usr/local/rvm/gems/ruby-2.7.1/gems/passenger-5.1.12/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
  PassengerRoot /usr/local/rvm/gems/ruby-2.7.1/gems/passenger-5.1.12
  PassengerDefaultRuby /usr/local/rvm/gems/ruby-2.7.1/wrappers/ruby
</IfModule>
```

### Deploy
Deploy as usual with the upgraded Ruby version. This will break the app, as it will still be using the older version of Passenger.

Change the Apache configuration to use it as soon a version of the dashboard gets deployed.
* `sudo nano /etc/apache2/apache2.conf`
* Change the PassengerDefaultRuby path, the PassengerRoot path, and the passenger_module path in the apache configuration, per the output of the passenger installation command. (Leave PassengerDefaultUser and PassengerInstanceRegisteryDir as they are.)
* `sudo service apache2 restart`

### Post-deployment

* Restart updates: `cap production sake task=batch:resume`
* Sidekiq should have restarted during deployment

### Troubleshooting

If passenger is failing to restart:
* Make sure rvm is available and defaults to the new version for the deploy user. This may differ between a login (ssh) session and a non-login Capistrano session. Check `/etc/bash.bashrc` and `~/.bashrc`, and make sure rvm is being sourced properly.
* Make sure Capistrano is trying to use the correct version of passenger. Add debugging commands, like `passenger -v`, to `config/deploy/production.rb`.
* Make sure Capistrano is using the new ruby version and the corresponding passenger executables.
