# SCR - Community Repositories 

## Package Requests

If you are not familiar with the infrastructure and how to make a Pull Request to get a package inside the community repositories, open a bug request https://bugs.sabayon.org/enter_bug.cgi?product=Community%20Repositories

## Setup a local testing environment

* Install vagrant-bin and virtualbox
<pre class="clear">
  equo install app-emulations/vagrant app-emulation/virtualbox-bin app-emulation/virtualbox-modules
</pre>

* Clone a copy of https://github.com/Sabayon/community-buildspec into a local directory and cd into it
* Startup the SCR vagrant VM

<pre class="clear">
  vagrant plugin install vagrant-persistent-storage
  vagrant up
</pre>

* Fork a copy of https://github.com/Sabayon/community-repositories in Github.
* The provisioning script checks out a copy into ./repositories inside your community-buildspec working copy. Update the origin remote URL to point at your own github fork 

<pre class="clear">
  cd repositories
  git remote set-url origin git@github.com:${your_github_username}/community-repositories.git
</pre>

* Make any changes to the repository config files you wish, e.g. adding a new package
* Run a test build of the repository on your own machine

<pre class="clear">
  vagrant ssh
  sudo su -
  cd /vagrant/repositories/${repo_you_want_to_build}/
  ../test.sh
</pre>

* Watch the output to see if it builds successfully
* Raise a pull request against https://github.com/Sabayon/community-repositories to have your changes considered for inclusion into the SCR

## Clean up caches

### Docker images

List current images:
<pre class="clear">
 sabayon ~ # docker images
 REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
 sabayon/builder-amd64-sihnon-common   latest              de101af3e5f0        4 minutes ago       7.919 GB
 <none>                                <none>              5d6b6223c3c6        47 minutes ago      6.747 GB
 sabayon/builder-amd64-sihnon-server   latest              6950337b5b61        2 days ago          8.17 GB
 <none>                                <none>              56c4b5cbccb2        2 days ago          6.486 GB
 sabayon/builder-amd64                 latest              023fbad8416f        2 days ago          3.535 GB
 sabayon/eit-amd64                     latest              fa07f471556f        2 days ago          1.701 GB
</pre>
Delete an image:
<pre class="clear">
 sabayon ~ # docker rmi -f sabayon/builder-amd64-sihnon-common
 Untagged: sabayon/builder-amd64-sihnon-common:latest
 Deleted: sha256:de101af3e5f0b75e38133e88aefb51f337573f2872d8c5fd334cc7333109a543
 Deleted: sha256:af2c266c622af79f5c7cd84ba96a78f5bfde1d2c593d5ba43f70c576ab0b1680
 Deleted: sha256:e75ef42c43ab2296ca96bb798f73e8fe2516b215fbe747a5c3b67148c4445c97
 Deleted: sha256:57835ffc466aec7f36b932d848419b570f6fc0d1ddc392570e63ed2b3a2c9478
</pre>
