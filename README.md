# Git-Hook-Scripts

Git-Hook-Scripts is my repository for various git hook scripts, and a mechanism for using 
them in other repostories with a minimum of duplication.

## The Hook Scripts

### check-puppet-syntax

Checks syntax of [Puppet](https://puppetlabs.com/) .pp files and .erb template files.


### create-tag-for-version

A post-commit script that creates a git tag corresponding to a Python package version.  
You need to set an environment variable `$PYTHON_PACKAGE_NAME ` to the name of the Python 
package containing the version.  

    git config --add hook-envvars.pre-commit.python-package-name pycrust

The script loads the package and looks for `pycrust.__version__` . If there is no tag 
corresponding to the version string, the script creates one.  

### pylint

Runs pylint, checking only for errors.  The repository should probably contain a pylintrc 
file so that all developers are using the same settings.  If you don't have it: 

    pip install pylint
    
This script expects to find a list of python modules to test in the `$PYLINT_PACKAGES` 
environment variable.  To set it in your git config:

    git config --add hook-envvars.post-commit.pylint-packages "PYLINT_PACKAGES"

`PYLINT_PACKAGES` should be list of names separated by spaces, with the list wrapped in 
quotes.


### run-hook-scripts

The main interface to hook scripts.  Configure your repository to use hook scripts as follows.

Create a symlink from run-hook-scripts to each hook you want to call.

    cd /path/to/your/repository/.git/hooks
    ln -s /path/to/git-hook-scripts/run-hook-scripts pre-commit

Set the hook scripts to be called in your repository configuration.

    git config --add hook-envvars.pre-commit.hook-scripts tox 
    
Set any other environment variables needed for the scripts (e.g. create-tag-for-version).  
Here is the relevant piece of .git/config from one of my repositories.

    [hook-envvars "post-commit"]
        python-package-name = wsgiapp
        hook-scripts = create-tag-for-version
    [hook-envvars "pre-commit"]
        hook-scripts = tox
        path = /Applications/Tower.app/Contents/Resources/git/libexec/git-core:/Applications/Tower.app/Contents/Resources/git/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Frameworks/Python.framework/Versions/3.4/bin:/usr/local/bin
	
On execution, run-hook-scripts reads the repository configuration and sets the environment 
variables, executes git stash, then executes the scripts in the order specified.  
If a script exits with code > 0, run-hook-scripts bails out.  On exit, run-hook-scripts 
runs git stash pop.


### tox

A pre-commit script that runs [tox](http://codespeak.net/tox/).


## Potential Issues

### $PATH

You may run into issues with the $PATH variable, especially if you use a GUI.  I work on a 
Mac and use Tower.  When it runs git, it sets 

    $PATH = /Applications/Tower.app/Contents/Resources/git/libexec/git-core:/Applications/Tower.app/Contents/Resources/git/bin:/usr/bin:/bin:/usr/sbin:/sbin

As a result, neither tox nor Python 3 could be found.  

The solution is to set the $PATH variable yourself in your git configuration.  Here's mine.

    git config --add hook-envvars.pre-commit.path \
    /Applications/Tower.app/Contents/Resources/git/libexec/git-core:\
    /Applications/Tower.app/Contents/Resources/git/bin:/usr/bin:/bin:/usr/sbin:/sbin:\
    /Library/Frameworks/Python.framework/Versions/3.4/bin:/usr/local/bin


If you run into this issue and need to see the value of $PATH, add this line to 
run-hook-scripts near the top.

    echo "PATH = $PATH"
    
It should appear in whatever output you get.  Alternatively, redirect to a file.

    echo "PATH = $PATH" > /tmp/PATH









