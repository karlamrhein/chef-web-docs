=====================================================
chef_data_bag_item
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/resource_chef_data_bag_item.rst>`__

.. warning:: .. tag notes_provisioning

             This functionality was available with Chef provisioning and was packaged in the Chef development kit.

             Chef Provisioning is no longer included with Chef DK, and will be officially end of life on August 31, 2019.  The source code of Chef Provisioning and the drivers have been moved into the chef-boneyard organization. Current users of Chef Provisioning should contact your Chef Customer Success Manager or Account Representative to review your options.

             .. end_tag

.. tag data_bag_item

A data bag is a container of related data bag items, where each individual data bag item is a JSON file. knife can load a data bag item by specifying the name of the data bag to which the item belongs and then the filename of the data bag item. The only structural requirement of a data bag item is that it must have an ``id``:

.. code-block:: javascript

   {
     /* This is a supported comment style */
     // This style is also supported
     "id": "ITEM_NAME",
     "key": "value"
   }

where

* ``key`` and ``value`` are the ``key:value`` pair for each additional attribute within the data bag item
* ``/* ... */`` and ``// ...`` show two ways to add comments to the data bag item

.. end_tag

Use the **chef_data_bag_item** resource to manage data bag items.

Syntax
=====================================================
The syntax for using the **chef_data_bag_item** resource in a recipe is as follows:

.. code-block:: ruby

   chef_data_bag_item 'name' do
     attribute 'value' # see properties section below
     ...
     action :action # see actions section below
   end

where

* ``chef_data_bag_item`` tells the chef-client to use the ``Chef::Provider::ChefDataBagItem`` provider during the chef-client run
* ``name`` is the name of the resource block and also the name of the data bag item
* ``attribute`` is zero (or more) of the properties that are available for this resource
* ``action`` identifies which steps the chef-client will take to bring the node into the desired state

Actions
=====================================================
This resource has the following actions:

``:create``
   Default. Use to create a data bag item.

``:delete``
   Use to delete a data bag item.

``:nothing``
   .. tag resources_common_actions_nothing

   This resource block does not act unless notified by another resource to take action. Once notified, this resource block either runs immediately or is queued up to run at the end of the Chef Infra Client run.

   .. end_tag

Properties
=====================================================
This resource has the following properties:

``chef_server``
   The URL for the Chef server.

``complete``
   Use to specify if this resource defines a data bag item completely. When ``true``, any property not specified by this resource will be reset to default property values.

``encrypt``
   Use to specify whether encryption is used for a data bag item.

``encryption_version``
   The minimum required version of data bag encryption. Possible values: ``0``, ``1``, and ``2``. When all of the machines in an organization are running chef-client version 11.6 (or higher), it is recommended that this value be set to ``2``.

``ignore_failure``
   **Ruby Type:** true, false | **Default Value:** ``false``

   Continue running a recipe if a resource fails for any reason.

``notifies``
   **Ruby Type:** Symbol, 'Chef::Resource[String]'

   .. tag resources_common_notification_notifies

   A resource may notify another resource to take action when its state changes. Specify a ``'resource[name]'``, the ``:action`` that resource should take, and then the ``:timer`` for that action. A resource may notify more than one resource; use a ``notifies`` statement for each resource to be notified.

   .. end_tag

   .. tag resources_common_notification_timers

   A timer specifies the point during the Chef Client run at which a notification is run. The following timers are available:

   ``:before``
      Specifies that the action on a notified resource should be run before processing the resource block in which the notification is located.

   ``:delayed``
      Default. Specifies that a notification should be queued up, and then executed at the end of the Chef Client run.

   ``:immediate``, ``:immediately``
      Specifies that a notification should be run immediately, per resource notified.

   .. end_tag

   .. tag resources_common_notification_notifies_syntax

   The syntax for ``notifies`` is:

   .. code-block:: ruby

     notifies :action, 'resource[name]', :timer

   .. end_tag

``raw_data``
   Use to create a data bag from a local file from ``./data_bags/bag_name/file``.

``raw_json``
   The data bag item as JSON data. For example:

   .. code-block:: javascript

      {
        "id": "adam",
        "real_name": "Adam Brent Jacob"
      }

``retries``
   **Ruby Type:** Integer | **Default Value:** ``0``

   The number of attempts to catch exceptions and retry the resource.

``retry_delay``
   **Ruby Type:** Integer | **Default Value:** ``2``

   The retry delay (in seconds).

``subscribes``
   **Ruby Type:** Symbol, 'Chef::Resource[String]'

   .. tag resources_common_notification_subscribes

   A resource may listen to another resource, and then take action if the state of the resource being listened to changes. Specify a ``'resource[name]'``, the ``:action`` to be taken, and then the ``:timer`` for that action.

   Note that ``subscribes`` does not apply the specified action to the resource that it listens to - for example:

   .. code-block:: ruby

    file '/etc/nginx/ssl/example.crt' do
      mode '0600'
      owner 'root'
    end

    service 'nginx' do
      subscribes :reload, 'file[/etc/nginx/ssl/example.crt]', :immediately
    end

   In this case the ``subscribes`` property reloads the ``nginx`` service whenever its certificate file, located under ``/etc/nginx/ssl/example.crt``, is updated. ``subscribes`` does not make any changes to the certificate file itself, it merely listens for a change to the file, and executes the ``:reload`` action for its resource (in this example ``nginx``) when a change is detected.

   .. end_tag

   .. tag resources_common_notification_timers

   A timer specifies the point during the Chef Client run at which a notification is run. The following timers are available:

   ``:before``
      Specifies that the action on a notified resource should be run before processing the resource block in which the notification is located.

   ``:delayed``
      Default. Specifies that a notification should be queued up, and then executed at the end of the Chef Client run.

   ``:immediate``, ``:immediately``
      Specifies that a notification should be run immediately, per resource notified.

   .. end_tag

   .. tag resources_common_notification_subscribes_syntax

   The syntax for ``subscribes`` is:

   .. code-block:: ruby

      subscribes :action, 'resource[name]', :timer

   .. end_tag
