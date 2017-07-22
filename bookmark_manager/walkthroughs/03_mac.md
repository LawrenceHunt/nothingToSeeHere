# Walkthrough - Setting up a Database

[Back to Challenge](../03_setting_up_a_database.md)

There are two ways of installing PostgreSQL. Firstly, you can download the PostgreSQL app. However, the app can sometimes be problematic, and actually leave you with a non-working PostgreSQL installation.

**We recommend the following method:**

In your terminal run `brew install postgresql`

After homebrew has downloaded the software it will show you some installation instructions, follow them!

Make sure you run these commands after installing:

```shell
ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
```
To see why that was necessary there is [information here](https://robots.thoughtbot.com/starting-and-stopping-background-services-with-homebrew)

You can check your installation by running `psql`

At first it can happen that you don't have a database named after your username (you will see a message along the lines `database "your_computer_username" does not exist`).

Let's create that database for you so that you can log in without having to specify which one: `psql postgres`

Which opens up the postgres terminal - it looks something like `postgres=#`.

Then type in the following:

```
create database "your_user_name_here";

CREATE DATABASE

postgres=# \q
```
The last command quits out of the postgres terminal.

From now on you will be able to log in to postgresql without having to specify the database you want to log into.

[next challenge](../04_creating_your_first_table.md)
