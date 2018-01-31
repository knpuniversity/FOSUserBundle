# Layout and Template Customization

Everything *works*, but if you go to `/register`... it looks *awful*. Well, of
*course* it looks awful! FOSUserBundle has *no* idea how the page should be styled.
But don't worry: we can get this looking *much* better, very quickly.

First, on the web debug toolbar, find the template icon and click it. This will
show you all the templates used to render this page... which is a *beautiful*
cheat sheet for knowing what templates you can override!

## Correcting the Base Layout

The one I'm interested in is `layout.html.twig`, which lives in FOSUserBundle.

In my editor, I'll press Shift+Shift to open that file. Ok, *every* Twig template
in FOSUserBundle extends *this* `layout.html.twig` file. For example, see the
"Logged in as" text? That's coming from here.

But, we want all of FOSUserBundle's templates to instead extend *our* `base.html.twig`
template. How can we do that?

## Overriding the Layout

By overriding `layout.html.twig`. Let's see how. First, to override *any* template
from a bundle, just go to `app/Resources`, then create a directory with the same
name as the bundle: `FOSUserBundle`. Inside, create one more directory: `views`.

***TIP
The location where templates should live to override bundle templates has changed in Symfony 4. But, the idea is still the same. For details, see: [https://symfony.com/blog/new-in-symfony-3-4-improved-the-overriding-of-templates](https://symfony.com/blog/new-in-symfony-3-4-improved-the-overriding-of-templates).
***

In this case, the `layout.html.twig` template lives right the root of the `views/`
directory in the bundle. So that's where we need to create our's. Inside, extend
the normal `base.html.twig`. 

[[[ code('a2bd06d299') ]]]

Here's the magic part. Hit Shift+Shift again and open `register.html.twig`: this
is the template for the registration page. Notice that it overrides a block called
`fos_user_content`. In `layout.html.twig`, this is printed in the middle.

So check this out: inside of our version of `layout.html.twig`, add `{% block body %}`:
that's the name of the block in our `base.html.twig`. Then, include `{% block fos_user_content %}`
and `{% endblock %}`.

[[[ code('a381314dff') ]]]

We're effectively *transferring* the content from the `fos_user_content` block to
the correct block: `body`.

These 5 lines of code are *huge*. Refresh the page! Ha! So much better! Not perfect,
but every page now lives in our layout. If you want, you can even add a little more
markup around the block.

[[[ code('1466a17f63') ]]]

## Overriding Individual Templates

Overriding the base layout is step one. But, each individual page still won't look
quite right. On this page, we *at least* need a "Register" `h1`, and I'd like to
make that button look better.

So in addition to overriding `layout.html.twig`, you really need to override every
template from FOSUserBundle that you use - like registration, reset password, login
and a few others.

Once again, click the templates link in the web debug toolbar. The template behind
this page is `register.html.twig`, which we already have open. But notice, it immediately
includes `register_content.html.twig`. This is a really common pattern in this bundle.

Let me show you: I'll click the `views` link to move my tree into FOSUserBundle.
In `Registration`, we have `register.html.twig` and `register_content.html.twig`.
In `Profile` there's the same for edit and show.

In most cases, you'll want to override the `_content.html.twig` template. Why? Well,
it doesn't really matter: by overriding the `_content.html.twig` template, you
don't need to worry about extending anything: you can just focus on the content.

Copy the contents of `register_content.html.twig`. Then, back in `app/Resources/views`,
create a `Registration` directory. I'm doing that because this template lives in
a `Registration` directory. Finally, create `register_content.html.twig` and paste
in the content. Let's add a couple of classes to the button and an h1 that says:
"Register Aquanauts!"

[[[ code('7e1481696d') ]]]

Ok, refresh! Love it! In your app, make sure to do this for all of the different
pages from the bundle that you're using. And remember, if you don't need a page -
like the edit profile page - save yourself some time by *not* importing its route
or overriding its template.
