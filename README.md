# Basic Setup for Ruby on Rails 5 Development
## using Git, Rbenv, Bundler, MySql or Postgres
### *Edwin W. Meyer*
#### Updated July 26,2019

## Background
Basic environment setup required prior to creating a Ruby on Rails 5 application on Linux is described.  
Some modifications will be required if installing on OS X.

## Documentation Conventions
Except where specially noted:
- "$" at the beginning of a line represents the terminal command prompt.
- ">" at the beginning of a line denotes a URL to be entered into a web browser.
- "#" represents the beginning of a comment.
- "-->" at the end of a terminal command indicates that the following text on the same or subsequent lines is the expected output.
- "\-" at the beginning of a line denotes instructions to be executed.

## Detailed Steps
***No specific working directory is required***  
Since we are creating the general environment required for all Ruby on Rails 5 applications, none of these steps require any particular working directory. Here we use the home directory.
```bash
$ cd
```

***Update the Package Manager***  
Update list of available packages &ndash; advisable prior to any apt installation.  
```bash
$ sudo apt-get update 
```

***Install Git***  
```bash
$ sudo apt-get install git
$ git --version --> git version 2.17.1 
```
Note: The versions of git and other packages installed below are current as of July 17, 2019. It is generally no problem if you have a later version.  

***Remove Ruby Version Manager (rvm)***  
We will be using the "Ruby environment manager" (rbenv) &ndash; see below, which conflicts with rvm. If you already have rvm installed (`which rvm` returns its path), remove it as follows:
```bash
$ rvm implode
```

***(Optional) Install gcc Compiler***  
gcc is required to compile a bash extension that speeds up Ruby Environment Manager (rbenv) installed below. However rbenv still works without it.
```bash
$ sudo apt install gcc
$ gcc --version --> gcc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
$ which make --> /usr/bin/make # or nothing, if `make` is not installed

If `which make` returns nothing:
$ sudo apt install make
$ sudo apt install make-guile
$ make -v --> GNU Make 4.1
```

***Install Ruby Environment Manager (rbenv)***  
rbenv is a "Ruby environment manager", which allows multiple projects with different versions and gems to co-exist on the same server without conflict. Its use in a production environment will similarly allow multiple Ruby/Rails apps with different versions to co-exist on the same server.  
(rbenv is increasing favored over the older rvm package.)

```bash
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
Optional - Compile dynamic bash extension to speed up rbenv. rbenv still works if it fails:
$ cd ~/.rbenv && src/configure && make -C src 

$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc 
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
Note: .bashrc is used on Ubuntu -- for some others: .bash_profile
$ source ~/.bashrc # or open new terminal window
$ rbenv -v --> rbenv 1.1.2-2-g4e92322
```

***Install ruby-build***  
ruby-build is an rbenv plugin that provides an 'rbenv install' command to compile and install different versions of Ruby on UNIX-like systems.  
```bash
$ git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

***Install Ruby 2.6.3***  
List the available Ruby versions. (2.6.3 is the latest stable version as of 7/17/2019)
```bash
$ rbenv install -l
```

Install a specific listed Ruby version:  
```bash
$ rbenv install 2.6.3 # This can take quite a while.
 --> 
 Installed ruby-2.6.3 to <home dir>/.rbenv/versions/2.6.3
