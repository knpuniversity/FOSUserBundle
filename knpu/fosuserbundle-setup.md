# Rock some FOSUserBundle!

The most *popular* bundle in *all* of Symfony is... [GifExceptionBundle](https://github.com/jolicode/GifExceptionBundle)!
Wait... that's not right... but it *should* be. The *actual* most popular bundle
is, of course, FOSUserBundle. And it's easy to know why: it gives you *a lot* of
crazy-cool stuff, out-of-the-box: registration, reset password, change password,
and edit profile pages. Honestly, it feels like stealing.

But guess what? A lot of smart devs don't like FOSUserBundle. How could that be?
The bundle *does* give you a lot of stuff. But, it's not a magician: it doesn't know
what your design looks like, what fields your registration form should have, the
clever text you want in your emails or where to redirect the user after registration.

See, there's a lot of stuff that makes your site *special*. And that means that if
you're going to use FOSUserBundle - which *is* awesome - you're going to need to
customize a lot of things. And you've come to the right place: once we're done,
you're going to be *embarrissingly* good extending this bundle. I mean, your co-workers
will gaze at you in amazement as you hook into events, customize text and override
templates. It's going to be beautiful thing.

Let's do it!

## Code along with me yo!

As always, you should *totally* code along with me... it's probably what the cool
kids are doing. Just click the download button on this page and unzip that guy. Inside,
you'll find a `start/` directory that has the same code you see here. Open up
`README.md` for hilarious text... and setup details.

The last step will be to find your favorite terminal and run:

```terminal
php bin/console server:run
```

to start the built-in PHP web server. Ok, load this up in your browser: `http://localhost:8000`.

Welcome to AquaNote! This is the same project we've been building in our main Symfony
tutorials, but *without* any security logic. Gasp! See that Login link? It's a lie!
It goes nowhere! The login link is a lie!

Google for the [FOSUserBundle documentation](http://symfony.com/doc/current/bundles/FOSUserBundle/index.html):
it lives right on Symfony.com. And make sure you're on the 2.0 version.

## Installing & Enabling the Bundle

We'll go through the install details... but in our own order. Of course, first,
copy the `composer require` line... but don't worry about the version number. Head
over to your terminal and run that:

```terminal
composer require friendsofsymfony/user-bundle
```

While we're waiting for Jordi to prepare out delicious FOSUserBundle package, go
back and copy the `new FOSUserBundle()` line from the docs, open our `app/AppKernel.php`
file and paste it to enable the bundle. Oh, and FOSUserBundle uses `SwiftmailerBundle`
to send the password reset and registration confirmation emails. So, uncomment that.
You can also create your own custom mailer or tell FOSUserBundle to not send emails.

[[[ code('ffc6ad0dd3') ]]]

Ok, flip back to your terminal. Bah! It exploded! Ok, it *did* install FOSUserBundle.
So, let's not panic people. It just went crazy while trying to clear the cache:

> The child node `db_driver` at path `fos_user` must be configured

Ah, so this bundle has some *required* configuration.

## Create your User Entity

Before we fill that in, I have a question: what does FOSUserBundle *actually* give
us? In reality, just 2 things: a `User` entity and a bunch of routes and controllers
for things like registration, edit password, reset password, profile and a login page.

To use the `User` class from the bundle, we need to create our own small `User`
class that extends their's.

Inside `src/AppBundle/Entity`, create a new PHP class called `User`. To extend the
base class, add a `use` statement for their `User` class with `as BaseUser` to avoid
a lame conflict. Then add, `extends BaseUser`.

[[[ code('d657ed76eb') ]]]

There's just one thing we *must* do in this class: add a `protected $id` property.
Beyond that, this is just a normal entity class. So I'll go to the Code->Generate
menu - or Command+N on a Mac - and choose `ORM Class` to get my fancy `@ORM\Entity`
stuff on top. Add ticks around the `user` table name - that's a keyword in some
database engines.

[[[ code('76caa81b23') ]]]

Now, go back to Code->Generate, choose `ORM Annotation` and select the `id` column.
Boom! We are annotated! Finally, go back to Code->Generate *one* last time... until
we do it more later - and generate the `getId()` method.

[[[ code('17ee472751') ]]]

This class is done!

***TIP
Actually, this is unnecessary - the base `User` class already has a `getId()` method.
***

## Adding the Required Configuration

And now we have everything we need to add the required configuration. In the docs,
scroll down a little: under the Configure section, copy their example config. Then,
back in your editor, open `app/config/config.yml`, and paste this down at the bottom.

Ok, The `db_driver` *is* `orm` and the `firewall_name` - `main` - is also correct.
You can see that key in `security.yml`.

And yea, the `user_class` is also correct. We're crushing it! For the email stuff,
it doesn't matter, use `hello@aquanote.com` and `AquaNote Postman`.

[[[ code('5330d3de53') ]]]

## Generate the Migraiton

*Finally*, our app should be un-broken! Try the console:

```terminal
php bin/console
```

It's alive! Now, we can generate the migration for our `User` class:

```terminal
php bin/console doctrine:migrations:diff
```

***TIP
If you get a "Command not Found" error, just install the
[DoctrineMigrationsBundle](https://knpuniversity.com/screencast/symfony-doctrine/migrations).
***

Yep, that looks about right. Run it:

```terminal
php bin/console doctrine:migrations:migrate
```

Perfect!

## Importing the Routing

At this point, the *only* thing this bundle has given us is the base `User` class...
which is nice, but nothing too special, it has a bunch of properties like
`username`, `email`, `password`, `lastLogin`, etc.

The *second* thing this bundle gives us is a bunch of free routes and controllers.
But to get those, we need to *import* the routes. Back in the documentation, scroll
down a bit until you see step 6: Import FOSUserBundle routing files. Ah ha! Copy
that routing import.

Find your `app/config/routing.yml` file and paste that on top.

[[[ code('a549b6ed16') ]]]

As *soon* as you do that, we have new routes! At your terminal, check them out:

```terminal
php bin/console debug:router
```

Awesome! We have `/login`, `/profile/edit`, `/register` and others for resetting
and changing your password. If we manually go to `/register` in the browser... yea!
A functional registration form. I know, it's *horribly*, *embarrassingly* ugly:
we'll fix that.

Oh, and back on the docs, all the way at the bottom, there's a page about
[Advanced routing configuration](http://symfony.com/doc/master/bundles/FOSUserBundle/routing.html).
Open that in a new tab. In your app, you may not need *all* of the pages that
FOSUserBundle gives you. Maybe you need registration and reset password, but you
don't need a profile page. No problem! Instead of importing this `all.xml` file,
just import the specific routes you want. Seriously feel free to do this: if some
route or controller isn't helping you, kill it.

## Enabling the Translator

Oh, and if your registration page doesn't look mine - if it has some weird keys
instead of real text, don't worry. In `app/config/config.yml`, just make sure to
uncomment the `translator` key under `framework`.

FOSUserBundle uses the translator to translate internal "keys" into English or
whatever other language.

## A *little* bit of Security

And basically... at this point... we're done installing the bundle! But how is that
possible? We haven't touched anything related to security!

Here's the truth: this bundle has almost *nothing*  to do with security: it just
gives you a `User` class and some routes & controllers! We could already register,
reset our password or edit our profile without doing any more setup.

Well, that's *almost* true: we do need a *tiny* bit of security to make registration
work. In `security.yml`, add an `encoders` key with `AppBundle\Entity\User` set
to `bcrypt`. When we register, FOSUserBundle needs to encode the plain-text password
before saving it. This tells it what algorithm to use.

[[[ code('28ceb37f8e') ]]]

There's one other small bit of security we need right now. In the documentation,
under step 4, copy the `providers` key. Paste that over the old `providers` key
in `security.yml`.

[[[ code('628eb7e8c7') ]]]

I'll talk about this more in a few minutes, but it's needed for registration only
because FOSUserBundle logs us in after registering... which is really nice!

In the next chapter, we'll do more security setup. But for right now, that's all
we need!

So try it out! Refresh `/register`. Let's use `aquanaut1@gmail.com`, `aquanaut1`,
and password `turtles`. And boom! We are registered *and* logged in! We even have
a role: `ROLE_USER`. More on that later.

With a `User` class, a route import and a *tiny* bit of security, everything in the
bundle works: registration, reset password, edit password and edit profile.

The only thing you *can't* do is log out... or login. But that's not FOSUserBundle's
fault: setting up security is *our* job. Let's do it next.
