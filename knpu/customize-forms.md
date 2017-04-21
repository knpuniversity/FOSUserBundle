# Customizing the Forms

What if we wanted to add our own fields to the registration form? Like, what if
we needed to add a "First Name" field? No problem!

## Adding a firstName field to User

Let's start in our `User` class. This is a normal entity, so we can add whatever
fields we want, like `private $firstName`. I'll go to Code->Generate, or Command+N
on a Mac, then select "ORM Annoations" to annotate `firstName`. I'll go back to
Code->Generate to add the getter and setter methods.

Now notice, right now, `firstName` is *not* nullable... and that's fine! If `firstName`
is optional in your app, you might want to add `nullable=true`. I'm mentioning
this for one specific reason: if you add any required fields to your `User`, the
`fos:user:create` command will *no* longer work... because it will create a new
`User` but leave those fields blank. I never use this command in production anyways,
but I wanted to warn you!

Move over to your terminal to generate the migration:

```terminal
php bin/console doctrine:migrations:diff
```

That looks correct! Run it:

```terminal
php bin/console doctrine:migrations:migrate
```

New field added! So how can we add it to this form? Simple! We can create our *own*
new form and then tell FOSUserBundle to use it instead.

## Creating the Override Form

In `src/AppBundle`, create a new `Form` directory and a new class called `RegistrationFormType`.
Extend the normal `AbstractType`. Then, I'll use Code->Generate or Command+N to
override the `buildForm()` method. Inside, just say `$builder->add('firstName')`.

In a moment, we'll tell FOSUserBundle to use *this* form instead of its normal
registration form.  But... instead of completely *replacing* the default form, all
I *really* want to do is *add* one field to it. Is there a way to extend the existing
form?

## Extending the Core Form

Absolutely! And once again, the web debug toolbar can help us out. Mouse over the
form icon and click that. This tells us what form is used on this page: it's called
`RegistrationFormType` - the same as our class name.

To build on *top* of that form, you don't actually extend it. Instead, override
a method called `getParent()`. Inside, we'll return the class that we want to extend.
At the top, add `use`, autocomplete `RegistrationFormType` from the bundle and
put `as BaseRegistrationFormType`, so it doesn't conflict with our class.

Now, in `getParent()`, we can say `return BaseRegistrationFormType::class`.

And that is it! This form will now contain the existing fields *plus* `firstName`.

## Registering the Form with FOSUserBundle

To tell FOSUserBundle about out form, we need to do two things. First, we need to
register this as a service. In my `app/config/services.yml`, add `app.form.registration`
with `class` set to the `RegistrationFormType`. It also needs to be tagged with
`name: form.type`.

Finally, copy the class name and go into `app/config/config.yml`. This bundle has
a *lot* of configuration. And at the bottom of the documentation page, you can
find a reference called `FOSUserBundle Configuration Reference`. I'll open it in
a new tab.

This is pretty awesome: a full dump of all of the configuration options. Some of
these are explained in more detail in other places in the docs, but I love seeing
everything right in front of me. And we can see what we're looking for under
`registration.form.type`.

Go back to your editor and add those: `registration`, `form` and `type`. Paste
our class name next to that!

We're done! Go back to registration and refresh. We got it! And that *will* save
when we submit.

## Customizing the Form Order

Why is it at the bottom? Well, remember inside of `register_content.html.twig`,
we're using `form_widget(form)`... which just dumps out the fields in whatever
order they were added. Need more control? Cool: remove that and instead use
`form_row(form.email)`, `form_row(form.username)`, `form_row(form.firstName)` and
`form_row(form.plainPassword)`.

If you're not sure what the field names are called, again, use your web debug toolbar
for the form: it shows you all of the fields.

Refresh that page! Yes! First Name is *exactly* where we want it to be.
