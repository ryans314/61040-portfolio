# Exercise 2 - Extending a familiar concept (PasswordAuthentication)

**1. Complete the definition of the concept state.**

a set of Users with:  
&emsp;a username String  
&emsp;a password String

**2. Write a requires/effects spec for each of the two** actions.

register(username: String, password: String): (user: User)  
&emsp;**requires** a user with this username does not already exist  
&emsp;**effects** create a new User with this username and password

authenticate(username: String, password: String): (user: User)  
&emsp;**requires** a user with this username and this password exists

**3. What essential invariant must hold on the state?**

An essential invariant that must hold is that there cannot be multiple users with the same username. It is preserved by the register action requiring that a user with the same username does not already exist.

**4. Extend the concept to add email confirmation functionality for registration**

(New parts italicized)

state:
A set of users with  
&emsp;a username String  
&emsp;a password String  
&emsp;a token String  
&emsp;a confirmed Flag

actions:  

register(username: String, password: String): (user: User, *token: String*)  
&emsp;**requires** a user with this username does not already exist  
&emsp;**effects** create a new User with this username and password, *confirmed to False, and token to a random string*

*confirm(username: String, token: String): (user: User)  
&emsp;**requires** a user with this username exists with a matching token  
&emsp;**effects** sets the user's confirmed flag to True*

authenticate(username: String, password: String): (user: User)  
&emsp;**requires** a user with this username and this password exists, *and confirmed is True*
