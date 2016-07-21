
If you have a record on your database, and want to save it using a web form. You are using Polymer (but you could be using anything, really). You follow tutorials, and you find that everything works as expected: you make up a form (it could be an iron-form), you run it, and the record appears. You think this is a success -- until you put this form in the real world and discover that all those tutorials were lying. They were showing you that something with pedals and two wheels will move faster -- but failed to show you that in order to ride it you also need breaks, lights, a helmet; yes, they are all props. But yes, you always need them.

Ask yourself these questions:

* Do you need to load data before showing the form?
* What if loading of pre-populated data fails?
* Do you want to overwrite a record (PUT with an existing ID), or create a new one (POST, no ID) or create a record with an arbitrary ID (PUT with a new ID)? Do you realise that you will need POST or PUT calls?
* Do you want to display something if the form is a success? Or a failure? If so, how? A paper-toast?
* Do you realise you will need two different paper-toast widgets, depending on error or success?
* Do you want to highlight the problem fields if the form is NOT a success?
* Do you want to sibmit the form when the user presses ENTER on a paper-input widget?

If the answer is yes, and you are coding this each time from scratch, then maybe you should consider using a decorator element that does _all_ of this for you. Welcome to `hot-form`.

`<hot-form>` is an element meant to wrap a `<form is="iron-form">`
element, to set fields as invalid (with the correct error message) if the server
returns an error

    <hot-form>
      <form is="iron-form" id="form" method="post" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
        <paper-button type="submit" raised on-click="_submit">Click!</paper-button>
      </form>
    </hot-form>

The element works by listening to the `on-iron-form-submit` and `on-iron-form-error`
to see what the server has returned: in case of success, it displays a
customisable "Success!"; in case of error, it looks at the server response sets
the problematic field as `invalid`, as well as setting the error messages.

    <hot-form success-message="Record saved!">
      <form is="iron-form" id="form" method="post" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
        <paper-button type="submit" raised on-click="_submit">Click!</paper-button>
      </form>
    </hot-form>

The message is provided with a `<paper-toast>` widget that will be looked for
as follows:

 - If `local-toast` is passed, then it will be searched in `document`
 - Otherwise, it will be searched within the widget

In terms of searching:

 - If `toast-id` is passed, it will be searched by the corresponding `id` field
 - Otherwise, it will be the first `<paper-toast>` widget found.

For example:

    <hot-form local-toast>
    ...
    </hot-form>

    <hot-form local-toast toast-id="some-local-toast-id">
    ...
    </hot-form>

    <hot-form toast-id="some-global-toast-id">
    ...
    </hot-form>

The server response could be 422; for this widget to set the errors properly,
the return message must be in the following format:

    {"message":"Unprocessable Entity","errors":[{"field":"name","message":"Not good!"}]}

This will cause `hot-form` to display the error message `Unprocessable Entity`
and set the field named `name` as `invalid`, with the invalid message set as
`Not good!`.

All automatically as it should be.

NOTE: sane-submit will make sure that 1) An invisible button is added to the form
so that pressing enter on the input fields will submit 2) When the paper-button
element in the form marked as "submit" is pressed, the form is submitted.
This option is still undocumented

