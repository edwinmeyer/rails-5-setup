# Basic Setup for Ruby on Rails 5 Development
## using Git, Rbenv, Bundler, Postgres
### *Edwin W. Meyer*

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
$ cd ~
```

***Update the Package Manager***  
Update list of available packages &ndash; advisable prior to any apt installation.  
```bash
$ sudo apt-get update 
```

***Install Git***  
```bash
$ sudo apt-get install git
$ git --version --> git version 2.7.4 # No problem if you have a newer version
```

***Remove Ruby Version Manager (rvm)***  
We will be using the "Ruby environment manager" (rvm) &ndash; see below, which conflicts with rbenv. If you already have rvm installed, remove it as follows:
```bash
$ rvm implode
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
$ rbenv -v --> rbenv 1.1.1-2-g615f844
```

***Install ruby-build***  
ruby-build is an rbenv plugin that provides an 'rbenv install' command to compile and install different versions of Ruby on UNIX-like systems.  
```bash
$ git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

***Install Ruby 2.4.1***  
List the available Ruby versions. (2.4.1 is the latest stable version as of 9/16/2017)
```bash
$ rbenv install -l
```

Install a specific listed Ruby version:  
```bash
$ rbenv install 2.4.1
 --> 
 Installed ruby-2.4.1 to <home dir>/.rbenv/versions/2.4.1
```

- You may get the error message  
ERROR: 'Ruby install aborted due to missing extensions'  
If so, perform per the error message:
```bash
$ sudo apt-get install -y libssl-dev libreadline-dev zlib1g-dev
Then repeat:
$ rbenv install 2.4.1
```

Post-install tasks:
```bash
$ rbenv rehash # Update the 'shims' used to invoke a specific command version
$ rbenv global 2.4.1 # use this Ruby if not specified in project directory
$ ruby -v --> ruby 2.4.1p111
```

- What are "shims"?  
rbenv sets up shims for each ruby version. A shim is a short bash script that determines which version of a command is to be used and executes it when the command is invoked.  For example, the shim for the bundle command is located at **`~/.rbenv/shims/bundle`**  
Unlike gems, which can have per-project versions (`bunder` handles this), a single command version serves all projects that use a particular Ruby version.  

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
```
_Note:_ If Node.js not installed, add "gem 'therubyracer'" to each Rails app Gemfile.

***Which Database?***  
When not required to use a specific SQL database system, I prefer Postgres because until recently it has been superior to other open source alternatives. It is also the database used in Heroku. 

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
- Replace 'new_postgres_user' with a role name of your choice.
  
```bash 
$ sudo -u postgres createuser -s new_postgres_user 
$ sudo -u postgres psql # Enter Postgres client
# \password new_postgres_user 
- Type password & password confirmation at prompt.
# \q
```

***Install SQLite***  
While I prefer to run the same database in the Development, Test and Production environments, SQLite is an acceptable choice for Development and Test unless non-standard DB features are used.  
Here's how to install the sqlite3 dev environment so that it will compile when 'bundle install' is run:
```bash 
$ sudo apt-get install libsqlite3-dev 
```

***Install Bundler and Ruby Gems***  
Bundler is the Ruby Gem manager used by most Rails projects these days.
```bash 
$ gem install bundler
```

- Print the location of all gems installed for Ruby 2.4.1
```bash 
$ gem env home 
--> 
<home dir>/.rbenv/versions/2.4.1/lib/ruby/gems/2.4.0
```

***Set Up a Rails Project to Use Postgres***  
You are now ready to set up your Rails project, either creating a new project using 'git init' or cloning an existing one using 'git clone'. How to do this is not described in detail here. Since the Rails (and Ruby) version may vary by project, first use 'git init/clone' to set up the project directory. 

- New project: 
Rails sets up a new project to use MySql by default, so add the option "-d postgresql" to the 'rails new' command.

- Existing project: 
1) In Gemfile: replace the gem specifying the current database (e.g. **`gem 'sqlite3'`**) with gem **`'pg'**
2) In database.yml: Change the db specifiers to use Postgres
3) Run 'bundle install', and recreate the DB using the 'rails db' commands. (Use 'rake db' instead if running or Rails 4 or earlier.)   
(https://coderwall.com/p/cpmdvg/rails-3-development-switch-from-sqlite3-to-pg-postgresql has more details.)

## Contact Me
Comments and/or corrections are welcome. Please contact me at **`edwin@edwinmeyer.com`**.

## Licensing
This README text is copyright &copy; 2017 Edwin Meyer Software Engineering. However it may be reproduced in whole without modification other than reformatting if the copyright legend is included.