```

- You may get the error message  
ERROR: 'Ruby install aborted due to missing extensions'  
If so, perform per the error message:
```bash
$ sudo apt-get install -y libssl-dev libreadline-dev zlib1g-dev
Then repeat:
$ rbenv install 2.6.3
```

Post-install tasks:
```bash
$ rbenv rehash # Update the 'shims' used to invoke a specific command version
$ rbenv global 2.6.3 # use this Ruby if not specified in project directory
$ ruby -v --> ruby 2.6.3p62 (2019-04-16 revision 67580) [x86_64-linux]
```

- What are "shims"?  
rbenv sets up shims for each ruby version. A shim is a short bash script that determines which version of a command is to be used and executes it when the command is invoked.  For example, the shim for the bundle command is located at **`~/.rbenv/shims/bundle`**  
Unlike gems, which can have per-project versions (`bunder`handles this), a single command version serves all projects that use a particular Ruby version.  

***Install Nokogiri***  
Nokogiri is an HTML and XML parser for Ruby. It is used by all Rails apps and takes time to install. So do it once in the global gemset.
```bash
$ gem install nokogiri 
```
- If you get "ERROR: Failed to build gem native extension", the following may fix it:  
```bash
$ sudo apt-get install libgmp-dev liblzma-dev zlib1g-dev # install as root user
```  
Then repeat `gem install nokogiri`.  
(For more info, see http://stackoverflow.com/questions/23661011/installing-nokogiri-on-ubuntu-debian-linux)

***Install Node.js***  
Since Rails 3.1, a JavaScript runtime has been needed for development on Ubuntu Linux. Install the Node.js server-side JavaScript environment.

```bash
$ sudo apt-get install nodejs
$ node -v --> v10.16.0
```
_Note:_ If Node.js not installed, add "gem 'therubyracer'" to each Rails app Gemfile.

***Which Database?***  
When not required to use a specific SQL database system, I use both MySQL and Postgres. SQLite is also used in local development. Instructions for installing these follow.

***Install Postgres***  
```bash
$ sudo apt-get install postgresql postgresql-contrib libpq-dev # version 9.3.10 released 10/31/2015
```

_Note 1:_ postgres now runs as a service started upon workstation boot.  
_Note 2:_ A 'postgres' role (user) has been created. 
  
***Run Client as 'postgres' User***  
```bash 
$ sudo -i -u postgres # start new shell as 'postgres' user
$ psql # start client as postgres user
# \q # exit psql
$ exit # exit postgres user shell
```

***Create a New Role (User) in Postgres***  
While Postgres has been installed with the server running as of system startup, only the 'postgres' role (aka user) has been created. You will likely need to create other roles depending upon your specific Rails project. Here's how to do it.  
_Note:_ In the below commands,   
- '#' represents the Postgres client prompt.
- Replace 'rails_user' with a role name of your choice.
  
```bash 
$ sudo -u postgres createuser -s rails_user 
$ sudo -u postgres psql # Enter Postgres client
# \password rails_user 
- Type a password of your choice & password confirmation at prompt.
# \q
```

***Install MySQL***  
_Credit:_ This section is partially based upon `https://vitux.com/how-to-install-and-configure-mysql-in-ubuntu-18-04-lts/` and `https://www.digitalocean.com/community/tutorials/how-to-use-mysql-with-your-ruby-on-rails-application-on-ubuntu-14-04`
```bash 
$ sudo apt-get install mysql-server mysql-client libmysqlclient-dev
$ mysqld --version --> mysqld  Ver 5.7.26-0ubuntu0.18.04.1 for Linux on x86_64 ((Ubuntu)) # Server
$ mysql --version --> mysql  Ver 14.14 Distrib 5.7.26, for Linux (x86_64) using  EditLine wrapper # Command line client
```

***Run Security Script (Optional)***  
MySQL as installed has some less secure defaults which are _essential_ to change in a production environment. The `mysql_secure_installation` script prompts the user to make these changes. Since I am running MySQL only in a local development environment without secure data, and I value convienience over security, I do not run this script.
```bash 
$ sudo mysql_secure_installation
```
`https://vitux.com/how-to-install-and-configure-mysql-in-ubuntu-18-04-lts/` has comprehensive instructions regarding these settings

***Check that the MySQL Server is Active***
```bash 
$ systemctl status mysql.service -->
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2019-07-20 11:16:57 CDT; 20min ago
   ... (additonal output omitted)
```   
MySQL starts automatically upon Ubuntu boot   

