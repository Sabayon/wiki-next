# Sabayon-devkit

### NOTICE
***Repository created with this method, can't be tracked by Eit. Responsability to rebuild the packages when they are broken against main repositories is on your behalf. The Article is still a draft***


***Sabayon Devkit*** is a suite of tools with the goal of making the life easy when dealing with Sabayon internal structure, mostly contains tools related to package mantaining. 

##  Installing

It is available on Entropy (see [//packages.sabayon.org/quicksearch?q=app-misc%2Fsabayon-devkit app-misc/sabayon-devkit], and [http://gpo.zugaina.org/app-misc/sabayon-devkit gpo.zugaina.org] for portage), but you can install on well on other distros from the sources(https://github.com/Sabayon/devkit), the tools leverage Docker and does not modify your running system.

From the terminal of your Sabayon box:

 sudo equo install sabayon-devkit

##  sabayon-entropypreservedlibs

When you see messages like this

 ☛ There are 2 preserved libraries on the system
 ╠ /usr/lib64/libkipi.so.11.1.0 [libkipi.so.11:2 -> kde-apps/libkipi-15.08.3]
 ╠ /usr/lib64/libkipi.so.11 [libkipi.so.11:2 -> kde-apps/libkipi-15.08.3]

This happens mostly when you mixed Entropy with Portage, or you may still have very old and outdated packages installed.

You can use ***sabayon-entropypreservedlibs*** to see what packages causes the breakage. 

 sabayon-entropypreservedlibs
 > No packages request outdated libs

##  Prerequisites for the building and repository creator script

* Docker installed in the machine (''sudo equo i docker''), and the daemon started (''sudo systemctl start docker''). You might want enable it on start ''sudo systemctl enable docker''
* if you don't want to run that as root, the user where are you running the script must be in the docker group (''sudo gpasswd -a $USER docker'') or better, having enabled the docker bin in sudoers.
* Space: At least you will need around 2GB of disk space due to the Docker Image that contains all the needed tools.


''sabayon-buildpackages'' and ''sabayon-createrepo'' are just a script wrapper around the development Docker container.

It's strongly encouraged to create a directory for each your project (repository) and run the commands inside to it.

##  Building packages
### NOTICE
***This phase requires bandwitdth and disk space that differ for each case. ''E.g. if you decide to compile a package that have a huge dependency list***
 
You can build packages by running:

 sabayon-buildpackages <your_package_atom>

Where ''<your_package_atom>'' is in the same syntax as Emerge.

 DOCKER_PULL_IMAGE=1 sabayon-buildpackages app-foo/foobar --equo foo-misc/foobar --layman foo --layman bar foo

* ***--layman foobar*** -- tells the script to add the "foobar" overlay from layman
* ***--equo foo-misc/foobar*** -- tells the script to install "foo-misc/foobar" before compiling

Environment variables:

* ***DOCKER_PULL_IMAGE*** -- tells the script to update the docker image before compiling, enable it with 1, disable with 0
* ***OUTPUT_DIR*** -- optional, default to "portage_artifacts" in your current working directory, it is the path where emerge generated tbz2 are stored (absolute path)
* ***LOCAL_OVERLAY*** -- optional, you can specify the path to your local overlay (absolute path) The arguments are the packages that you want compile, they can be also in the complete form e.g. =foo-bar/misc-1.2

Docker will now pull (if not already) the 64bit Docker Development image. 
Then will be created a folder in your current working directory named ***portage_artifacts***. That folder will contain packages built with [[Portage]] by the Docker image upon a successful build.

##  Folder structure

This is the folder structure of your workspace by default, but you can tweak part of it to your tastes with Environment variables:


 myproject/
 myproject/portage_artifacts/
 myproject/entropy_artifacts/
 myproject/local_overlay/
 myproject/specs/

* ''myproject/portage_artifacts/'' -- Created by default when sabayon-buildpackages is used. It contains the portage artifacts, they will be consumed in the next steps
* ''myproject/entropy_artifacts/'' -- Created when sabayon-createrepo is started. It contains the entropy repository files (It can be already present if you want to update your repository)
* ''myproject/local_overlay/'' -- is the location of your personal overlay (if necessary)
* ''myproject/specs'' -- Create it to customize the building process. It can contain custom files for make.conf, uses, envs, masks, unmasks and keywords for package compilation options
the specs folder is structured like this and it's merely optional.

as long as you create those files inside the ***spec/*** folder they are used:

* ''custom.unmask'' -- that's the place for custom unmasks
* ''custom.mask'' -- contain your custom masks
* ''custom.use'' -- contain your custom use flags
* ''custom.env'' -- contain your custom env specifications
* ''custom.keywords'' -- contain your custom keywords
* ''make.conf'' -- it will replace the make.conf on the container with yours.
you can override the Architecture folder in which files are placed specifying in the SAB_ARCH environment variable. Default is "intel" (can be armarch as for now)

##  Advanced usage

The tool allows customization with environment variables and option that alters it's behavior [https://github.com/Sabayon/devkit/blob/master/README.md#building-package-in-a-clean-environment]

##  Creating repository

### NOTICE
***The tools are intended to run subsequentially, but they are completely untied. You can create repository with `sabayon-createrepo` also if where not compiled with `sabayon-buildpackages` ***


You can build packages by running on the same directory you run ''sabayon-buildpackages'':

 sabayon-createrepo
 REPOSITORY_NAME=mytest REPOSITORY_DESCRIPTION="My Wonderful Repository" sabayon-createrepo


* ***REPOSITORY_NAME*** -- optional, is your repository name id
* ***REPOSITORY_DESCRIPTION*** -- optional, is your repository description
* ***PORTAGE_ARTIFACTS*** -- optional if you use the tools in the same dir, you can specify where portage artifacts (*.tbz2 files) are (absolute path required)
* ***OUTPUT_DIR*** -- optional, you can specify where the entropy repository will be stored

You can also put your .tbz2 file externally built inside ''entropy_artifacts/'' in your workspace folder (you can create it if not already present) and run ***sabayon-createrepo*** to generate a repository from them. ***sabayon-createrepo*** can be called sequentially, and it will update the repository found on ''entropy_artifacts/'' with the packages ( in .tbz2 ) in ''portage_artifacts/''.  

In both sabayon-createrepo and sabayon-buildpackages you can override the docker image used with the environment variable ***DOCKER_IMAGE***.


It will be created an ''entropy_artifacts/'' folder that will contain your repository files generated from the packages built from the step above.

### NOTICE
***When running sabayon-createrepo the *.tbz2 of the ***PORTAGE_ARTIFACTS*** (''entropy_artifacts/'' by default) folder are removed.***

##  Advanced usage

The tool allows customization with environment variables and option that alters it's behavior [https://github.com/Sabayon/devkit/blob/master/Documentation.md]
