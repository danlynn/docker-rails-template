# Template: Rails

This template is based on the tutorial [Quickstart: Compose and Rails](https://docs.docker.com/compose/rails/) on the docker website and allows you to create a new rails app or convert an existing rails app over to docker.

## How to create a new rails app using this template

1. Duplicate this template directory and rename it as appropriate for your project.
2. Change the ruby version identified in this template's `Dockerfile` to the version desired.
3. Change the rails version identified in this template's `Gemfile` to the version desired.
4. Change the docker-compose.yml `environment` settings `MYSQL_*` to your desired values. Note that the `MYSQL_DATABASE` database will be created when the db image is first built. 
5. Run the following command which will create the docker container, install the minimal gems necessary to create a rails app, then actually create the rails app:
   
   ```bash
   $ docker-compose run web rails new . --force --database=mysql --skip-bundle
   ```
6. Run the following command to rebuild the image with the new gems.  You should re-run this command any time you change the Gemfile.
   
   ```bash
   $ docker-compose build
   ```
7. Replace the generated `config/database.yml` with the `config/sample_database.yml` file from this template.  Be sure to change the `database` names for each environment to match up with the values in the docker-compose.yml file.
8. Run the following command to start the rails instance.
   
   ```bash
   $ docker-compose up
   ```
9. Before opening the app in the browser, create the test database in a separate terminal as follows. Note that the db service must already be running (`docker-compose up db`):

   ```bash
   $ docker-compose run --rm db mysql -h db -u root -p
   
   Enter password: *****
   
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 40
   Server version: 5.5.54 MySQL Community Server (GPL)

   mysql> show databases;
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | myapp_development  |
   | mysql              |
   | performance_schema |
   +--------------------+
   4 rows in set (0.00 sec)

   mysql> create database core_test;
   Query OK, 1 row affected (0.00 sec)

   mysql> grant all privileges on core_test.* to rails@"%";
   Query OK, 0 rows affected (0.00 sec)


   mysql> show databases;
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | myapp_development  |
   | myapp_test         |
   | mysql              |
   | performance_schema |
   +--------------------+
   5 rows in set (0.00 sec)
   ```
   
10. Verify database connection works from rails console:
    
    ```
    docker-compose run --rm web rails c
    
    Loading development environment (Rails 4.2.3)

    irb(main):001:0> ActiveRecord::Base.establish_connection
    
    => #<ActiveRecord::ConnectionAdapters::ConnectionPool:0x005593520f6460 @mon_owner=nil, @mon_count=0, @mon_mutex=#<Thread::Mutex:0x005593520f63e8>, @spec=#<ActiveRecord::ConnectionAdapters::ConnectionSpecification:0x005593520f8f08 @config={:adapter=>"mysql2", :database=>"core_development", :pool=>5, :username=>"rails", :password=>"password", :host=>"db"}, @adapter_method="mysql2_connection">, @checkout_timeout=5, @reaper=#<ActiveRecord::ConnectionAdapters::ConnectionPool::Reaper:0x005593520f6370 @pool=#<ActiveRecord::ConnectionAdapters::ConnectionPool:0x005593520f6460 ...>, @frequency=nil>, @size=5, @reserved_connections=#<ThreadSafe::Cache:0x005593520f62d0 @backend={}, @default_proc=nil>, @connections=[], @automatic_reconnect=true, @available=#<ActiveRecord::ConnectionAdapters::ConnectionPool::Queue:0x005593520f6118 @lock=#<ActiveRecord::ConnectionAdapters::ConnectionPool:0x005593520f6460 ...>, @cond=#<MonitorMixin::ConditionVariable:0x005593520f6000 @monitor=#<ActiveRecord::ConnectionAdapters::ConnectionPool:0x005593520f6460 ...>, @cond=#<Thread::ConditionVariable:0x005593520f5e48>>, @num_waiting=0, @queue=[]>>
    ```
    
    If you get an error stating that the mysql2 gem is missing (even tho it is not) then you may be using a version of rails 3 or 4 that has a bug.  Try changing the Gemfile with the following line:
   
   ```ruby
   gem 'mysql2', '0.3.20'
   ```
   
11. Open the app in your web browser: http://localhost:3000/
12. To stop the rails server, hit ctrl-c.


## How to update an existing rails app to use this template

1. Copy docker-compose.yml and Dockerfile over from template to your project.
2. Change the Dockerfile ruby version as desired.
3. OPTIONALLY, if you have any unbundled gems located in directories within your rails app, then you will need to be sure to add some `ADD` statements to the Dockerfile so that these unbundled gems will be available when the Dockerfile runs `bundle install`.  These `ADD` statements should precede the `bundle install` line like below:

   ```
   ADD lib/inmar /myapp/lib/inmar
   ADD lib/you /myapp/lib/you
   RUN bundle install
   ```
4. Change the docker-compose.yml `environment` settings `MYSQL_*` to your desired values. Note that the `MYSQL_DATABASE` database will be created when the db image is first built.
5. Run the following command to rebuild the image with the new gems.  You should re-run this command any time you change the Gemfile.
   
   ```bash
   $ docker-compose build
   ```
6. Modify existing config/database.yml changing the 'host' values to "db" (the name of the docker-compose db service)
7. Add rails user and grant them access to the development and test databases:

   ```bash
   $ docker-compose run --rm db mysql -h db -u root -p
   
   Enter password: *****
   
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 40
   Server version: 5.5.54 MySQL Community Server (GPL)

   mysql> grant all privileges on *.* to rails@"%";
   Query OK, 0 rows affected (0.00 sec)
   
   mysql> exit
   ```
8. Create the development and test databases:

   ```bash
   $ docker-compose run --rm web bundle exec rake db:create
   ```
   Alternatively, create the dbs, load the schema.rb, and run the seed files:
   
   ```bash
   $ docker-compose run --rm web bundle exec rake db:setup
   ```
9. Copy database data from your old db to the new docker db.
   1. Create an mysql dump of your database from your old mysql instance (alternatively, Sequel Pro can do this).  The dump file should be in SQL format and contain both schema and content.
      
      ```bash
      $ docker-compose run --rm db mysqldump -h hostname -u rails -p --result-file=/myapp/dump.sql database_name
      ```
      This command will connect to the database at `hostname` with user `rails` and create a `dump.sql` file in the current project directory.  It will prompt for the password.
   2. Copy this dump file to your project dir which is shared with the docker container.
   3. Make sure that the docker-compose 'db' service is running first: 
      
      ```bash
      $ docker-compose up db
      ```
   4. Import the dump file into the docker db:
      
      ```bash
      $ docker-compose run --rm db mysql -h db -u root --password=password development < dump.sql
      ```
      Note that the password must be specified in the command since the dump.sql file is being redirected to stdin for the import.

10. Verify that database looks good from rails console:

    ```bash
    $ docker-compose run --rm web rails c
    ```

11. Start the rails app:

    ```bash
    $ docker-compose up
    ```


## Useful docker-compose commands

| command                                 | action performed                           |
|-----------------------------------------|--------------------------------------------|
| `docker-compose up`                     | starts the db & rails server               |
| `docker-compose up db`                  | starts the db server only                  |
| `docker-compose ps`                     | show services state (db, web, etc.)        |
| `docker-compose stop`                   | stops the rails server                     |
| `docker-compose restart`                | restarts the rails server                  |
| `docker-compose run --rm web rake db:create` | run a rake task                       |
| `docker-compose build`                  | run after changes to Gemfile to build gems |
| `docker-compose run --rm web rails c`   | start the rails console                    |
| `docker-compose run --rm web bash`      | start bash session                         |
| `docker-compose run --rm db mysql -h db -u rails -p` | run mysql cli connected to db |
| `docker run -it --rm mysql:5.5 mysql -h 192.168.111.111 -u rails -p` | run mysql cli against remote host |

## RubyMine Docker Integration Setup

RubyMine starting with verison 2017.1 can work well with a Rails project that has been setup using the above instructions.  The following shows you how to configure the project to work within RubyMine.

First, we will setup a static IP for your laptop that can always be accessed from the container regardless of what wifi network you are currently connected to.  This avoids issues with dynamic IPs and the host IP specified in your project's 'config/database.yml' file.

Then, we will setup a RubyMine with a Docker deployment, deployment runtime configuration, and a remote Ruby SDK for Docker.

Before doing any of the following, be sure to stop your docker-compose containers if they are already running:

```bash
$ docker-compose stop
```

### Reconfigure database.yml

1. Modify 'config/database.yml' changing all the 'host' values to '192.168.111.111'

### Configure temporary static IP

1. Execute the following on the command line in order to associate '192.168.111.111' with your laptop's loopback interface.
   
   ```bash
   $ sudo ifconfig lo0 alias 192.168.111.111
   ```
   
   Note, however, that this will only last until the next reboot.  If you want this IP alias to be permanent then you will need to perform the steps in the following section.
   
### Make the temporary static IP permanent

In order to make this change permanent, we will actually just be setting up a launchd daemon to be ran upon every reboot of your laptop.  Note that you will need sudo permissions in order to perform this step (if you don't run with admin privileges). If you just setup your Mac with default user that was created when you first got your laptop then you will by default have admin privileges.  But, since many advanced users who might be following these instructions will have created a non-privileged user for their regular account, I've added a couple steps here to show you how to temporarily add then remove yourself from the list of sudoers - thus, temporarily granting yourself sudo (root) permissions.

1. If you don't run with admin privileges then perform the following steps to grant yourself sudo access:
   
   1. Switch to your admin account, enter your admin password then launch visudo:
      
      ```bash
      $ su admin
      Password: *****
      $ sudo visudo
      Password: *****
      ```
      
      This will open the sudoers file in the 'vi' editor.  Do not attempt to edit the sudoers file in any other way.
   
   2. Add your regular account username below the root and %admin group entries like the following (except use your regular username instead of 'danlynn'):
   
      ```bash
      # root and users in group wheel can run anything on any machine as any user
      root            ALL = (ALL) ALL
      %admin          ALL = (ALL) ALL
      danlynn         ALL = (ALL) ALL
      ```
      
      These lines are usually found near the bottom of the file.  After adding that last line, simply save the changes and exit back to the command line.
   
2. Create a shell script to execute the IP alias command from the previous section:
   1. Create a `.login` directory in your home directory
   
      ```bash
      $ cd ~
      $ mkdir .login
      $ cd .login
      $ touch alias_loopback.sh
      $ open .
      ```
      
   2. That last command should open a Finder window displaying the new file.  Open that file in your favorite editor, add the following, then save it:
      
      ```bash
      #!/bin/sh
      ifconfig lo0 alias 192.168.111.111
      ```
3. Create a launchd daemon plist file to execute the `alias_loopback.sh` file upon startup of your laptop:
   1. Create the `alias_loopback.plist` daemon plist file and set its ownership and permissions
      
      ```bash
      $ cd /Library/LaunchDaemons/
      $ sudo touch alias_loopback.plist
      Password: *****
      $ sudo chown root:wheel alias_loopback.plist
      $ sudo chmod 755 alias_loopback.plist
      ```
      
      Note that you will only have to enter your regular user permissions on the first use of sudo.  The elevated sudo permissions will stay in effect for about 5 minutes before being rescinded.
      
   2. Open the `alias_loopback.plist` file in your favorite editor (which allows you to authenticate as admin to save changes - like sublime)
   3. Modify the contents of the `alias_loopback.plist` file as follows:
      
      ```xml
      <plist version="1.0">
          <dict>
              <key>Label</key>
              <string>alias_loopback</string>
              <key>RunAtLoad</key>
              <true />
              <key>Program</key>
              <string>/Users/danlynn/.login/alias_loopback.sh</string>
          </dict>
      </plist>
      ```
   4. In theory launchd should see when the file is saved and pick up and run it.  However, if it doesn't then try the following:
      
      ```bash
      $ sudo launchctl load /Library/LaunchDaemons/alias_loopback.plist
      ```
      
   5. Test that the launch control daemon is working by restarting your laptop and then pinging the new static IP alias:
      
      ```bash
      $ ping 192.168.111.111
      
      PING 192.168.111.111 (192.168.111.111): 56 data bytes
      64 bytes from 192.168.111.111: icmp_seq=0 ttl=64 time=0.053 ms
      64 bytes from 192.168.111.111: icmp_seq=1 ttl=64 time=0.045 ms
      64 bytes from 192.168.111.111: icmp_seq=2 ttl=64 time=0.050 ms
      64 bytes from 192.168.111.111: icmp_seq=3 ttl=64 time=0.034 ms
      ```
      
      If you get timeout messages instead of results like this then try going back over these instructions and figuring out what you missed.  It might be useful to tail the system log for launchd messages in this case:
      
      ```bash
      $ sudo tail -f /var/log/system.log
      ```
      
      Also, you can stop and restart the daemon without a reboot with:
      
      ```bash
      $ sudo launchctl unload /Library/LaunchDaemons/alias_loopback.plist
      $ sudo launchctl load /Library/LaunchDaemons/alias_loopback.plist
      ```
      
4. Remove yourself from the sudoers file (if you don't run with admin privileges)
   1. Edit the sudoers file:
         
      ```bash
      sudo visudo
      Password: *****
      ```
   2. Remove your regular username from the list of users with sudo privileges so that it only contains the original root and admin entries like:
      
      ```
      # root and users in group wheel can run anything on any machine as any user
      root            ALL = (ALL) ALL
      %admin          ALL = (ALL) ALL
      ```

### Setup RubyMine

1. Add the 'Docker integration' plugin in Settings > Plugins then click the 'Install JetBrains plugin...'
2. Add a Docker deployment in Settings > Build, Execution, Deployment > Docker by clicking the '+' button.
   1. Change the 'API URL' to: "unix:///var/run/docker.sock" (when using Docker for Mac)
   2. Clear the 'Certificates folder' field
   3. Set the 'Docker Compose executable' to "/usr/local/bin/docker-compose"
3. Create docker deployment runtime configuration
   1. Select Run > Edit Configurations...
   2. Click the '+' button and select 'Docker Deployment' from the options
   3. Name the deployment "Docker Deployment"
   4. Select the 'Server' that you created in step 2 (probably "Docker")
   5. Select the 'Deployment' type of "docker-compose.yml"
   6. Click 'OK'
4. Run the docker deployment (build & start the containers)
   1. Select "Docker Deployment" from the runtime configurations list in the toolbar
   2. Click the run button.
   
   This will open the 'Docker' tool window.  You can see the deployed containers and their logs here.  You can also stop/restart/redeploy the Docker containers here.  For example, after changing the Gemfile, you would want to click the 'Compose: docker-compose.yml' node and click the 'Redeploy' button on the left in order to rebuild the gems and restart the containers.
   
   Note that the docker-compose services will not be automatically stopped when exiting out of RubyMine.  You should stop them using the 'Docker' tool window controls - or execute the following:
   
   ```bash
   $ docker-compose stop
   ```
5. Fix any empty gem directories before adding a remote ruby SDK

   RubyMine will download all the rubygems from the docker container into a local cache after the ruby SDK is added.  This cache is used for code introspection, db column introspection, etc.  It is also used to determine that certain gems are installed before performing certain tasks like running the rails server from the RubyMine run/debug configurations.  However, when copying the gem directories and file to the local cache, RubyMine skips any gems that have empty directories.  This sounds reasonable, except that certain critical gems ACTUALLY DO have empty gem directories.  Specifically, certain versions of the rails gem (like rails-3.2.21) has an empty gem directory.  This causes RubyMine to refuse to run/debug the Rails server with an error message about the rails gem missing.  To work around this issue, be sure to insert an empty file into the blank rails gem directory in the container by adding a line like the `touch` RUN command below.  It should be placed AFTER the `bundle install` and BEFORE the `VOLUME /usr/local/bundle` lines of the Dockerfile:
   
   ```
   RUN bundle install
   RUN touch /usr/local/bundle/gems/rails-3.2.21/.keep
   VOLUME /usr/local/bundle
   ```
   
6. Add a remote Ruby SDK for the docker container

   1. Go to Settings > Languages & Frameworks > Ruby SDK and Gems then click the '+' button and select 'New remote...'
   2. Select 'Docker' as the Remote Ruby Interpreter type
   3. Select the 'Server' docker deployment name that you created in step 2 (probably "Docker")
   4. Select the 'Image name' that was created by running the deployment in step 4. This image will default to the name of the rails project directory (without spaces) with the docker-compose service name ("web") appended after an "_" then followed by a ":latest". (eg: "testrails_web:latest").
   5. You should be able to leave the 'Ruby interpreter path' as its default "ruby"
   6. Click 'OK' then wait a few secs as RubyMine connects to the container and retrieves the list of deployed gems (creates local cache).
   7. Click the 'Edit Path Mappings' button (icon button found above list of ruby SDKs) and add a mapping that associates the Rails project directory on your laptop to the `/myapp` directory in the container.  This is required for certain internal RubyMine tools to work correctly (like scanning the names of rake tasks, generators, etc.)
7. Update the Rails run/debug configurations
   1. Open each Rails run/debug configuration by first selecting 'Edit Configurations...' from the run/debug configurations drop-down in the RubyMine toolbar.
   2. Select a Rails configuration
   3. Click the Add button '+' below the 'Before launch: Activiate tool window' section in the right pane.  Select 'Run External tool' from the drop-down.
   4. In the 'External Tools' window, click the Add button '+' to open the 'Create Tool' window.
   5. In the 'Create Tool' window, enter the following info leaving all other defaults:

      | field       | value          |
      |-------------|----------------|
      | Name:       | Start db       |
      | Program:    | docker-compose |
      | Parameters: | start db       |
      
      ...then click 'OK'
   6. Now back in the 'External Tools' window, click the Add button '+' again to open the 'Create Tool' window.
   7. This time, in the 'Create Tool' window, enter the following info:

      | field       | value          |
      |-------------|----------------|
      | Name:       | Stop web       |
      | Program:    | docker-compose |
      | Parameters: | stop web       |
      
      ...then click 'OK'
   8. Click 'OK' to dismiss the 'External Tools' window then click 'OK' to dismiss the 'Run/Debug Configurations' window.
   
   When adding these 'before launch' tasks to subsequent rails run/debug configurations, you can re-use the 2 tasks that you added by simply clicking to select one and hitting 'OK' in the 'External Tools' window.
   
   These 'before launch' tasks will make sure that the db is running and any web container is stopped whenever you 'run' the rails server.
   
   You could also simply do these tasks manually in the terminal before running the rails instance if you wanted to.  But, I'm generally to lazy to try to remember to do all that.
8. Configure the RubyMine database client to connect to the db service container:

   This is essentially the 100% standard RubyMine technique for setting up the data sources for the integrated Database client.  The database client not only provides a very helpful tool for interacting with the database, but it also helps you configure rubymine to be able to interact with the database directly in order to provide code completion and other introspections.
   
   1. Open the Database tool window.  This can be done in multiple ways, but the easiest to describe is probably ctrl+tab then while still holding ctrl hit the 'd' key.
   2. Click the new connection '+' button at the top-left of the Database tool window and select 'Import from sources...' at the bottom.  This will scan your config/database.yml file and automatically create connections for each.
   3. Then in the 'Data Sources and Drivers' window that pops up, check to see if a "Download missing driver files" warning is displayed at the bottom.  If it is then this is the first time that you have created a connection to the current type of database (eg: mysql).  Simply click the 'Download' link and RubyMine will take care of the rest.
   4. Add your your password and click the 'Remember password' checkbox on the right.
   5. Click 'Test Connection' to verify that it can connect to the database.  If successful then you are done!

## General RubyMine Workflow

#### Preparing to Open Project

Before launching RubyMine and/or opening your project, you probably want to check to see if any other docker containers are running which may have conflicting ports (db: 3306, web: 3000).  This can be determined by executing the following command which displays all the running docker containers (not just current project containers):

```bash
$ docker ps

$ docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS                                             NAMES
1b1213e49179        coredynamicemail_web:latest   "ruby -e $stdout.s..."   4 minutes ago       Up 4 minutes        0.0.0.0:3000->3000/tcp                            practical_meitner
9465f4000ef2        mysql:5.5                     "docker-entrypoint..."   2 hours ago         Up 8 minutes        0.0.0.0:3306->3306/tcp, 0.0.0.0:32788->3306/tcp   coredynamicemail_db_1
```

If any of the docker containers listed are NOT for the current project then you can shut them down if they have conflicting ports by going to that other project's dir and doing a `docker-compose stop` or by issuing direct container stop commands via docker using the CONTAINER ID (first column of output) like `docker stop 1b1213e49179`.

### Opening Project

If you are opening the project for the first time then you will need to deploy the project (docker build) via the 'Docker Deployment' run/debug configuration that you created in the 'RubyMine Setup' instructions earlier.  This is done by selecting 'Docker Deployment' from the run/debug configurations drop-down in the RubyMine toolbar then clicking the 'Run' (green triangle icon) button.  This will build/rebuild and launch the project's containers.

If the project's containers are already up to date and the docker containers built (but not necessarily running) then you can skip this step.

### Rake Tasks and Rails Generators

These can be ran in the normal fashion via the rake popup (option+r) and the generator popup (option+g).

### Run the Rails Server

Simply select the desired rails run/debug configuration from the drop-down in the toolbar then hit the 'Run' button (green triangle icon).  This will open up the console output and rails log tabs in the 'Run' tool window at the bottom.

### Bundle Install

Whenever you make changes to the Gemfile, you should rebuild the docker image which in turn runs `bundle install`.  This can be accomplished as follows:

1. Displaying the 'Docker' tool window by clicking the 'Docker' tool button at the botton of the RubyMine window.
2. Select the "Compose: docker-compose.yml" node at the top.
3. Click the 'Redeploy' button in the toolbar on the left.

This will rebuild the web container from the Dockerfile reinstalling the gems.

### Closing Project

Simply manually stop the db and web containers with:

```bash
$ docker-compose stop
```

...no use leaving them running.
