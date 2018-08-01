---
title: Install Docker and Triton on macOS
slug: install-docker-and-triton-on-macos
date_published: 2017-02-16T19:24:30.064Z
date_updated:   2017-02-16T19:24:30.063Z
---

By default, the Mac does not come with quite everything you need in order to install the Joyent Triton command-line tools, and Joyent's web site tends to skip some of the requirements. Here are the complete steps for getting up to speed with Docker and Triton CLI tools on a shiny new macOS machine.

# The components

1. Docker Mac
2. pkgsrc
3. triton-cli

# The steps

1. Install the Docker application from the Docker web site. This provides a native GUI management app, which in turn installs the required system components. [Download the Docker.dmg](https://www.docker.com/products/overview) file from the web site, open the image, and run the .pkg installer. You'll need to provide administrator credentials at first launch in order to install the system components.
2. Install the pkgsrc package manager. [Copy the snippet from the web site](https://pkgsrc.joyent.com/install-on-osx/), paste into your Terminal window, and voilà.
3. Update pkgsrc. Not strictly necessary, but it doesn't hurt to run `pkgin -y update` after a fresh installation.
4. Install node.js via pkgsrc: `sudo pkgin -y install nodejs`
5. Install the triton cli tools: `sudo npm install -g triton`

Now you're all set to build Docker images and manage them on Triton from your Mac.


