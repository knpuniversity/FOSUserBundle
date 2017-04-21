# Customizing Text via Translations

Create a new user: `aquanaut2@gmail.com`, `aquanaut2`, `turtles`, `turtles`. Hey,
a pretty flash message!

> The user has been created successfully

***TIP
If you don't see this, make sure you're rendering `success` flash messages in
your base template: http://knpuniversity.com/screencast/symfony-forms/save-redirect-set-flash#rendering-the-flash-message
***

Well, that's message lacks some spunk! How can we change it? I mean, it's probably
being set deep in some PHP file somewhere. Do I need to override that?

Nope! Every string you see from this bundle is being passed through the translator.
So to change text, you just need to translate it!

Back in PHPStorm, let's close a few tabs. The, I'll press Shift+Shift and look for
`FOSUserBundle.en.yml`. Whenever FOSUserBundle translates something, it translates
it through a *domain* called `FOSUSerBundle`... which isn't that important... it just
means that when you translate its strings, they'll live in a file called
`FOSUserBundle.{language}.yml`.

Search for "The user has been created successfully". There it is! Under `registration`,
`flash`, `user_created`.

## Overriding the Translation

Let's breathe some life into this translation! To do that, inside `app/Resources/translations`,
create a new file: `FOSUserBundle.en.yml`. Then, put the same keys: `registration`, `flash`,
and `user_created` with the message:

> Welcome! Now let's do some science!

Since this is a brand new translation file, you'll need to clear you cache for Symfony
to see it:

```terminal
php bin/console cache:clear
```

I know, you should almost *never* need to clear cache in the dev environment: this
is one of those rare times. When you *update* this translation file, you will *not*
need to clear the cache.

Cool! Go back to /register and repeat: `aquanaut3`, `aquanaut3@gmail.com`, turtles,
turtles... and yes! Let's do some science! Great idea!

Except, instead of science, let's customize some forms!
