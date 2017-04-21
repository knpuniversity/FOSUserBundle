# Dynamic Roles and Canonical Fields

I want to look at what this all looks like in the database. To query the user table,
we can use the console:

```terminal
php bin/console doctrine:query:sql 'SELECT * FROM user'
```

Nice! We inherited several columns from the base `User` class, like `username`,
`email` and `enabled`. It even tracks our last login.

## Canonical Fields!?

But there are two weird fields: `username_canonical` and `email_canonical`. What
the heck? This is one of the more controversial things about FOSUserBundle, but it's
easy to understand. First, when you set `username` or `email`, the corresponding
canonical field is automatically set for you: it's not something you need to worry
about. 

So what's the point then? Suppose that when you registered, you used some capital
letters in your username. The `username` field will be exactly as you typed it: *with*
the capital letters. But `username_canonical` will be lowercased. Then, when you
login, FOSUserBundle lowercases the submitted username and then queries the `username_canonical`
column.

Why? Because some databases - like Postgresql - are case *sensitive*. The canonical
fields allow users to login with a case *insensitive* username - `aquanaut` in all
lowercase, uppercase or any combination.

But mostly... this is just a detail you don't need to worry about. It's all handled
for you and other than being ugly in the database, doesn't hurt anything.

## Logging in with Username or Email

And by the way, right now you can *only* login with your `username`. If you want to
be able to login with `username` or `email`, no problem! The documentation has a
section about this. Just change your user provider to `fos_user.user_provider.username_email`.

What does this do? When you submit your login form, the `provider` section is responsible
for taking what you entered and finding the correct `User` record. Our current
user provider finds the `User` by the `username_canonical` field. This other one
looks up a `User` by username *or* email.

## Dynamic Roles

Check out the database query again. Look at `roles`. I know, it looks weird: this
is an `array` field. I'll hold command and click to open the base `User` class from
FOSUserBundle. See, `roles` holds an array. When you save, it's automatically serialized
to a string in the database. This is done with the Doctrine `array` field type.

Notice that even though it's empty in the database, when we login, our user has
`ROLE_USER`. This is thanks to the base `User` class from FOSUserBundle: it makes
sure our `User` has whatever roles it has in the database *plus* `ROLE_USER`.

## Creating an Admin User

Let's see an example of a `User` that has a different role. Run the console:

```terminal
php bin/console
```

Ah, so the bundle comes with a number of handy console commands, for activating,
creating, promoting and demoting users. Let's create a new one:

```terminal
php bin/console fos:user:create
```

How about `admin`, `admin@aquanote.com` and password `admin`.

Let's promote our user!

```terminal
php bin/console fos:user:promote
```

Let's promote `admin` and give them `ROLE_ADMIN`. Ok, try the query again:

```terminal
php bin/console doctrine:query:sql 'SELECT * FROM user'
```

Eureka! Our new user has `ROLE_ADMIN`! Quick, go login! Well, logout first, then
click login. Use `admin` and `admin`. Woohoo! We have *both* `ROLE_USER` and `ROLE_ADMIN`.

In your app, if you want to give different roles to your users, you can do it 2
different ways. First, at the command line with `fos:user:promote`. If you only
have a few users that need special permissions, that's pretty easy. Or, you can
create a user admin area and use the `ChoiceType` with the `'multiple' => 'true'`
option to you can select the roles from some checkboxes.

Ok, time to squash the ugly and get the FOSUserBundle pages inside our layout!
