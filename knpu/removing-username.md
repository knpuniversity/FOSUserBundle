# My Users don't have a Username!

Okay: a challenge! In some apps... you don't even *need* a username - you register
and login entirely with your email.

But... with FOSUserBundle, we *always* have a username field. So, are we stuck?

Nope! It *is* true that you can't remove the `username` field from your `user` table.
But, we *can* remove it from everywhere else.

## Auto-Setting the Username Field

Start in the `User` class. I'll go to the Code->Generate menu - or Command+N on a
Mac - go to "Override Methods" and choose `setEmail()`. Before the parent call,
add `$this->setUsername($email)`.

[[[ code('e3befd9c09') ]]]

This is big! Thanks to this, we no longer need to worry about *ever* setting the
`username` field. In the database, `username` will always equal the `email`... which
is definitely redundant and unnecessary, but in practice, it works fine.

## Removing the Username Field from the Form

The only other thing we need to do is remove the `username` field from the registration
form. How? Easy: in `RegistrationFormType`, add `->remove('username')`.

[[[ code('c637e72a3e') ]]]

Then, in the template, remove its `form_row()`.

[[[ code('a6d3131d33') ]]]

And yea, that's it! Refresh! No more username! Register as `aquanaut4@gmail.com`
and submit. Check it out in the database:

```terminal
php bin/console doctrine:query:sql 'SELECT * FROM user'
```

There it is! The `username` is now equal to the email.

Oh, and if you're using the profile edit page from the bundle, you'll also want to
remove the `username` field from the `ProfileFormType` form. It's done in exactly
the same way.
