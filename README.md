
Imagine you are using Polymer, you have a **record on your database, and want to save it** using a web form. **You follow tutorials**, and you find that everything works as expected: you make up a form (it could be an iron-form), you run it, and the record appears. **You think this is a success** -- until you put this form in the real world and discover that **all those tutorials were lying. They were showing you that something with pedals and two wheels will move faster -- but failed to show you that in order to ride it you also need breaks, lights, a helmet;** yes, they are all props. But yes, **you always need them**.

Ask yourself these questions:

* Do you need to load data before showing the form?
* What if loading of pre-populated data fails?
* Do you want to overwrite a record (PUT with an existing ID), or create a new one (POST, no ID) or create a record with an arbitrary ID (PUT with a new ID)? Do you realise that you will need POST or PUT calls?
* Do you want to display something if the form is a success? Or a failure? If so, how? A paper-toast?
* Do you realise you will need two different paper-toast widgets, depending on error or success?
* Do you want to highlight the problem fields if the form is NOT a success?
* Do you want to sibmit the form when the user presses ENTER on a paper-input widget?

If the answer is yes, and you are coding this each time from scratch, then maybe you should consider using a decorator element that does _all_ of this for you. Welcome to `hot-form`.

Imagine you have a form like this:

    <hot-form>
      <form is="iron-form" id="form" method="post" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
        <paper-button type="submit" raised on-click="_submit">Click!</paper-button>
      </form>
    </hot-form>

Here is what it does, and how.

## Show error messages, or a custom message in case of success, using a `paper-toast`

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

 - If `local-toast` is passed, then it will be searched in its light DOM. This is the default behaviour, as it's best to have your `paper-toast` as global widgets.
 - Otherwise, it will be searched in the document (if you really know what you are doing).

In terms of searching:

 - If `toast-id` is passed, it will be searched by the corresponding `id` field
 - Otherwise, it will be the first `<paper-toast>` widget found.

You can specify two toasts if you want to: one is for successful messages (hint: green) and one for unsuccessful ones (hint: red). In terms of searching for the error toast:

- If error-toast-id is passed, then that ID is used
- Otherwise, it's the SECOND toast in order of appearance

For example:

    <hot-form local-toast>
    ...
    </hot-form>

    <hot-form toast-id="some-global-toast-id">
    ...
    </hot-form>

    <hot-form local-toast toast-id="some-local-toast-id">
    ...
    </hot-form>

## Invalidate fields based on server response

The server normally responds either with an OK messsage (200, 201, etc.) or with an error. In case of errors, the server normally will tell you what the problem was, and which fields were affected. Unless you enjoy repeating code, you want the "invalidation" to happen automaticallty.

If the server returns errors (for example 422), `hot-form` will expect a JSON object back from the server like so:

    {"message":"Unprocessable Entity","errors":[{"field":"name","message":"Not good!"}]}

This will cause `hot-form` to display the error message `Unprocessable Entity` using the error `paper-toast`, and set the field named `name` as `invalid`, with the invalid message set as `Not good!`.

If the server returned a non-error, the server will set the "normal" `paper-toast` displaying "Success!", or the string set as success-message:

    <hot-form success-message="All done, thanks!">
    ...
    </hot-form>


## Make the right request to the server (PUT or POST), based on record-id

(To be documented)

## Load of record dynamically, pre-setting form values

(To be documented)

## Submit form when you press enter on a `paper-input` field, or when clicking on the button marked as `type=submit`.

Yes, you have to do these things by hand normally. Every time. Just wrap your form with hot-form, and you won't have to worry about it.


    <hot-form no-enter-submit>
    ...
    </hot-form>


If you want to disable "submit by pressing enter", just add `no-enter-submit` to `hot-form`.

    <hot-form no-enter-submit>
    ...
    </hot-form>


If you don't want to submit the form with a button automatically, just avoid setting a button as `type=submit`.

