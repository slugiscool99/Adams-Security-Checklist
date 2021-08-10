# Adam's Security Checklist

Hackers love startups, because usually their systems aren't properly secured. After spending time building fintech MVPs, I decided to write down basic security considerations. Hopefully this can provide a reference on easy ways to prevent your application from being hacked. I am by no means a smart hacker, and these methods will not dissuade a motivated smart opponent. These will however make it more difficult for the majority of less intelligent fraudstrs. In my opinion, this should be the bare minimum you do when releasing an MVP (especially if you're planning on handling money or other sensitive data). I'll be updating this list as I go with everything I learn.

**These methods are not sufficient to prevent a motivated hacker from gaining access to your systems.** “You don’t have to run faster than the bear to get away. You just have to run faster than the guy next to you.”

## Table of contents
[APIs](#api)

[Apps](#app)

[ACH & Money](#ach)

<a name="api"/>

## APIs
These are absolute basics you need to think about when building an API

### Client requests
Never make sensitive requets (especially those involving secrets or API keys) from the client. Always go through the server and assume any API exposed to the client can be programatically accessed by anyone. Don't make database changes directly from the client, have that go through an endpoint first. Make sure you log everything so you can detect suspicious activity.

### Authentication
Use a third party authentication provider if possible. If you have to use your own authentication, use JWT tokens and use a popular reputable library to manage them. Choose a good secret to prevent [brute forcing](https://github.com/brendan-rius/c-jwt-cracker). If you use phone numbers, [check if the number is virtual](https://www.twilio.com/lookup). If you use emails, validate the email (it's not a disposable domain, it's deliverable, etc). 

### Authorization (IDOR issues)
Ensure that an authenticated user only has access to their own resources. Using a UUID as an object identifier and hoping users won't be able to guess other users' resource names is not a valid method of preventing unauthorized access.

### Race conditions (idempotency)
Make sure you've thought about the possibility of simultaneous requests. For example, if a user requests a 5 discounts at the same time (`POST /discount/:token`) they should only recieve one discount!

### Rate limiting
You should be rate limiting to prevent brute forcing and malicious programatic access. You should probably be doing this for each account and ip address. (e.g. Only allow 1 request / sec per account and 50 requests / sec per IP address. Be careful of limiting IP addresses that may be shared by multiple clients, like schools or libraries.)

### Prevent Cross-site Request Forgery (CSRF)
You need to prevent hackers from accessing your API through other domains. You need to prevent a hacker from tricking one of your users into submitting a form that makes a request like: `POST https://bank.com/transfer {acct: 12345, amount: 100000}`. Use a [SameSite cookie](https://portswigger.net/web-security/csrf/samesite-cookies) to prevent this.

### Endpoint naming
Try not to have commonly used names for API endpoints, especially ones you don't want people to access programatically. Many undocumented endpoints can be found by just bruteforcing requets with [wordlists](https://wordlists.assetnote.io/). Don't reveal too much in your API errors (e.g. `/users is not a valid route. Valid routes are /api/users or /api/posts`). 

<a name="app"/>

## Apps
Apps are a little easier since the client code is somewhat more difficult to access. However, competent hackers still can with tools like [this](https://rada.re/).

### SSL Pinning
Your application must send requests to your server. These are most likely encrypted with SSL, which is good to keep others from sniffing the request. However, this doesn't prevent someone from sniffing their own requests through an SSL proxy (using something like [Charles Proxy](https://www.charlesproxy.com/)). Basically, any user can view the entirety (URL, body, headers, etc) of any request your app makes. They can then use that to make their own requests. In order for a user to successfully do this, they need your app to trust a proxy SSL server. However, you can prevent this from happening with *SSL pinning*. 

There are various libraries for [iOS](https://github.com/datatheorem/TrustKit) or [Android](https://github.com/datatheorem/TrustKit-Android) that can do this for you. Or, you can [do it yourself](https://www.raywenderlich.com/1484288-preventing-man-in-the-middle-attacks-in-ios-with-ssl-pinning). 

This cuts off one easy way for a user to figure out what your APIs are on apps. If the user jailbreaks or roots their device, it won't help. Also, if you have a web application this doesn't work.

### Obfuscation
You should probably obfuscate your production code as much as you can. There are various free tools that can help with this instantly ([javascript](https://obfuscator.io/), [swift](https://github.com/rockbruno/swiftshield), etc). This adds a little bit of hassle to anyone looking into your code, but not much.

### Making keys difficult to access
Assume that any keys on the client will be discoverable. **Don't store keys that matter on the client** However, using a library like [this](https://github.com/orta/cocoapods-keys) for iOS or [this](https://github.com/nomtek/android-client-secrets) for Android can make it more difficult for a hacker to find a secret. You can use these hidden keys to further obfuscate requests (for example, using one as a public key to encrypt request payloads). Again, this just makes things harder for a hacker to understand and there's no real way to do this with a web app. 


<a name="ach"/>

## ACH & Money
The ACH network is prone to fraud. Here are things to consider

### Stolen KYC info
Many fraudulent users will submit stolen information for KYC, which will pass. That's why it's not enough just to rely on KYC to prevent fraud. You need to be checking 

### Check their balance
It's a good idea to check the user's balance before initiating an ACH pull (with something like Plaid). This is a basic prevention against getting a NSF (insufficient funds) return code. You also might want to check transaction history to make sure the account is real and has been operating normally for some time.

### Fraudulent returns
Anyone can call their bank and claim an ACH pull was fraudulent. Collect as much information (application logs, user information, etc) as ammo to prove you were a legitimate transaction.

### Bait-and-switch
A user can initiate an ACH request when they have money in their bank, and then withdraw the funds before the ACH request completes. If you give users access to their money before the ACH request completes and they spend that money, you run the risk of hitting an empty account and being left with nothing.

### Burner cards
If you don't charge a card before providing a service, you run the risk that the card has an artificial limit (using a service like [Privacy](http://privacy.com/)). If you absolutely must charge later, you can check if a card is physical/virtual/prepaid and act accordingly. Most legitimate consumers will have a real physical card and you can probably get away with disallowing all virtual/prepaid cards if you're in a risky industry. 

