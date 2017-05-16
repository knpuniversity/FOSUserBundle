## TargetPathTrait: Redirect to Previous Page

We now have control over where the user goes after registering. But... it's not as
awesome as it could be. Let me show you why.

First, look at my `app/config/security.yml` file. In order to access any URL that
start with `/admin`, you need to be logged in. For example, if I go to `/admin/genus`,
it sends me to the login page.

Thanks to Symfony's `form_login` system, if we logged in, it would automatically
redirect us *back* to `/admin/genus`... which is awesome! That's clearly where the
user wants to go.

But what if I instead clicked a link to register and submitted that form? Shouldn't
that *also* redirect me back to `/admin/genus` after? Yea, that would be *way* better:
I want to keep the user's experience really smooth.

How can we do this?

## Using TargetPathTrait

Guess what? It's almost *effortless*, thanks to a trait that was added in Symfony
3.1: `TargetPathTrait`. In your subscriber, `use TargetPathTrait`. Then, down in
`onRegistrationSuccess`, add `$url = $this->getTargetPath()` - a method provided
by that trait. Pass this `$event->getRequest()->getSession()` and for the "provider key"
argument, pass `main`. Provider key is a fancy term for your firewall's name.

[[[ code('c533e7c3cb') ]]]

What's going on here? Well, whenever you try to access a secured page anonymously,
Symfony stores that URL somewhere in the session before redirecting you to the login
page. Then `form_login` uses that to redirect you after you authenticate. The `TargetPathTrait`
is just a shortcut for *us* to read that *same* key from the session.

If `$url` is empty - it means the user went directly to the registration page. No
worries! Just send them to the homepage.

[[[ code('da633d851c') ]]]

Let's try the *entire* flow. I'll go back to `/admin/genus`: it redirects me to
the login page and sets that session key behind the scenes. Then, I'll manually
type in `/register` - but pretend like we clicked a link. Register as `aquanaut6`,
password `turtles`.

Booya! Logged in *and* on the `/admin/genus` page. That's a kick butt registration
form.
