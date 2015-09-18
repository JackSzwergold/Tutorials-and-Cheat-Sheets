# Cheat Sheet - Mac OS X - HTOP

By Jack Szwergold, September 15, 2015

This tutorial explains how you can install `htop` directly from source code on Mac OS X. Tested on Mac OS X 10.9.5. Note that these versions of `htop` are somewhat out of date compared to the official Linux `htop` release. But honestly, the version used here—0.8.2.1 (AndyA) and 0.8.2.2 (stormerider)—still works fine for basic needs on Mac OS X.

***

#### A more up to date version of `htop` (0.8.2.7) with more meter options courtesy of “max-horvath.”

First clone the sourcecode from max-horvath’s `htop-osx` GitHub repository:

	git clone git@github.com:max-horvath/htop-osx.git
	
Now go into the repository directory:

	cd htop-osx
	
Run `libtoolize` and `automake -f` like this:

	libtoolize && automake -f

Then run `autogen.sh`:

	./autogen.sh
	
Run this `configure` command:

	./configure
	
Once the `configure` process completes, run `make`:

	make
	
Finally install it by running `sudo make install`:

	sudo make install

#### A nicer, forked version of `htop` (0.8.2.2) with more meter options courtesy of “stormerider.”

First clone the sourcecode from stormerider’s `htop-osx` GitHub repository:

	git clone git://github.com/stormerider/htop-osx.git
	
Now go into the repository directory:

	cd htop-osx
	
Set the `CFLAGS` like this:

	export CFLAGS="-m32"

Run `libtoolize` and `automake -f` like this:

	libtoolize && automake -f

Then run `autogen.sh`:

	./autogen.sh
	
Run this `configure` command:

	./configure
	
Once the `configure` process completes, run `make`:

	make
	
Finally install it by running `sudo make install`:

	sudo make install

#### A basic install of `htop` (0.8.2.1) courtesy of “AndyA.”

First clone the sourcecode from AndyA’s `htop-osx` GitHub repository:

	git clone git://github.com/AndyA/htop-osx.git
	
Now go into the repository directory:

	cd htop-osx
	
Checkout the `osx` branch of the repository like this:

	git checkout osx
	
Set the `CFLAGS` like this:

	export CFLAGS="-m32"

Run `libtoolize` and `automake -f` like this:

	libtoolize && automake -f

Then run `autogen.sh`:

	./autogen.sh
	
Run this `configure` command:

	./configure
	
Once the `configure` process completes, run `make`:

	make
	
Finally install it by running `sudo make install`:

	sudo make install

***

*Cheat Sheet - Mac OS X - HTOP (c) by Jack Szwergold*

*This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License (CC-BY-NC-SA-4.0).*