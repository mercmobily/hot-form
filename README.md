
`<hot-form-validator>` is an element meant to wrap a `<form is="iron-form">`
element, to set fields as invalid (with the correct error message) if the server
returns an error

    <hot-form-validator>
      <form is="iron-form" id="form" method="post" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
        <paper-button type="submit" raised on-click="_submit">Click!</paper-button>
      </form>
    </hot-form-validator>

The element works by listening to the `on-iron-form-submit` and `on-iron-form-error`
to see what the server has returned: in case of success, it displays a
customisable "Success!"; in case of error, it looks at the server response sets
the problematic field as `invalid`, as well as setting the error messages.

    <hot-form-validator success-message="Record saved!">
      <form is="iron-form" id="form" method="post" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
        <paper-button type="submit" raised on-click="_submit">Click!</paper-button>
      </form>
    </hot-form-validator>

The message is provided with a `<paper-toast>` widget that will be looked for
as follows:

 - If `global-toast` is passed, then it will be searched in `document`
 - Otherwise, it will be searched within the widget

In terms of searching:

 - If `toast-id` is passed, it will be searched by the corresponding `id` field
 - Otherwise, it will be the first `<paper-toast>` widget found.

For example:

    <hot-form-validator global-toast>
    ...
    </hot-form-validator>

    <hot-form-validator global-toast toast-id="some-global-toast">
    ...
    </hot-form-validator>

    <hot-form-validator toast-id="some-toast">
    ...
    </hot-form-validator>

The server response could be 422; for this widget to set the errors properly,
the return message must be in the following format:

    {"message":"Unprocessable Entity","errors":[{"field":"name","message":"Not good!"}]}

This will cause `hot-form-validator` to display the error message `Unprocessable Entity`
and set the field named `name` as `invalid`, with the invalid message set as
`Not good!`.

All automatically as it should be.

NOTE: sane-submit will make sure that 1) An invisible button is added to the form
so that pressing enter on the input fields will submit 2) When the paper-button
element in the form marked as "submit" is pressed, the form is submitted.
This option is still undocumented

