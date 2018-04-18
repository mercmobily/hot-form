
[![Published on webcomponents.org](https://img.shields.io/badge/webcomponents.org-published-blue.svg)](https://www.webcomponents.org/element/mercmobily/hot-form)

(Note: if you use this element, you should also consider using [hot-network (GitHub)](https://github.com/mercmobily/hot-network) [hot-network (WebComponents)](https://www.webcomponents.org/element/mercmobily/hot-network)

Imagine you are using Polymer, you have a **record on your database, and want to save it** using a web form. **You follow tutorials**, and you find that everything works as expected: you make up a form (it could be an iron-form), you run it, and the record appears. **You think this is a success** -- until you put this form in the real world and discover that **all those tutorials were lying. They were showing you that something with pedals and two wheels will move faster -- but failed to show you that in order to ride it you also need breaks, lights, a helmet;** yes, they are all props. But yes, **you always need them**.

Ask yourself these questions:

* Do you need to load data from the server before showing the form?
* Do you want to overwrite a record (PUT with an existing ID), or create a new one (POST, no ID) or create a record with an arbitrary ID (PUT with a new ID)? Do you realise that you will need POST or PUT calls?
* Do you want to render ALL input fields _automatically_ disabled while the form is being submitted?
* Do you want to highlight the problem fields if the form is NOT a success?
* Do you want to submit the form when the user presses ENTER on a paper-input widget?
* Do you want to refresh the data with the saved info coming back from the server after saving?
* Do you want to display messages to the user?
* Do you want to have fields in a form that won't actually get submitted?

If the answer is yes, and you are coding this each time from scratch, then maybe you should consider using a decorator element that does _all_ of this for you. **Welcome to `hot-form`**.

Imagine you have a form like this:

    <hot-form>
      <form is="iron-form" method="post" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
        <paper-button type="submit" raised on-click="_submit">Click!</paper-button>
      </form>
    </hot-form>

Here is what it does, and how.

## Invalidate fields based on server response

The server normally responds either with an OK status (200, 201, etc.) or with an error status. In case of errors, the server normally will tell you what the problem was, and which fields were affected. Unless you enjoy repeating code, you want the "invalidation" to happen automaticallty.

If the server returns errors (for example 422), `hot-form` will expect a JSON object back from the server like so:

    {"message":"Unprocessable Entity","errors":[{"field":"name","message":"Not good!"}]}

This will cause `hot-form` to set the field named `name` as `invalid`, with the invalid message set as `Not good!`.

**NOTE**: in some cases, the actual submitting field is a hidden one, whereas the UI input field is only used for input. In such a case, you can assign the `vname` property to the UI field, which will get the error message in case the server responds with one.

## Emit events so that other widgets can display messages

The element will fire the following messages:

* `user-message-info`. Fired when a form is submitted. `e.detail` will have `{ message: msg }`. The default message is `Saving...`, and it can be changed by setting `submit-message`.
* `user-message-success`. Fired when a successful response is received. `e.detail` will have `{ message: msg }`. The default message is `Success!`, and it can be changed by setting `success-message`.
* `user-message-error`. Fired when an error response is received. `e.detail` will have `{ message: msg }`. The default message will depend on the `error` field returned by the server. If the `error` field isn't there, the defaut  message is `Error!` and it can be changed by setting `default-error-message`.

These events will bubble up. So, you can put elements that listen to them, and display these messages to the user.

## Make the right request to the server (PUT or POST), based on `record-id`

If `record-id` is set, the element will make sure that the `request` object used by the `iron-ajax` form is manipulatd before sending it, so that the `action` attribute is changed to `/original/url/:recordId` (with `:recordId` appended), and the method used by `iron-ajax` is `PUT`. This means that the element can be used to make spefific PUT calls on specific records, based on `recordId`. For example:

    <hot-form id="hot-form" record-id="57902ef29b880cd678a3d7a9">
      <form is="iron-form" id="iron-form" method="POST" action="/stores/polymer">
        <paper-input required id="name" name="name" label="Your name"></paper-input>
        <paper-input required id="surname" name="surname" label="Your surname"></paper-input>
        <paper-input required type="number" id="age" name="age" label="Your age"></paper-input>
        <paper-toggle-button id="active" name="active" label="Active?">Active?</paper-toggle-button>
        <paper-button raised type="submit">Click!</paper-button>
      </form>
    </hot-form>

When the form is submitted, `PUT /stores/polymer/57902ef29b880cd678a3d7a9` will actually be called.

### Modifiers for GET and PUT requests

In some cases, you might want the PUT request to use slightly different URLs:

* `no-record-id-append`: when running the PUT request, it will not append the record ID. This is useful when the server alreayd knows the record ID based on the session (e.g. while fetching the user's record)

* `put-suffix`: in some cases, the server might expect an extra path when putting data. It's rare, but it does happen. Whatever is in `put-suffix` will be added to the PUT URL. In the example above, setting `put-suffix` to '/extra` would turn the put into `PUT /stores/polymer/57902ef29b880cd678a3d7a9/extra`.

## Load of record with an AJAX call, pre-setting corresponding form values

If `record-id` is set, the element will make a GET AJAX call to the `action` URL (in the example above, `GET /stores/polymer/57902ef29b880cd678a3d7a9` fetching the current record's value. It will also pre-set the form's value to
match those returned by the server.

Record autoload will happen automatically when `record-id` is set; it can be turned off with the skip-autoload attribute (in case values are already available from a cache, but you still want `hot-form` to _save_ them).

**Note**: the response must be a JSON record, where each field's key corresponds to the element's `name` attribute. In the example above, a valid JSON in return would be:

    {
      "name":"Tony",
      "surname":"Mobily",
      "age":40,
      "active":false,
      "id":"57902ef29b880cd678a3d7a9",
    }

This means that you can create your form, _and_ know that the existing record's value are already set.

Just like `put-suffix`, you can define a `get-suffix` parameter which will be appended to GET (preload) requests.

## Reset form when response arrives, or set values based on returned values

When getting the response after issuing a POST request (no `record-id` appended), the form is automatically reset. This can be changed with the `action-after-post-response` attribute, that can be set to `set`, `reset` (the default) or `none`.

When getting the response after issuing a PUT request (with `record-id` appended to the request), the form is automatically set to the values that came back from the server. This means that if the server did any manipulation to the data, the form will display the correct, current information. This can be changed with the `action-after-put-response` attribute, that can be set to `set` (the default), `reset` or `none`.

## Set an object's keys when the response arrives

Sometimes, you want to be able to see what was returned by the form's AJAX call. This is useful when a field's visibility  depends on information on the database.
For example, you might have a button tha says "Send email reminder", but only if the email is set.
For this purpose, you can bind a variabl to `info`, which allows 2-way binding.
For example:

    <iron-ajax url="/stores/contacts/{{routeData.contactId}}" last-response="{{info}}" auto></iron-ajax>

    <iron-ajax method="post" id="passwordResetAjax" url="/routes/passwordReset/{{routeData.contactId}}"></iron-ajax>
    <paper-button disabled="{{_emptyString(info.email)}}" raised on-tap="_sendPasswordResetEmail">Send reset password email</paper-button>

    <hot-form submit-message="Edit" record-id="{{info.id}}" info="{{info}}" skip-autoload action-after-put-response="none">
      <hot-network>
        <iron-form id="ironForm">
          <form enctype="application/json" method="post" action="/stores/companies">
            <paper-input value="[[info.email]]" required id="email" name="email" label="Email"></paper-input>
            <paper-button type="submit" raised>Save!</paper-button>
          </form>
        </iron-form>
      </hot-network>
    </hot-form>

As you can see, `paper-button` is disabled if `info.email` is not there. However, since the element's `info` property is bound to `hot-form`'s `info` property, when `hot-form` assigns values to `info`, `{{_emptyString(info.email)}}` will actually work (since the element's `info.email` will be set to the email value returned by the AJAX call).

## Sane submission when pressing "enter" or clicking the submit button

This element allows you to automatically submit form when you press enter on a `paper-input` field, or when clicking on the button marked as `type=submit`.

Yes, you have to do these things by hand normally (!). Every time. Just wrap your form with hot-form, and you won't have to worry about it.

    <hot-form>
    ...
    </hot-form>

If you want to disable "submit by pressing enter", just add `no-enter-submit` to `hot-form`.

    <hot-form no-enter-submit>
    ...
    </hot-form>

If you don't want to submit the form with a button automatically, just avoid setting a paper-button as `type=submit`.

## Do not submit specific fields

Sometimes, you need to have input fields in the form, and yet not submit those values. For example, you might have a "country" field that is actually an auto-complete field, which in turns will set the country's ID as an input field.
Every input field must have a name -- for example, if they don't they won't reset.
For this use case, just prefix "local" fields (not to be submitted) with `_`. THe following example is a typical use case: the country ID will depend on what was picked by hot-autocomplete; prefixing the paper-input's name with `_` will prevent the field `country` to be submitted.

    Country code: {{pickedCountry.id}}
    <hot-autocomplete must-match picked="{{pickedCountry}}" suggestions-path="name" url="/stores/countries?nameStartsWith={{countryName}}" method="get">
      <paper-input name="_country" vname="country" value="{{countryName}}" required id="countryName" label="Country"></paper-input>
    </hot-autocomplete>
    <input type="hidden" name="countryId" value="{{pickedCountry.id}}">


## Emit event before submission

Before submitting the form, hot-form will emit a `iron-form-presubmit` event. Note that this is fires _way before_ `iron-form`'s `iron-form-presubmit` event, which is fired when the form is already serialised and ready to be sent over the wire. `iron-form-presubmit` is fired before running `submit()` on the `iron-form` itself. This is handy for example in case you want to re-enable some fields (so that they are serialised) while they were read-only (or disabled) on the form, so that their values are included in the form's serialisation.