***Run MySQL Command Line Client & Create User***  
The MySQL installation has created a 'root' user for host 'localhost' which can be authenticated only using the 'auth_socket' plugin, and _not_ by providing a password.  (For more information, see `https://dev.mysql.com/doc/refman/5.7/en/socket-pluggable-authentication.html`)  
The root user's authentication method could be changed to use 'mysql_native_password' authentication. However, the tack employed here is to create another user for the Ruby on Rails application to use. 

```bash 
$ sudo mysql # Enter the MySQL client as the root user - 'sudo' is required
mysql> CREATE USER 'rails_user'@'localhost'IDENTIFIED WITH mysql_native_password BY 'railspassword';
-- Select your own values for 'rails_user' and 'railspassword'
mysql> use mysql # Make the 'mysql' administrative data base current
mysql> select Host, User, plugin from user; # select a small subset of the fields from the user table
--> 
+-----------+------------------+-----------------------+
| Host      | User             | plugin                |
+-----------+------------------+-----------------------+
| localhost | root             | auth_socket           |
... 
| localhost | rails_user       | mysql_native_password |
+-----------+------------------+-----------------------+
``` 

***Grant Privileges and Create Development Database***  
rails_user needs privileges to be able to do anything. Additionally, the development database needs be created before running `rails db:migrate`
```bash 
mysql> GRANT ALL PRIVILEGES ON *.* TO 'rails_user'@'localhost';
mysql> CREATE DATABASE IF NOT EXISTS `<rails project name>_development`; # Note the backtick quotes
exit # from MySQL to command line
```
*Note:* In the above, \<rails project name\> is the same as the rails project root directory name.

***Install SQLite***  
While I prefer to run the same database in the Development, Test and Production environments, SQLite is an acceptable choice for Development and Test unless non-standard DB features are used.  
Here's how to install the sqlite3 dev environment so that it will compile when 'bundle install' is run:
```bash 
$ sudo apt-get install libsqlite3-dev mysql_native_password
```

***Install Bundler and Ruby Gems***  
Bundler is the Ruby Gem manager used by most Rails projects these days.
```bash 
$ gem install bundler --> Successfully installed bundler-2.0.2
```

Print the location of all gems installed for Ruby 2.6.3
```bash 
$ gem env home 
--> 
<home dir>/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0
```

***Set Up a Rails Project to Use Postgres/MySQL/SQLite***  
You are now ready to set up your Rails project, either creating a new project using 'git init' or cloning an existing one using 'git clone'. How to do this is not described in detail here. Since the Rails (and Ruby) version may vary by project, first use 'git init/clone' to set up the project directory. 

- New project: 
Rails sets up a new project to use SQLite by default, so add the option "-d postgresql" or "-d mysql" to the 'rails new' command if using Postgres or MySQL, respectively.

- Existing project:  
1) In Gemfile: replace the gem specifying the current database (e.g. **`gem 'sqlite3'`**) with **`gem 'pg'`** (Postgres) or **`gem 'mysql2'`** (MySQL)
2) In database.yml: Change the db specifiers to use Postgres, MySQL or SQLite
3) Run 'bundle install', and recreate the DB using the 'rails db' commands. (Use 'rake db' instead if running or Rails 4 or earlier.)    
(https://coderwall.com/p/cpmdvg/rails-3-development-switch-from-sqlite3-to-pg-postgresql has more details.)

## References
> https://gorails.com/deploy/ubuntu/18.04
> https://www.rubyplus.com/articles/5901-Install-MySQL-5-7-on-Ubuntu-18-4

## Contact Me
Comments and/or corrections are welcome. Please contact me at **`edwin@edwinmeyer.com`**.

## Licensing
This README text is copyright &copy; 2017-2019 Edwin Meyer Software Engineering. 
However it may be reproduced in whole or in part if the copyright legend is included.  
_Note:_ Some code or text upon which this README is based may have different licensing terms.
