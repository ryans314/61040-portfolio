# PSET 2

## Concept Questions

**1. Contexts** 

*Question: The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?*

Contexts are for differentiating different base URL domains (e.g. tinyurl.com or bit.ly), since the same nonce can be used in different contexts (e.g. tinyurl.com/abcd and bit.ly/abcd are distinct, despite both having a nonce of "abcd").

In the URL shortening app, a context will end up being some identifier unique to the app, which will act as the prefix/shortUrlBase for the shortened URL. 

**2. Storing used strings**

*Question: Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation?*

*NonceGeneration* must store sets of used strings in order to avoid reusing a string that has already been used.

For the counter implementation, the set of used strings is the set of numbers (as strings) from 0 to the counter's current value. For example, if the count for a given context was 100, then the set of used strings would be {"0", "1", ..., "99"}. (this could change slightly depending on the exact implementaiton, which could, for example, consider used strings as [1, counter] instead of [0, counter) )

**3. Words as nonces**

*Question: One option for nonce generation is to use common dictionary words resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?*

From the user's perspective, one advantage of the scheme would be ease of use; it's a lot easier to tell someone to go to "url/computer" than "url/x1oi9Sz5," since an unintelligble randomized string is more prone to typos.

One disadvantage would be the short lifetime of the link (e.g. how long until it expires). Assuming we're only using "common dictionary words" in the english language, there are only ~500,000 words that can be used. Bit.ly reports generating ~200 million links per month, which far exceeds the number of available words, meaning that links would have to have a very short lifetime (at least if used on a platform that large). Yellkey gets around this by limiting link lifetimes to 24 hours, with 5 minutes being the default, and this system would likely need to do something similar. 

To realize this idea, I would change the generate action to the following:

generate (context: Context, lifetimeMinutes: Number) : (nonce: String)  
&emsp;**requires** lifetimeMinutes <= 1440  
&emsp;**effect** returns a nonce that is a commonly used english word and is also not already used by the context, which will expire in lifetimeMinutes minutes. 

I would also add a remove action, which would remove the url's nonce from the used strings for that context. 

## Synchronization Questions

**1. Partial matching**

*Question: In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?*

In the *generate* sync, the **then** clause only requires shortUrlBase in order to use NonceGeneration.generate. Since targetUrl is not required for the action invocation, it is not necessary to include it in the action completion in the **when** clause.

In the *register* sync, since targetUrl is required in the UrlShortening.register action invocation, it must be included in the Request.shortenUrl action completion. 

**2. Omitting names**

*Question: The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?*

The name omission convention isn't used in every case since it wouldn't make sense in a lot of cases to name the output of an action completion the same as the input to the action invocation. 

For example, in the *setExpiry* sync, the output of UrlShortening.register is named shortUrl, since that is a descriptive name for that return value. It wouldn't make sense to name it "resource" in the **when** clause, since UrlShortening.register doesn't *just* return a resource, it returns a shortUrl. This explains why ExpiringResource.setExpiry had to bind shortUrl to the resource variable, rather than using the shorthand for resource: resource.  

**3. Inclusion of request**

*Question: Why is the request action included in the first two syncs but not the third one?*

In the first two syncs, Request is used to represent a broad application request to shorten a URL, an action that is not associated with only a single concept. In the third sync, the action invocation happens in direct response to the action completion of another concept's action. Regardless of what the user is trying to do (the Request), whenever UrlShortening.register is completed, ExpiringResources.setExpiry should be invoked according to the *setExpiry* sync's instructions. 

**4. Fixed domain**

*Question: Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?*

If the domain name is always fixed, then I would simplify the *generate* and *register* syncs by removing shortUrlBase from both **when** clauses, and replacing shortUrlBase with "bit.ly" (or whatever the fixed domain name is) in both **then** clauses. (for the *register* sync, it would technically be replaced with shortUrlBase: "bit.ly")

**5. Adding a sync**

*Task: These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.*

**sync** expire  
**when** ExpiringResource.expireResource (): (resource)  
**then** UrlShortening.delete(shortUrl: resource)

