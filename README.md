## AdaptiveMD Installer
Installer script for AdaptiveMD providing platform and configuration.

There are additional guides with the details of handling each step. When
deploying AdaptiveMD on a new resource, there are some important things
to consider before you can give installation a try. Functionality and
software versions are variously included (and named) as modules, and
the best way to install the platform requires being aware of what these
are on each resource. You will also need to determing what is the best
place on the filesystem to install software, which is often different
than the best place to create and store data (which you also need to
determine then specify).

### Some Important Notes before starting
During workflow execution, we will rely on your
bashrc to set up the
basics of the environment in the form of variables and some loading
operations that are needed to prepare software in each shell instance.
Getting the bashrc right should be a trivial extension of getting the
installation done successfully.
During the course of a workflow, many shell instances will come and go,
so the requisite software must be available reliably. We currently use
your user profile to manage this. Some platform components will overwrite
these variables at runtime, if it seems like this overwrite process is
faulty, please let me (John Ossyra, jrossyra@gmail.com) know as the
logic behind the runtime (re)configuration should be resource-independent.

One thing that is not resource-independent, obviously, is the local
resource management system (LRMS). The file `exectasks.pbs` is the
platforms current way through the LRMS when executing without Radical
Pilot, and this cannot be generalized even to PBS-based schedulers,
so certainly message me if you cannot retool this file for your LRMS.
General tools for this layer are things like RP, which is well beyond
the scope of AdaptiveMD itself, so you must use a supported backend
for execution if you do not want to deal with retooling our launcher.
Configuring RP, as an example, to do this on your resource may be more
complicated than this retooling process, which basically requires you
to 1) paste operations from `exectasks.pbs` into a working job submission
script from your LRMS and 2) modify `wf_funcs.sh` to use the correct
job submission syntax for your LRMS. Message me for help with this.

### Back on track with installation...
This installation process requires some GNU compilers in the PATH when it is
run, if you do not have these availabe you will have to figure this out
separately. The rest of the installation dependencies might come from
either the installer or via modules on the resource. For example,
you can remove numpy from the list of Python packages to install if
load a python numpy module (often called "python_numpy").

Consider these things, * means required:
 - Modules available for any functionality (look through installer and compare to list from "module avail" command)
 - Location of gnu compilers
 - * MongoDB version 3 compatibility (test older/unencrypted versions in case of library problems)
 - * CUDA module/build location for OpenMM
 - * Filesystem location for software
 - * Filesystem location for data
 - * Is Radical Pilot (RP) configured on this resource?
 - * Installing for User, or Group of users?

#### Check back on the considerations above before starting!
To run the installer, make sure to first run through the script
and make any changes based on the considerations above. If you are
moving to a new resource, there is no way around making resource-
specific edits to the installation process when creating an
environment for all the functionality required for the AdaptiveMD
platform. 

After preparing your install script guided by the existing one and
the considerations above, these are the broad install steps. Trial- 
and-error is likely necessary at this step. See `harddelete.sh`,
modify and use carefully to expediently reset your installation process.
Again, additional help is available in more specific guides as listed here.


1. `./install_platform.sh` to execute installation
   - see AdaptiveMD-Platform for more details about configuring the
     installer to create a new AdaptiveMD Workflow Platform.

2. Add the "AdaptiveMD-Platform" setup to your (preferebly otherwise empty)
   bashrc. You should not need to modify many of the "ADMD_XXX" environment variables,
   the top ones change the global configuration for a platform instance,
   and the rest are all
   creating an internal structure that should not need any modification.
   - see "AdaptiveMD-Platform", and start the pasting operation from
     "example.olcftitan.bashrc". if you are using titan, you don't need
     to make a single change (to this or any other install file) to create
     a functioning AdaptiveMD Platform.

3. In a terminal with the AdaptiveMD configurations loaded from bashrc
   (ie a fresh login or after `source ~/.bashrc`), type these commands.
   You don't need the `$ADMD_RUNTIME` bit if this directory is in your path
   as it will be if didn't delete this part of the example configuration.
   ```bash
   cd $ADMD_WORKFLOWS/test-workflows/chignolin
   $ADMD_RUNTIME/launch_amongod.sh mongo/test
   $ADMD_RUNTIME/list_mongods
   python
   ```
   The `list_mongods` command should show you 1 current process with mongod as
   the executable. In the python interpreter you just opened, check
   that your AdaptiveMD instance connects to a MongoDB:
   ```python
   import adaptivemd
   adaptivemd.Project.list()
   p = adaptivemd.Project('funone')
   p.initialize()
   p.close()
   adaptivemd.Project.list()
   ```
   After the initialize, the second listing should tell you `['funone']`
