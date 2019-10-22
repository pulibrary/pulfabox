# PulfaBox
A Vagrant file for creating a VM for working on the pulfa codebase

# Prerequisites
  * Vagrant
  * a clone of [the pulfa codebase](https://github.com/pulibrary/pulfa)
  * a working copy of the findingaids data
  
# Configuration
You **must** edit the Vagrantfile and configure the synced folders.  See instructions in the Vagrantfile.

# Installation
When you have configured the Vagrantfile to conform with your host environment, <kbd>cd</kbd> to the directory you have cloned and run <kbd>vagrant up</kbd>.

``` sh
$ cd pulfabox
$ vagrant up
```

(NB: do not type the <kbd>$</kbd> character; that's the shell prompt. It's printed here to show you what your command line should look like.)

When vagrant has finished building the VM, type <kbd>vagrant ssh</kbd> to connect to the running VM.

Solr and eXistdb should both be running after you've built the machine.  Should you ever need to restart the VM, you will have to restart eXistdb manually; to do so, type this at the commandline:

``` sh
$ sudo -u existdb /usr/local/lib/exist/bin/startup.sh &
```

# Usage
You **must** activate the Python virtual environment before running any of the pulfamanager Python scripts: to activate the Python virtual environment, make sure you are in /usr/local/bin/pulfamanager.

``` sh
$ cd /usr/local/bin/pulfamanager
$ source venv/bin/activate
```

## Loading Individual Records
To load individual records into eXist and the solr index, become the pulfa user (using the standard credentials) and go to the `/usr/local/bin/pulfamanager` directory. Make sure the virtual environment has been activated.

``` sh
$ su pulfa
$ cd /usr/local/bin/pulfamanager
$ source venv/bin/activate
$ python load_record.py path/to/record
```

where <kbd>path/to/record</kbd> is the file path to the record you wish to load.  For example, if you wanted to load `LAE001.EAD.xml` into eXist and have it indexed by Solr, you would type the following:

``` sh
$ python load_record.py /data/eads/lae/LAE001.EAD.xml
```
