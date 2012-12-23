# Nugrant

Vagrant plugin that brings user specific configuration
options. It will enable a `.vagrantuser` at different
location that will get imported into the main vagrant
config.

For example, let say the git repository you want to
expose is not located under the root folder of
your `Vagrantfile`. That means you will need to specify
an absolute host path to share the folder on the guest vm.

Your `Vagrantfile` would look like this:

    Vagrant::Config.run do |config|
      config.vm.share_folder "git", "/git", "/home/user/work/git"
    end

However, what happens when multiple developers
need to share the same `Vagrantfile`? This is the main
use case this plugin address.

## Installation

First, you need to have Vagrant installed for this gem
to work correctly.

There is two different way to install the gem. You can
install it via Vagrant or via the system gem containers.

When you install from Vagrant, the main benefit is that
it's decoupled from your other system gems. There is less
chance for this gem's dependencies, even if they are minimal,
to clash with gems already installed on your system. This is the
recommended installation method. To install, simply run:

    > vagrant gem install nugrant

If you prefer to install the gem in a system wide matters,
please use this command instead:

    > gem install nugrant

## Usage

When Vagrant starts, via any of the `vagrant` commands,
it loads all vagrant plugins it founds under the `GEM_PATH`
variable. If you installed the plugin with one of the two
methods we listed above, you DO NOT need to setup this
environment variable.

To use the plugin, first create a yaml file named
`.vagrantuser` where your `Vagrantfile` is located. The file
must be a valid yaml file:

    vm_port: 2223
    repository:
      project: "/home/user/work/git"

The configuration hierarchy you define in the `.vagrantuser` file
is imported into the `config` object of the `Vagrantfile`
under the key `user`. So, with the `.vagrantuser` file above, you
could have this `Vagrantfile` that abstract absolute paths.

    Vagrant::Config.run do |config|
      config.ssh.port config.user.vm_port

      config.vm.share_folder "git", "/git", config.user.repository.project
    end

This way, paths can be customized by every developer. They just
have to add a `.vagrantuser` file where user specific configuration
values can be specified. The `.vagrantuser` should be ignored by you
version control system so it is to committed with the project.

Additionally, you can also have a `.vagrantuser` under your user home
directory. This way, you can set parameters that will be
available to all your `Vagrantfile'. The project `.vagrantuser`
file will overrides parameters defined in the `.vagrantuser` file
defined in the user home directory

For example, you have `.vagrantuser` file located at `~/.vagrantuser`
that has the following content:

    vm_port: 2223
    repository:
      project: "/home/user/work/git"

And another `.vagrantuser` at the root of your `Vagrantfile`:

    vm_port: 3332
    repository:
      personal: "/home/user/personal/git"

Then, the `Vagrantfile` could be defined like this:

    Vagrant::Config.run do |config|
      config.ssh.port config.user.vm_port

      config.vm.share_folder "git", "/git", config.user.repository.project
      config.vm.share_folder "personal", "/personal", config.user.repository.personal
    end

That would be equivalent to:

    Vagrant::Config.run do |config|
      config.ssh.port 3332

      config.vm.share_folder "git", "/git", "/home/user/work/git"
      config.vm.share_folder "personal", "/personal", "/home/user/personal/git"
    end

As you can see, the parameters defined in the second `.vagrantuser` file
(the project one) overrides settings defined in the `.vagrantuser` found
in the home directory (the user one).

### Parameters access

Parameters in the `Vagrantfile` can be retrieved via method call
of array access.

    config.user['repository']['project'] # Array access
    config.user.repository.project       # Method access

You can even mix the two if you want, but we do not recommend
it since its always better to be consistent:

    config.user['repository'].project # Mixed access
    config.user.repository['project'] # Mixed access

Only the root key, i.e. `config.user`, cannot be access with
both syntax, only the method syntax can be used since this
is not provided by this plugin but by Vagrant itself.

### Default values

When using parameters, it is often needed so set default values
for certain parameters so if the user does not define one, the
default value will be picked up.

For example, say you want a parameter that will hold the ssh
port of the vm. This parameter will be accessible via the
parameter `config.user.vm.ssh_port`.

You can use the following snippet directly within your Vagrantfile
to set a default value for this parameter:

    Vagrant::Config.run do |config|
      config.user.defaults = {
        "vm" => {
          "ssh_port" => "3335"
        }
      }

      config.ssh.port config.user.vm.ssh_port
    end

With this Vagrantfile, the parameter `config.user.vm.ssh_port`
will default to `3335` in cases where it is not defined by the
user.

If the user decides to change it, he just has to set it in his
own `.vagrantuser` and it will override the default value defined
in the Vagrantfile.

### Vagrant commands

In this section, we describe the various vagrant commands defined
by this plugin that can be used to interact with it.

#### Parameters

This command will print the currently defined parameters at the
given location. All rules are respected when using this command.
It is usefull to see what parameters are available and what are
the current values of those parameters.

Usage:

    > vagrant user parameters
    ---
    config:
      user:
        chef:
          cookbooks_path: /Users/Chef/kitchen/cookbooks
          nodes_path: /Users/Chef/kitchen/nodes
          roles_path: /Users/Chef/kitchen/roles

## Contributing

You can contribute by filling issues when something goes
wrong or was not what you expected. I will do my best
to fix the issue either in the code or in the documentation,
where applicable.

You can also send pull requests for any feature or improvement
you think should be included in this plugin. I will evaluate
each of them and merge them as fast as possible.
