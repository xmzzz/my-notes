# How to Set, List and Remove Environment Variables in Linux
Environment variables are key-value pair in Linux which are stored permanently or temporarily to be used by applications through shell.
In this guide you are going to learn how to setup environment variables in Linux, list them and remove them.

The global environment variables are stored in etc/environment. Any changes that is made in this file reflects throughout the system for all users.

* Set Temporary Environment Variables
Temporary variables are only available to the current shell session. The variables will get deleted once you close the terminal.

You can create temporary variables using the following syntax.
```
KEY1=value
KEY2="value 2"
KEY3=value1:value2
```
- The environment variable names should be in UPPERCASE. They are case sensitive.
- The name and value pair should be separated by = sign without any spaces around it.
- Multiple values can be added to a single variable which is separated using colon:.
- The values that are having spaces should be enclosed using quotes " ".

* List Environment Variables
```
env
printenv
```

* Read Environment Variables
```
printenv HOME USERNAME
echo $HOME $USERNAME
```

* Delete Environment Variables
```
unset variablename
```
* Set Permanent Environment Variables
- /etc/environemnt: This file stores the variables that are globally accessible by all users throughout the system.
- /etc/profile: Whenever a bash shell in entered the variables in this file will gets loaded.
    To add environment variable to this file you need to use the export command.
- ~/.bashrc: User specific environment variables are added here.
    To load the added variables in your current session you need to use the source command. 
```
source ~/.bashrc
```

# Conclusion
Now you have learned how to set environment variables, list them and remove if not needed.

	-- from: https://www.cloudbooklet.com/how-to-set-list-and-remove-environment-variables-in-linux/