**Note**: it would also be good to free up the nonce for the appropriate context in the NonceGeneration concept, but the question only asks for ExpiringResource and UrlShortening concepts (plus the actions to remove a string from the used set don't exist yet).

## Extending the design

*1. Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state). It should not be necessary to make any changes to the existing concepts.*

**concept** UserOwnership [User, Resource]  
**purpose** track what resources a user owns  
**principle** after a user is created, any resources they create are tracked and tied to their user  
**state**  
a set of Users with:  
&emsp;a set of Resources  
**actions**  
addUser(user: User)  
&emsp;**requires** user exists  
&emsp;**effect** adds user to the set of Users with an empty set of resources

addResource(user: User, resource: Resource)  
&emsp;**requires** resource exists and user is in the set of Users  
&emsp;**effect** if user is not in the set of Users, add the user with an empty set of resources.  

**note**: this concept does not define user creation, since that should be done with some sort of username/password authentication concept. 

**concept** UsageAnalytics \[Resource]  
**purpose** keep track of how many times a resource has been accessed  
**principle** after a resource is created, each access can be tracked, and analytics can be retrieved  
**state**  
a set of Analytics with:  
&emsp;a counter Number  
&emsp;a resource Resource  
**actions**  
createAnalytics(resource: Resource): (analytics: Analytics)  
&emsp;**requires** resource exists and is not already part of another Analytics object  
&emsp;**effect** creates a new Analytics object with resource = resource and counter = 0

accessResource(resource: Resource)  
&emsp;**requires** there exists an Analytics with the given resource  
&emsp;**effect** increments the counter on the associated Analytics object

retrieveAnalytics(resource: Resource): (count: Number)  
&emsp;**requires** resource exists and there is an Analytics object associated with the resource  
&emsp;**effects** returns the associated Analytics' counter value.

**Notes**: We also assume that there is not a use for deleting Analytics, since users may want to view analytics for since-expired resources. Also, authentication for analytics is handled in the sync

*2. Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics. (Hint: for the last one, you can simplify by assuming a request for analytics that just asks for the number of lookups for a particular short URL, but that ensures that the result is not visible to everyone.)*

**sync** createShortening  
**when**  
&emsp;Request.shortenUrl (user)  
&emsp;UrlShortening.register (): (shortUrl)  
**then**  
&emsp;UserOwnership.addResource (user, resource: shortUrl)  
&emsp;UsageAnalytics.createAnalytics (resource: shortUrl)

**note**: assumes that the user has already been created, since the problem asks to only specify 3 syncs

**sync**: accessShortening  
**when** UrlShortening.lookup (shortUrl)  
**then** UsageAnalytics.accessResource (resource: shortUrl)

**sync**: retrieveAnalytics  
**when**: Request.requestAnalytics (user, resource)  
**where**: UserOwnership: user owns the resource (resource is in user's resource set)  
**then**: UsageAnalytics.retrieveAnalytics (resource)


*3. As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:*

- Allowing users to choose their own short URLs;
    - (assuming "choose" means to pick the suffix,) This could be realized by changing NonceGeneration's generate action to have an optional parameter "chosenNonce: String", which would be returned iff not already used by the given context. 
- Using the “word as nonce” strategy to generate more memorable short URLs;
    - This could be realized by changing NonceGeneration.generate's effect to limit nonces to only dictionary words. 
- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;
    - This would not be desirable, and should not be included. Viewing analytics for all short URLs that point to a given target URL would allow users to view other user's short URL analytics (which they do not have access to), which would be a privacy/security risk. A better feature would be to allow users to request aggregated analytics for any group of shortUrls which they have ownership/access to.
- Generate short URLs that are not easily guessed;
    - This could be easily realized by changing NonceGeneration.generate's effect to return only nonces that meet certain "difficulty" criteria, e.g. being at least 16 characters or having a mix of letters and numbers
- Supporting reporting of analytics to creators of short URLs who have not registered as user.
    - This would also be undesirable and should not be included. If creators have not registered for an account, then their identity cannot be verified in order to view their shortURL analytics. As a result, anyone that wasn't signed in could view all short URL analytics made by un-registered creators (a major privacy violation).
    
