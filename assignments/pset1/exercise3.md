# Exercise 3 - Comparing Concepts

## Personal Access Token Concept

**concept** PersonalAccessToken \[User, Scope, Date\]

**purpose** provide additional authentication options to known users

**principle** an existing user creates a PersonalAccessToken with specific scopes and expiration dates;  
the user can use the PAT to access resources according to the PAT's scopes until the expiration date, after which it will no longer work.

**state**  
a set of PersonalAccessTokens with  
    a user User  
    a value String  
    an expiration Date  
    a set of Scopes


**actions**  
create(user: User, expiry: Date, scopes: List\[Scope\]): (pat: PersonalAccessToken)  
&emsp;**effects** creates a personal access token associated with user, expiring on expiry, with the scopes in scopes, where value is a random 40 character string, and returns the personal access token

delete(pat: PersonalAccessToken)  
&emsp;**requires** that pat is an existing PersonalAccessToken  
&emsp;**effects** removes pat from the set of PersonalAccessTokens

authenticate(user: User, date: Date, pat: String, scope: Scope)  
&emsp;**requires** pat corresponds to a PersonalAccessToken where pat.user and user are equal, and date is before pat.expiration, and the scope is in pat.scopes

## Concept Questions

**How does PersonalAccessToken differ from PasswordAuthentication?**

PasswordAuthentication creates a one-to-one association of a user and a password, which is used to authenticate everything related to that user. PersonalAccessToken allows users to have multiple, expirable PATs, with different Scopes (sets of allowed categories that it can authenticate).

**How might you change the GitHub documentation to explain this?**

I would include a brief section in the introduction for "Personal Access Tokens (classic) vs Passwords," with a table showing the differences. 