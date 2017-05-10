# Customize everything with Events

We can override templates. We can override translations. We can override forms.
But there's *more* than that. For example, after we finish registration, we're
redirected to this registration confirmation page. You know what? I'd rather do
something else: I'd rather redirect to the homepage and skip this page entirely.
How can we do that?

Well, let's do a little bit of digging. If you hover over the route name in the
web debug toolbar, you can see that this is page rendered by `RegistrationController`.
Back in my editor I'll press Shift+Shift and look for `RegistrationController` in
the bundle. Specifically,` registerAction()` is responsible for both rendering the
registration page *and* processing the form submit.

And check this out: after the form is valid, it redirects to the confirmation page.

## Events to the Rescue!

So at first, it seems like we need to override the controller itself. But not so
fast! The controller - in fact *every* controller in FOSUserBundle is *littered*
with events: `REGISTRATION_INITIALIZED`, `REGISTRATION_SUCCESS`, `REGISTRATION_COMPLETED`
and `REGISTRATION_FAILURE`. Each of these represents a *hook* point where we can
add custom logic.

In this case, if you look closely, you can see that after it dispatches an event
called `REGISTRATION_SUCCESS`, below, it checks to see if the `$event` has a response
set on it. If it does not, it redirects to the confirmation page. But if it *does*,
it uses that response.

That's the key! If we can add a listener to `REGISTRATION_SUCCESS`, we can create
our own `RedirectResponse` and set that on the event so that this controller uses
it. Let's go!

## Creating the Event Subscriber

Inside of AppBundle, create a new directory called `EventListener`. And in there,
a new PHP class: how about `RedirectAfterRegistrationSubscriber`. Make this implement
`EventSubscriberInterface`: the interface that all event subscribers must have.
I'll use our favorite Code->Generate menu, or Command+N on a Mac, go to
"Implement Methods" and select `getSubscribedEvents`.

[[[ code('1d3893a9a7') ]]]

We want to attach a listener to `FOSUserEvents::REGISTRATION_SUCCESS`, which, by
the way, is just a constant that equals some string event name.

In `getSubscribedEvents()`, add `FOSUserEvents::REGISTRATION_SUCCESS` assigned to
`onRegistrationSuccess`. This means that when the `REGISTRATION_SUCCESS` event
is fired, the `onRegistrationSuccess` method should be called. Create that above:
`public function onRegistrationSuccess()`.

[[[ code('d868577943') ]]]

Oh, and notice that when this event is dispatched, the bundle passes a `FormEvent`
object. That will be the first argument to our listener method: `FormEvent $event`.
That's what we need to set the response onto.

## Investigating all the Events

Before we go any further, I'll hold command and click into the `FOSUserEvents` class...
cause it's awesome! It holds a list of *every* event dispatched by FOSUserBundle,
what its purpose is, and what event object you will receive. This is gold.

## Creating the RedirectResponse

Back in `onRegistrationSuccess`, we need to create a `RedirectResponse` and set
it on the event. But to redirect to the homepage, we'll need the router. At the
top of the class, create `public function __construct()` with a `RouterInterface $router`
argument. Next, I'll hit Option+Enter, select "Initialize Fields" and
choose `router`.

[[[ code('09df5ac847') ]]]

That was just a shortcut to create the `private $router` property and set it in
the constructor: nothing fancy.

Now, in `onRegistrationSuccess()` we can say `$url = $this->router->generate('homepage')`,
and `$response = new RedirectResponse($url)`. You may or may not be familiar with
`RedirectResponse`. In a controller, to redirect, you use `$this->redirectToRoute()`.
In reality, that's just a shortcut for these two lines!

Finally, add `$event->setResponse($response)`.

[[[ code('1f72f2a493') ]]]

Ok, this class is *perfect*! To tell Symfony about the event subscriber, head to
`app/config/services.yml`. At the bottom, add `app.redirect_after_registration_subscriber`,
set the class, and add `autowire: true`. By doing that, thanks to the `RouterInterface`
type-hint, Symfony will automatically know to pass us the router.

Finally, add a tag on the bottom: `name: kernel.event_subscriber`. And we are done!

[[[ code('51fce9e66a') ]]]

Try it out! Go back to `/register` and signup as `aquanaut5@gmail.com`. Fill out
the rest of the fields and submit!

Boom! Back to our homepage! You can customize just about *anything* with events.
So don't override the controller. Instead, hook into an event!
 