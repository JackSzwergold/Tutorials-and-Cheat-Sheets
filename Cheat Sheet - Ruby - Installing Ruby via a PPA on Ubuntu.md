# Cheat Sheet - Ruby - Installing Ruby via a PPA on Ubuntu

By Jack Szwergold, September 20, 2015

#### Adding the Brightbox PPA to the system.

First install `python-software-properties` like this:

    sudo aptitude install python-software-properties

Next add the Brightbox PPA repository to the system like this:

    sudo add-apt-repository ppa:brightbox/ruby-ng

With that done, run `aptitude update` like this:

    sudo aptitude update

#### Install various versions of Ruby via the Brightbox PPA.

Install Ruby 1.8:

    sudo aptitude install ruby1.8 ruby1.8-dev

Install Ruby 1.9:

    sudo aptitude install ruby1.9 ruby1.9-dev

Install Ruby 2.0:

    sudo aptitude install ruby2.0 ruby2.0-dev

Install Ruby 2.1:

    sudo aptitude install ruby2.1 ruby2.1-dev

***

*Cheat Sheet - Ruby - Installing Ruby via a PPA on Ubuntu (c) by Jack Szwergold*

*This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC-BY-NC-SA-4.0).*