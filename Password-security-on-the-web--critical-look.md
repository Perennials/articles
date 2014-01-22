Password security on the web - critical look
============================================

I always feel outraged when I sign up for a service and they send me
confirmation email with my password in clear text! I thought of writing this
article because I want to scream to the world DON'T STORE MY PASSWORD. My
password is my own, even if a service provider is perfectly secure it is a
matter of trust. "We don't ask you for your password, we don't need to know
your password, we understand you don't know us and don't trust us, we
understand your password may be key to a lot in your life, we respect your
privacy so much that we don't event want you to trust us by giving us this key
since we will under no circumstances in the world use it, and since we will
not use it EVER, we don't need you to send it to us".

### My password should never leave the browser
It is common sense today that the backends shouldn't store passwords in clear
text but as cryptographic hashes. But even with modern services which don't
store your password, it should never leave the browser. Why? Because in theory
if someone is tapping the wire he can gain access to your traffic and thus
your password. And if this is the password to my email, he could then gain
access to all my other passwords and much more than that - PayPal, credit
cards, bank accounts, etc. I recently was curios and inspected what is sent to
Google's servers upon login and there it is... my password sent as clear text
to the server on every login. Why? __All you need to do is convert the
password to hash code before sending it to the server!__ It takes two extra
lines of code on the frontend part and requires absolutely no modifications on
the backend part. Ideally every service will add something unique to the
password before hashing it, so even if an attacker (agent) acquires the hashed
password, he will not be able to use it with other services. In this case the
only theoretical vulnerability would be to reverse the hash sent to the
server, find the original password and use it with other services. But this is
really complicated and in some cases impossible. __Yet noone is doing it__ and
I'm the only person (that I'm aware of) that is hashing the passwords in the
frontend before sending them to the server. Although I must admit I didn't
think of adding something unique the hash before writing this article. __It is
common sense to treat other's passwords the way you want your password to be
treated.__ And unfortunately there are still many older services just
conveniently storing passwords in clear text and offer password reminders to
the users. I had an argument with a hosting provider why they do this and why
they store my password. They basically said, although not in the exact same
words, "because some customers are so dumb, they can't use the password
recovery so we need to send them the password". They promised to not store my
password, but of course I didn't believe they will tweak their system to treat
my password differently.

I want to take this article further. There is a lot of talk recently about
Internet privacy in the context of surveillance agencies. I once had the task
of making a system which had to transfer sensitive data securely over the
Internet. I started analyzing

### Is it possible to transfer the data in a such a way that it is impossible for unauthorized parties to access it?
And if we accept the scenario where someone is tapping the wire, which back
then I did because I was interested in conspiracy theories at the time, we
have real problem. Why? Lets assume that we can encrypt the data and it will
be impossible or unfeasible for the attackers to decrypt it. So first we need
to choose encryption algorithm. This in itself is not easy, because you can
never be sure how secure they are. Even if you are some kind of genius who can
actually understand the algorithm you can never be sure that there are not
holes that you didn't think of. Not to mention that many of these algorithms
are invented by the guys we want to protect ourselves from, how can one be
sure they didn't leave a backdoor? But lets say we investigate and choose
encryption algorithm. How do you make your two remote programs talk to each
other in secret? If you have random key for the encryption, then you are not
able to decrypt it on the other side. And if you send the key over the wire
and someone is tapping it he could use it to decrypt your message. This means
we can't have security that works in all cases. For example we couldn't write
a secure web server that we distribute as open source. Because once the way it
communicates is known by someone having access to the Internet traffic, he
could apply the same technique used by the software to decrypt the traffic. It
will be more complicated for him, which is better than nothing, but these guys
are not short on resources. So we send encrypted data and we can't send the
key over the network nor have completely random key. What are we left with? We
could have a key that only the two remote apps know about. This way the only
way for the attacker to acquire the key would be if he had access either the
server or the client software so it can be reverse engineered. This technique
is not easily applicable on mass scale though. For example if I run some kind
of web service and I want to offer it to many clients every client needs to
have his own client-software with his personal password, I couldn't have one
client-software for all my clients, otherwise an attacker could pretend to be
a client, acquire a copy of the client-software and reverse engineer it to
gain access to the key for the whole system. But it gets worse. Having
personal software for each client is some kind of OK, it is can be done
automatically and this technique can be applied to a service where we offer
personal data to the client, so this data is personally encrypted with his own
key. But what if we have to offer the same data to many clients and they are
using different keys we need to re-encrypt the data for each client and send
him his personalized crypt-stream. Obviously we coudn't keep a copy of the
whole Internet personally encrypted for everyone.

### So the next time you are told about security
think about if your service provider asked you to make up a personal password,
enter it in their client personally, and also bring it personally on a USB
stick to their office so they can plug it into their server that you will be
communicating with.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)