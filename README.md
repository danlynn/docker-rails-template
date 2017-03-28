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
   
   Enter password: 
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
3. Change the docker-compose.yml `environment` settings `MYSQL_*` to your desired values. Note that the `MYSQL_DATABASE` database will be created when the db image is first built.
4. Run the following command to rebuild the image with the new gems.  You should re-run this command any time you change the Gemfile.
   
   ```bash
   $ docker-compose build
   ```
5. Create the test db as shown above in step 9 above.
6. Modify existing config/database.yml changing the 'host' values to "db" (the name of the docker-compose db service)
7. Copy database data from your old db to the new docker db.
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
      $ docker-compose run --rm db mysql -h db -u root --password=password development < development.sql
      ```
      Note that the password must be specified in the command since the development.yml file is being redirected to stdin for the import.

8. Verify that database looks good from rails console:

    ```bash
    docker-compose run --rm web rails c
    ```

9. Start the rails app:

    ```bash
    docker-compose up
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
5. Add a remote Ruby SDK in Settings > Languages & Frameworks > Ruby SDK and Gems by clicking the '+' button and selecting 'New remote...'
   1. Select 'Docker' as the Remote Ruby Interpreter type
   2. Select the 'Server' name that you created in step 2 (probably "Docker")
   3. Select the 'Image name' that was created by running the deployment in step 4. This image will default to the name of the rails project directory (without spaces) with the docker-compose service name ("web") appended after an "_" then followed by a ":latest". (eg: "testrails_web:latest").
   4. You should be able to leave the 'Ruby interpreter path' as its default "ruby"
   5. Click 'OK' then wait a few secs as RubyMine connects to the container and retrieves the list of deployed gems.
6. Configure the RubyMine database client to connect to the db service container
   1. TBD...
