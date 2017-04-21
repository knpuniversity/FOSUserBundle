# Security Setup

With FOSUserBundle setup, the only things we *can't* do is login and logout. FOSUserBundle
*does* give us a `/login` page, but it's just a static HTML form: setting up the
actual authentication part is *entirely* up to us. And that's why, if you try to
login now, you get this angry message!

To prove how little FOSUserBundle is doing, go to `/login`. If you hover over your
web debug toolbar, you can see that the controller behind this page is called `SecurityController`.
Cool! In my editor, I'll find that file by filename - Shift+Shift in PHP Storm.

Sweet! See `loginAction`? This renders the login page. And *all* it does is check
for any authentication errors stored in the session and render a template. It has
*no* logic *whatsoever* for processing the form submit, logging in, or anything
else related to security.

## Configuring Logout

So let's *finally* add some security goodness, starting with logging out. Right now,
if you go to `/logout`, you see an error message. This is coming from that same
controller: FOSUserBundle gives us a `/logout` route, but its controller is never
supposed to be called. To fix this, in `security.yml`, add `logout: ~`. That's it.

Try going to `/logout` again. It works! We are anonymous! By adding the `logout`
key, Symfony is now waiting for us to go to `/logout`. When we do, it *intercepts*
the request and logs us out. Other than giving us the `/logout` route, FOSUserBundle
has nothing to do with this.

## Configuring form_login Security

What about logging in? It's the same thing. Under your firewall, add `form_login`.
That's actually all you need. But, I'll add a bit more: `csrf_token_generator: security.csrf.token_manager`.
That will make sure the CSRF token - which is already added in the FOSUserBundle
login template - is verified when we submit.

As *soon* as we do that, go to `/login` and login with `aquanaut1` password `turtles`.
Winning! We are in! FOSUserBundle gives us a login form, but *we* need to take
care of the rest... which is pretty easy.

## Adding Remember Me Functionality

Oh, and on the login form, we also have a remember me checkbox. If you want this
to work, you'll need to add one more setting: `remember_me:` with `secret: '%secret%'`
to use the secret from `parameters.yml`.

Ok, so about 5 lines to get our entire security system working. That kicks butt!
And now, we can hook up the login link for real. Open `app/Resources/views/base.html.twig`
and find the static link. Add an if statement: `if is_granted('ROLE_USER')`, then
`else` and `endif`. FOSUserBundle guarantees that every user always at least
`ROLE_USER`. So it's safe to use this to figure out whether or not the user is logged
in.

For the logout link, use the route `fos_user_security_logout`, then we'll say "Logout".
Oh, put all of this stuff inside an `li` tag. If you run:

```terminal
php bin/console debug:router
```

you can see that this is one of the routes we imported. Use a similar one for login:
just copy the logout line, and change it to login.

Nice! Go back, and refresh. Hit Logout! Woohoo! Let's see what this looks like in
the database, and talk about roles.
