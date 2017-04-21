# Dynamic Roles and Canonical Fields

Let's see what this all looks like in the database! To query the user table,
we canactually use the console:

```terminal
php bin/console doctrine:query:sql 'SELECT * FROM user'
```

Nice! We inherited a bunch of columns from the base `User` class, like `username`,
`email` and `enabled`. It even tracks our last login. Thanks!

## Canonical Fields!?

But there are two weird fields: `username_canonical` and `email_canonical`. What
the heck? These are one of the more controversial things about FOSUserBundle. Before
we explore them, first, just know that when you set `username` or `email`, the corresponding
canonical field is automatically set for you. So, these canonical fields are not
something you normally need to worry or think about.

So why do they exist? Suppose that when you registered, you used some capital
letters in your username. The `username` column will be exactly as you typed it:
*with* the capital letters. But `username_canonical` will be lowercased. Then, when
you login, FOSUserBundle lowercases the submitted username and queries via the
`username_canonical` column.

Why? Because some databases - like Postgresql - are case *sensitive*. The canonical
fields allow a user to login with a case *insensitive* username - `aquanaut` in all
lowercase, uppercase or any combination.

But mostly... this is just a detail you shouldn't think about. It's all handled
for you and other than being ugly in the database, it doesn't hurt anything.

## Logging in with Username or Email

And by the way, right now you can *only* login with your `username`. If you want to
be able to login with `username` or `email`, no problem! The documentation has a
section about this. Just change your user provider to `fos_user.user_provider.username_email`.

What does this do? When you submit your login form, the `provider` section is responsible
for taking what you entered and finding the correct `User` record. Our current
user provider finds the `User` by the `username_canonical` field. This other one
looks up a `User` by username *or* email. And you're 100% free to create your *own*
user provider, if you need to login with some other, weird logic. FOSUserBundle
won't notice or care.

## Dynamic Roles

Check out the database result again and look at `roles`. I know, it's strange: this
is an `array` field. I'll hold command and click to open the base `User` class from
FOSUserBundle. See, `roles` holds an array. When you save, it's automatically serializes
to a string in the database. This is done with the Doctrine `array` field type.

Notice that even though it's empty in the database, when we login, our user has
`ROLE_USER`. This is thanks to the base `User` class from FOSUserBundle: it makes
sure the `User` has whatever roles are stored in the database *plus* `ROLE_USER`.

## Creating an Admin User

Let's try an example of a `User` that has a different role. Run the console:

```terminal
php bin/console
```

Ah, so the bundle comes with a few handy console commands, for activating, creating,
promoting and demoting users. Let's create a new one:

```terminal
php bin/console fos:user:create
```

How about `admin`, `admin@aquanote.com` and password `admin`.

And now promote it!

```terminal
php bin/console fos:user:promote
```

Hmm, let's give `admin`, `ROLE_ADMIN`. Ok, try the query again:

```terminal
php bin/console doctrine:query:sql 'SELECT * FROM user'
```

Booya! Our new user has `ROLE_ADMIN`! Quick, go login! Well, logout first, then
go login! Use `admin` and `admin`. Woohoo! We have *both* `ROLE_USER` and `ROLE_ADMIN`.

In your app, if you want to give different roles to your users, you have 2 options.
First, via the command line by using `fos:user:promote`. If you only have a few users
that need special permissions, this is a great option. Or, you can create a user
admin area and use the `ChoiceType` with the `'multiple' => true` and `'expanded' => true`
to select the roles as checkboxes.

Ok, time to squash the ugly and make the FOSUserBundle pages use our layout!
