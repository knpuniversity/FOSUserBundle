# FOSUserBundle <3's Guard Authenticators

We *now* understand that FOSUserBundle *just* gives us a nice `User` class and some
routes & controllers for registration, reset password, edit profile and a few other
things. The bundle does *not* provide any authentication. Open `app/config/security.yml`.
The `form_login` authentication mechanism we're using is core to Symfony itself,
not this bundle.

So, one of the questions we get a lot is: how can I use Guard authentication with
FOSUserBundle? It turns out, it's simple! Guard authentication and FOSUserBundle
solve different problems, and they work together beautifully. Teamwork makes the
dream work!

But, why would you want to use Guard authentication with FOSUserBundle? Well, as easy
as `form_login` is, it's a pain to customize. Guard is more work up front, but gives
you a lot more control. You can also use Guard to add some sort of API authentication
on top of `form_login`.

## Creating the Authenticator

Let's replace `form_login` with a more flexible Guard authenticator. At the root
of our project, you should have `tutorial/` directory with a file called `LoginFormAuthenticator.php`.
In `src/AppBundle`, create a new directory called `Security` and paste that file
here.

[[[ code('46305e4007') ]]]

This `LoginFormAuthenticator` is almost an exact copy of the authenticator we created
in our [Symfony Security tutorial](http://knpuniversity.com/screencast/symfony-security).
I've just added CSRF token checking - since our HTML login form has a CSRF token
in it - and made a few other minor tweaks. For example at the bottom, I updated the
login route name to use the one from FOSUserBundle.

The authenticator is very straightforward: It looks for the submitted `_username`
and `_password` fields from the login form. It doesn't care if you built that login
form yourself, or if it comes from FOSUserBundle. Then, it queries for your `User`
object by email only and checks to see if the password is valid. Obviously you can
write your authenticator to do anything.

## Registering the Authenticator

To get this to work, like all authenticators, we need to register it as a service.
I'll add `app.security.login_form_authenticator`, set the class to `LoginFormAuthenticator`
and use `autowire: true`.

[[[ code('a458eb6e23') ]]]

Copy that service ID. Then open `app/config/security.yml`. Ok, let's comment-out
`form_login` entirely. And instead, add `guard`, `authenticators`, then paste the
service ID.

[[[ code('9efbd45740') ]]]

That's it! FOSUserBundle doesn't care who or what is processing the login form submit.

Let's try it! Click log out, click login and login with admin@aquanote.com. Yea,
this *does* still say "Username", but we know that our authenticator actually logs
us in via email. So, we'll want to tweak that language. Use the password `admin`
and... boom!

Congrats! You just used a Guard authenticator with FOSUserBundle. Wasn't that nice?
You should feel empowered to use FOSUserBundle because you want things like a registration
page or reset password system. But, you can *still* take control of your actual login
mechanism and do whatever the heck you want.

The last part of this bundle that you'll need to customize are the emails: the reset
password email and the registration confirmation email, if you want to send that one.
The docs are good on this topic, and it's mostly a matter of overriding templates...
which we already mastered.

All right guys, go use FOSUserBundle to quickly bootstrap your site! As long as you
understand what it does... and does *not* give you, it's awesome. Seeya next time!
