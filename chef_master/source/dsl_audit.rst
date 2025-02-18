=====================================================
About the Audit Mode DSL
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/dsl_audit.rst>`__

.. tag dsl_recipe_summary

The Recipe DSL is a Ruby DSL that is primarily used to declare resources from within a recipe. The Recipe DSL also helps ensure that recipes interact with nodes (and node properties) in the desired manner. Most of the methods in the Recipe DSL are used to find a specific parameter and then tell the Chef Infra Client what action(s) to take, based on whether that parameter is present on a node.

.. end_tag

Because the Audit Mode DSL is a Ruby DSL, then anything that can be done using Ruby can also be done in a recipe, including ``if`` and ``case`` statements, using the ``include?`` Ruby method, including recipes in recipes, and checking for dependencies.

Use Ruby
=====================================================
Common Ruby techniques can be used with the Recipe DSL methods.

if Statements
-----------------------------------------------------
.. tag ruby_style_basics_statement_if

An ``if`` statement can be used to specify part of a recipe to be used when certain conditions are met. ``else`` and ``elseif`` statements can be used to handle situations where either the initial condition is not met or when there are other possible conditions that can be met. Since this behavior is 100% Ruby, do this in a recipe the same way here as anywhere else.

For example, using an ``if`` statement with the ``platform`` node attribute:

.. code-block:: ruby

   if node['platform'] == 'ubuntu'
     # do ubuntu things
   end

.. future example: step_resource_ruby_block_reload_configuration
.. future example: step_resource_ruby_block_run_specific_blocks_on_specific_platforms
.. future example: step_resource_mount_mysql
.. future example: step_resource_package_install_sudo_configure_etc_sudoers
.. future example: step_resource_ruby_block_if_statement_use_with_platform
.. future example: step_resource_remote_file_transfer_remote_source_changes
.. future example: step_resource_remote_file_use_platform_family
.. future example: step_resource_scm_use_different_branches
.. future example: step_resource_service_stop_do_stuff_start

.. end_tag

case Statements
-----------------------------------------------------
.. tag ruby_style_basics_statement_case

A ``case`` statement can be used to handle a situation where there are a lot of conditions. Use the ``when`` statement for each condition, as many as are required.

For example, using a ``case`` statement with the ``platform`` node attribute:

.. code-block:: ruby

   case node['platform']
   when 'debian', 'ubuntu'
     # do debian/ubuntu things
   when 'redhat', 'centos', 'fedora'
     # do redhat/centos/fedora things
   end

For example, using a ``case`` statement with the ``platform_family`` node attribute:

.. code-block:: ruby

   case node['platform_family']
   when 'debian'
     # do things on debian-ish platforms (debian, ubuntu, linuxmint)
   when 'rhel'
     # do things on RHEL platforms (redhat, centos, scientific, etc)
   end

.. future example: step_resource_package_install_package_on_platform
.. future example: step_resource_package_use_case_statement
.. future example: step_resource_service_manage_ssh_based_on_node_platform

.. end_tag

include? Method
-----------------------------------------------------
.. tag ruby_style_basics_parameter_include

The ``include?`` method can be used to ensure that a specific parameter is included before an action is taken. For example, using the ``include?`` method to find a specific parameter:

.. code-block:: ruby

   if %w(debian ubuntu).include?(node['platform'])
     # do debian/ubuntu things
   end

or:

.. code-block:: ruby

   if %w{rhel}.include?(node['platform_family'])
     # do RHEL things
   end

.. end_tag

Array Syntax Shortcut
-----------------------------------------------------
.. tag ruby_style_basics_array_shortcut

The ``%w`` syntax is a Ruby shortcut for creating an array without requiring quotes and commas around the elements.

For example:

.. code-block:: ruby

   if %w(debian ubuntu).include?(node['platform'])
     # do debian/ubuntu things with the Ruby array %w() shortcut
   end

.. end_tag

control
----------------------------------------------------
Use the ``control`` method to define a specific series of tests that comprise an individual audit. A ``control`` method MUST be contained within a ``control_group`` block. A ``control_group`` block may contain multiple ``control`` methods.



The syntax for the ``control`` method is as follows:

.. code-block:: ruby

   control_group 'audit name' do
     control 'name' do
       it 'should do something' do
         expect(something).to/.to_not be_something
       end
     end
   end

where:

* ``control_group`` groups one (or more) ``control`` blocks
* ``control 'name' do`` defines an individual audit
* Each ``control`` block must define at least one validation
* Each ``it`` statement defines a single validation. ``it`` statements are processed individually when the Chef Infra Client is run in audit-mode
* An ``expect(something).to/.to_not be_something`` is a statement that represents the individual test. In other words, this statement tests if something is expected to be (or not be) something. For example, a test that expects the PostgreSQL pacakge to not be installed would be similar to ``expect(package('postgresql')).to_not be_installed`` and a test that ensures a service is enabled would be similar to ``expect(service('init')).to be_enabled``
* An ``it`` statement may contain multiple ``expect`` statements



directory Matcher
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Matchers are available for directories. Use this matcher to define audits for directories that test if the directory exists, is mounted, and if it is linked to. This matcher uses the same matching syntax---``expect(file('foo'))``---as the files. The following matchers are available for directories:

.. list-table::
   :widths: 60 420
   :header-rows: 1

   * - Matcher
     - Description, Example
   * - ``be_directory``
     - Use to test if directory exists. For example:

       .. code-block:: ruby

          it 'should be a directory' do
            expect(file('/var/directory')).to be_directory
          end

   * - ``be_linked_to``
     - Use to test if a subject is linked to the named directory. For example:

       .. code-block:: ruby

          it 'should be linked to the named directory' do
            expect(file('/etc/directory')).to be_linked_to('/etc/some/other/directory')
          end

   * - ``be_mounted``
     - Use to test if a directory is mounted. For example:

       .. code-block:: ruby

          it 'should be mounted' do
            expect(file('/')).to be_mounted
          end

       For directories with a single attribute that requires testing:

       .. code-block:: ruby

          it 'should be mounted with an ext4 partition' do
            expect(file('/')).to be_mounted.with( :type => 'ext4' )
          end

       For directories with multiple attributes that require testing:

       .. code-block:: ruby

          it 'should be mounted only with certain attributes' do
            expect(file('/')).to be_mounted.only_with(
              :attribute => 'value',
              :attribute => 'value',
          )
          end



file Matcher
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Matchers are available for files and directories. Use this matcher to define audits for files that test if the file exists, its version, if it is executable, writable, or readable, who owns it, verify checksums (both MD5 and SHA-256) and so on. The following matchers are available for files:

.. list-table::
   :widths: 60 420
   :header-rows: 1

   * - Matcher
     - Description, Example
   * - ``be_executable``
     - Use to test if a file is executable. For example:

       .. code-block:: ruby

          it 'should be executable' do
            expect(file('/etc/file')).to be_executable
          end

       For a file that is executable by its owner:

       .. code-block:: ruby

          it 'should be executable by owner' do
            expect(file('/etc/file')).to be_executable.by('owner')
          end

       For a file that is executable by a group:

       .. code-block:: ruby

          it 'should be executable by group members' do
            expect(file('/etc/file')).to be_executable.by('group')
          end

       For a file that is executable by a specific user:

       .. code-block:: ruby

          it 'should be executable by user foo' do
            expect(file('/etc/file')).to be_executable.by_user('foo')
          end

   * - ``be_file``
     - Use to test if a file exists. For example:

       .. code-block:: ruby

          it 'should be a file' do
            expect(file('/etc/file')).to be_file
          end

   * - ``be_grouped_into``
     - Use to test if a file is grouped into the named group. For example:

       .. code-block:: ruby

          it 'should be grouped into foo' do
            expect(file('/etc/file')).to be_grouped_into('foo')
          end

   * - ``be_linked_to``
     - Use to test if a subject is linked to the named file. For example:

       .. code-block:: ruby

          it 'should be linked to the named file' do
            expect(file('/etc/file')).to be_linked_to('/etc/some/other/file')
          end

   * - ``be_mode``
     - Use to test if a file is set to the specified mode. For example:

       .. code-block:: ruby

          it 'should be mode 440' do
            expect(file('/etc/file')).to be_mode(440)
          end

   * - ``be_owned_by``
     - Use to test if a file is owned by the named owner. For example:

       .. code-block:: ruby

          it 'should be owned by the root user' do
            expect(file('/etc/sudoers')).to be_owned_by('root')
          end

   * - ``be_readable``
     - Use to test if a file is readable. For example:

       .. code-block:: ruby

          it 'should be readable' do
            expect(file('/etc/file')).to be_readable
          end

       For a file that is readable by its owner:

       .. code-block:: ruby

          it 'should be readable by owner' do
            expect(file('/etc/file')).to be_readable.by('owner')
          end

       For a file that is readable by a group:

       .. code-block:: ruby

          it 'should be readable by group members' do
            expect(file('/etc/file')).to be_readable.by('group')
          end

       For a file that is readable by a specific user:

       .. code-block:: ruby

          it 'should be readable by user foo' do
            expect(file('/etc/file')).to be_readable.by_user('foo')
          end

   * - ``be_socket``
     - Use to test if a file exists as a socket. For example:

       .. code-block:: ruby

          it 'should be a socket' do
            expect(file('/var/file.sock')).to be_socket
          end

   * - ``be_symlink``
     - Use to test if a file exists as a symbolic link. For example:

       .. code-block:: ruby

          it 'should be a symlink' do
            expect(file('/etc/file')).to be_symlink
          end

   * - ``be_version``
     - Microsoft Windows only. Use to test if a file is the specified version. For example:

       .. code-block:: ruby

          it 'should be version 1.2' do
            expect(file('C:\\Windows\\path\\to\\file')).to be_version('1.2')
          end

   * - ``be_writable``
     - Use to test if a file is writable. For example:

       .. code-block:: ruby

          it 'should be writable' do
            expect(file('/etc/file')).to be_writable
          end

       For a file that is writable by its owner:

       .. code-block:: ruby

          it 'should be writable by owner' do
            expect(file('/etc/file')).to be_writable.by('owner')
          end

       For a file that is writable by a group:

       .. code-block:: ruby

          it 'should be writable by group members' do
            expect(file('/etc/file')).to be_writable.by('group')
          end

       For a file that is writable by a specific user:

       .. code-block:: ruby

          it 'should be writable by user foo' do
            expect(file('/etc/file')).to be_writable.by_user('foo')
          end

   * - ``contain``
     - Use to test if a file contains specific contents. For example:

       .. code-block:: ruby

          it 'should contain docs.chef.io' do
            expect(file('/etc/file')).to contain('docs.chef.io')
          end



package Matcher
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Matchers are available for packages and may be used to define audits that test if a package or a package version is installed. The following matchers are available:

.. list-table::
   :widths: 60 420
   :header-rows: 1

   * - Matcher
     - Description, Example
   * - ``be_installed``
     - Use to test if the named package is installed. For example:

       .. code-block:: ruby

          it 'should be installed' do
            expect(package('httpd')).to be_installed
          end

       For a specific package version:

       .. code-block:: ruby

          it 'should be installed' do
            expect(package('httpd')).to be_installed.with_version('0.1.2')
          end



port Matcher
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Matchers are available for ports and may be used to define audits that test if a port is listening. The following matchers are available:

.. list-table::
   :widths: 60 420
   :header-rows: 1

   * - Matcher
     - Description, Example
   * - ``be_listening``
     - Use to test if the named port is listening. For example:

       .. code-block:: ruby

          it 'should be listening' do
            expect(port(23)).to be_listening
          end

       For a named port that is not listening:

       .. code-block:: ruby

          it 'should not be listening' do
            expect(port(23)).to_not be_listening
          end

       For a specific port type use ``.with('port_type')``. For example, UDP:

       .. code-block:: ruby

          it 'should be listening with UDP' do
            expect(port(23)).to_not be_listening.with('udp')
          end

       For UDP, version 6:

       .. code-block:: ruby

          it 'should be listening with UDP6' do
            expect(port(23)).to_not be_listening.with('udp6')
          end

       For TCP/IP:

       .. code-block:: ruby

          it 'should be listening with TCP' do
            expect(port(23)).to_not be_listening.with('tcp')
          end

       For TCP/IP, version 6:

       .. code-block:: ruby

          it 'should be listening with TCP6' do
            expect(port(23)).to_not be_listening.with('tcp6')
          end



service Matcher
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Matchers are available for services and may be used to define audits that test for conditions related to services, such as if they are enabled, running, have the correct startup mode, and so on. The following matchers are available:

.. list-table::
   :widths: 60 420
   :header-rows: 1

   * - Matcher
     - Description, Example
   * - ``be_enabled``
     - Use to test if the named service is enabled (i.e. will start up automatically). For example:

       .. code-block:: ruby

          it 'should be enabled' do
            expect(service('ntpd')).to be_enabled
          end

       For a service that is enabled at a given run level:

       .. code-block:: ruby

          it 'should be enabled at the specified run level' do
            expect(service('ntpd')).to be_enabled.with_level(3)
          end

   * - ``be_installed``
     - Microsoft Windows only. Use to test if the named service is installed on the Microsoft Windows platform. For example:

       .. code-block:: ruby

          it 'should be installed' do
            expect(service('DNS Client')).to be_installed
          end

   * - ``be_running``
     - Use to test if the named service is running. For example:

       .. code-block:: ruby

          it 'should be running' do
            expect(service('ntpd')).to be_running
          end

       For a service that is running under supervisor:

       .. code-block:: ruby

          it 'should be running under supervisor' do
            expect(service('ntpd')).to be_running.under('supervisor')
          end

       or daemontools:

       .. code-block:: ruby

          it 'should be running under daemontools' do
            expect(service('ntpd')).to be_running.under('daemontools')
          end

       or Upstart:

       .. code-block:: ruby

          it 'should be running under upstart' do
            expect(service('ntpd')).to be_running.under('upstart')
          end

   * - ``be_monitored_by``
     - Use to test if the named service is being monitored by the named monitoring application. For example:

       .. code-block:: ruby

          it 'should be monitored by' do
            expect(service('ntpd')).to be_monitored_by('monit')
          end

   * - ``have_start_mode``
     - Microsoft Windows only. Use to test if the named service's startup mode is correct on the Microsoft Windows platform. For example:

       .. code-block:: ruby

          it 'should start manually' do
            expect(service('DNS Client')).to have_start_mode.Manual
          end



Examples
+++++++++++++++++++++++++++++++++++++++++++++++++++++

**A package is installed**

For example, a package is installed:

.. code-block:: ruby

   control_group 'audit name' do
     control 'mysql package' do
       it 'should be installed' do
         expect(package('mysql')).to be_installed
       end
     end
   end

The ``control_group`` block is processed when the Chef Infra Client run is run in audit-mode. If the audit was successful, the Chef Infra Client will return output similar to:

.. code-block:: bash

   Audit Mode
     mysql package
       should be installed

If an audit was unsuccessful, the Chef Infra Client will return output similar to:

.. code-block:: bash

   Starting audit phase

   Audit Mode
     mysql package
     should be installed (FAILED - 1)

   Failures:

   1) Audit Mode mysql package should be installed
     Failure/Error: expect(package('mysql')).to be_installed.with_version('5.6')
       expected Package 'mysql' to be installed
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:22:in 'block (3 levels) in from_file'

   Finished in 0.5745 seconds (files took 0.46481 seconds to load)
   1 examples, 1 failures

   Failed examples:

   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:21 # Audit Mode mysql package should be installed



**A package version is installed**

A package that is installed with a specific version:

.. code-block:: ruby

   control_group 'audit name' do
     control 'mysql package' do
       it 'should be installed' do
         expect(package('mysql')).to be_installed.with_version('5.6')
       end
     end
   end



**A package is not installed**

A package that is not installed:

.. code-block:: ruby

   control_group 'audit name' do
     control 'postgres package' do
       it 'should not be installed' do
         expect(package('postgresql')).to_not be_installed
       end
     end
   end

If the audit was successful, the Chef Infra Client will return output similar to:

.. code-block:: bash

   Audit Mode
     postgres audit
       postgres package
         is not installed



**A service is enabled**

A service that is enabled and running:

.. code-block:: ruby

   control_group 'audit name' do
     control 'mysql service' do
       let(:mysql_service) { service('mysql') }
       it 'should be enabled' do
         expect(mysql_service).to be_enabled
       end
       it 'should be running' do
         expect(mysql_service).to be_running
       end
     end
   end

If the audit was successful, the Chef Infra Client will return output similar to:

.. code-block:: bash

   Audit Mode
     mysql service audit
       mysql service
         is enabled
         is running



**A configuration file contains specific settings**

The following example shows how to verify ``sshd`` configration, including whether it's installed, what the permissions are, and how it can be accessed:

.. code-block:: ruby

   control_group 'check sshd configuration' do

     control 'sshd package' do
       it 'should be installed' do
         expect(package('openssh-server')).to be_installed
       end
     end

     control 'sshd configuration' do
       let(:config_file) { file('/etc/ssh/sshd_config') }
       it 'should exist with the right permissions' do
         expect(config_file).to be_file
         expect(config_file).to be_mode(644)
         expect(config_file).to be_owned_by('root')
         expect(config_file).to be_grouped_into('root')
       end
       it 'should not permit RootLogin' do
         expect(config_file.content).to_not match(/^PermitRootLogin yes/)
       end
       it 'should explicitly not permit PasswordAuthentication' do
         expect(config_file.content).to match(/^PasswordAuthentication no/)
       end
       it 'should force privilege separation' do
         expect(config_file.content).to match(/^UsePrivilegeSeparation sandbox/)
       end
     end
   end

where

* ``let(:config_file) { file('/etc/ssh/sshd_config') }`` uses the ``file`` matcher to test specific settings within the ``sshd`` configuration file



**A file contains desired permissions and contents**

The following example shows how to verify that a file has the desired permissions and contents:

.. code-block:: ruby

   controls 'mysql config' do
     control 'mysql config file' do
       let(:config_file) { file('/etc/mysql/my.cnf') }
       it 'exists with correct permissions' do
         expect(config_file).to be_file
         expect(config_file).to be_mode(0400)
       end
       it 'contains required configuration' do
         expect(its('contents')).to match(/default-time-zone='UTC'/)
       end
     end
   end

If the audit was successful, the Chef Infra Client will return output similar to:

.. code-block:: bash

   Audit Mode
     mysql config
       mysql config file
         exists with correct permissions
         contains required configuration



**Test an attribute value**

To audit attribute values in a ``control`` block, first assign the attribute as a variable, and then use the variable within the ``control`` block to specify the test:

.. code-block:: ruby

   memory_mb = node['memory']['total'].gsub(/kB$/i, '').to_i / 1024
   control 'minimum memory check' do
     it 'should be at least 400MB free' do
       expect(memory_mb).to be >= 400
     end
   end

control_group
----------------------------------------------------
Use the ``control_group`` method to define a group of ``control`` methods that comprise a single audit. The name of each ``control_group`` must be unique within the organization.



The syntax for the ``control_group`` method is as follows:

.. code-block:: ruby

   control_group 'name' do
     control 'name' do
       it 'should do something' do
         expect(something).to/.to_not be_something
       end
     end
     control 'name' do
       ...
     end
     ...
   end

where:

* ``control_group`` groups one (or more) ``control`` blocks
* ``'name'`` is the unique name for the ``control_group``; the Chef Infra Client will raise an exception if duplicate ``control_group`` names are present
* ``control`` defines each individual audit within the ``control_group`` block. There is no limit to the number of ``control`` blocks that may defined within a ``control_group`` block



Examples
+++++++++++++++++++++++++++++++++++++++++++++++++++++

**control_group block with multiple control blocks**

The following ``control_group`` ensures that MySQL is installed, that PostgreSQL is not installed, and that the services and configuration files associated with MySQL are configured correctly:

.. code-block:: ruby

   control_group 'Audit Mode' do

     control 'mysql package' do
       it 'should be installed' do
         expect(package('mysql')).to be_installed.with_version('5.6')
       end
     end

     control 'postgres package' do
       it 'should not be installed' do
         expect(package('postgresql')).to_not be_installed
       end
     end

     control 'mysql service' do
       let(:mysql_service) { service('mysql') }
       it 'should be enabled' do
         expect(mysql_service).to be_enabled
       end
       it 'should be running' do
         expect(mysql_service).to be_running
       end
     end

     control 'mysql config directory' do
       let(:config_dir) { file('/etc/mysql') }
       it 'should exist with correct permissions' do
         expect(config_dir).to be_directory
         expect(config_dir).to be_mode(0700)
       end
       it 'should be owned by the db user' do
         expect(config_dir).to be_owned_by('db_service_user')
       end
     end

     control 'mysql config file' do
       let(:config_file) { file('/etc/mysql/my.cnf') }
       it 'should exist with correct permissions' do
         expect(config_file).to be_file
         expect(config_file).to be_mode(0400)
       end
       it 'should contain required configuration' do
         expect(config_file.content).to match(/default-time-zone='UTC'/)
       end
     end

   end

The ``control_group`` block is processed when the Chef Infra Client is run in audit-mode. If the Chef Infra Client run was successful, the Chef Infra Client will return output similar to:

.. code-block:: bash

   Audit Mode
     mysql package
       should be installed
     postgres package
       should not be installed
     mysql service
       should be enabled
       should be running
     mysql config directory
       should exist with correct permissions
       should be owned by the db user
     mysql config file
       should exist with correct permissions
       should contain required configuration

If an audit was unsuccessful, the Chef Infra Client will return output similar to:

.. code-block:: bash

   Starting audit phase

   Audit Mode
     mysql package
     should be installed (FAILED - 1)
   postgres package
     should not be installed
   mysql service
     should be enabled (FAILED - 2)
     should be running (FAILED - 3)
   mysql config directory
     should exist with correct permissions (FAILED - 4)
     should be owned by the db user (FAILED - 5)
   mysql config file
     should exist with correct permissions (FAILED - 6)
     should contain required configuration (FAILED - 7)

   Failures:

   1) Audit Mode mysql package should be installed
     Failure/Error: expect(package('mysql')).to be_installed.with_version('5.6')
       expected Package 'mysql' to be installed
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:22:in 'block (3 levels) in from_file'

   2) Audit Mode mysql service should be enabled
     Failure/Error: expect(mysql_service).to be_enabled
       expected Service 'mysql' to be enabled
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:35:in 'block (3 levels) in from_file'

   3) Audit Mode mysql service should be running
      Failure/Error: expect(mysql_service).to be_running
       expected Service 'mysql' to be running
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:38:in 'block (3 levels) in from_file'

   4) Audit Mode mysql config directory should exist with correct permissions
     Failure/Error: expect(config_dir).to be_directory
       expected `File '/etc/mysql'.directory?` to return true, got false
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:45:in 'block (3 levels) in from_file'

   5) Audit Mode mysql config directory should be owned by the db user
     Failure/Error: expect(config_dir).to be_owned_by('db_service_user')
       expected `File '/etc/mysql'.owned_by?('db_service_user')` to return true, got false
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:49:in 'block (3 levels) in from_file'

   6) Audit Mode mysql config file should exist with correct permissions
     Failure/Error: expect(config_file).to be_file
       expected `File '/etc/mysql/my.cnf'.file?` to return true, got false
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:56:in 'block (3 levels) in from_file'

   7) Audit Mode mysql config file should contain required configuration
     Failure/Error: expect(config_file.content).to match(/default-time-zone='UTC'/)
       expected '-n\n' to match /default-time-zone='UTC'/
       Diff:
       @@ -1,2 +1,2 @@
       -/default-time-zone='UTC'/
       +-n
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:60:in 'block (3 levels) in from_file'

   Finished in 0.5745 seconds (files took 0.46481 seconds to load)
   8 examples, 7 failures

   Failed examples:

   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:21 # Audit Mode mysql package should be installed
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:34 # Audit Mode mysql service should be enabled
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:37 # Audit Mode mysql service should be running
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:44 # Audit Mode mysql config directory should exist with correct permissions
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:48 # Audit Mode mysql config directory should be owned by the db user
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:55 # Audit Mode mysql config file should exist with correct permissions
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:59 # Audit Mode mysql config file should contain required configuration
   Auditing complete



**Duplicate control_group names**

If two ``control_group`` blocks have the same name, the Chef Infra Client will raise an exception. For example, the following ``control_group`` blocks exist in different cookbooks:

.. code-block:: ruby

   control_group 'basic control group' do
     it 'should pass' do
       expect(2 - 2).to eq(0)
     end
   end

.. code-block:: ruby

   control_group 'basic control group' do
     it 'should pass' do
       expect(3 - 2).to eq(1)
     end
   end

Because the two ``control_group`` block names are identical, the Chef Infra Client will return an exception similar to:

.. code-block:: ruby

   Synchronizing Cookbooks:
     - audit_test
   Compiling Cookbooks...

   ================================================================================
   Recipe Compile Error in /Users/grantmc/.cache/chef/cache/cookbooks
                           /audit_test/recipes/error_duplicate_control_groups.rb
   ================================================================================

   Chef::Exceptions::AuditControlGroupDuplicate
   --------------------------------------------
   Audit control group with name 'basic control group' has already been defined

   Cookbook Trace:
   ---------------
   /Users/grantmc/.cache/chef/cache/cookbooks
   /audit_test/recipes/error_duplicate_control_groups.rb:13:in 'from_file'

   Relevant File Content:
   ----------------------
   /Users/grantmc/.cache/chef/cache/cookbooks/audit_test/recipes/error_duplicate_control_groups.rb:

   control_group 'basic control group' do
     it 'should pass' do
       expect(2 - 2).to eq(0)
     end
   end

   control_group 'basic control group' do
     it 'should pass' do
       expect(3 - 2).to eq(1)
     end
   end

   Running handlers:
   [2015-01-15T09:36:14-08:00] ERROR: Running exception handlers
   Running handlers complete



**Verify a package is installed**

The following ``control_group`` verifies that the ``git`` package has been installed:

.. code-block:: ruby

   package 'git' do
     action :install
   end

   execute 'list packages' do
     command 'dpkg -l'
   end

   execute 'list directory' do
     command 'ls -R ~'
   end

   control_group 'my audits' do
     control 'check git' do
       it 'should be installed' do
         expect(package('git')).to be_installed
       end
     end
   end


