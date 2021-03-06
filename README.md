#redux-form


`redux-form` works with [React Redux](https://github.com/gaearon/react-redux) to enable an html form in [React](https://github.com/facebook/react) to use [Redux](https://github.com/gaearon/redux) to store all of its state.

## Installation

```
npm install --save redux-form
```

## Benefits

Why would anyone want to do this, you ask? React a perfectly good way of keeping state in each component! The reasons are threefold.

#### Unidirectional Data Flow

For the same reason that React and Flux is superior to Angular's bidirectional data binding. Tracking down bugs is much simpler when the data all flows through one dispatcher.

#### Redux Dev Tools

When used in conjunction with [Redux Dev Tools](https://github.com/gaearon/redux-devtools), you can fast forward and rewind through your form data entry to better find bugs.

#### Stateless Components

By removing the state from your form components, you inherently make them easier to understand, test, and debug. The React philosophy is to always try to use `props` instead of `state` when possible.

## How it works

When you are adding your reducers to your redux store, add a new one with `createFormReducer(])`.

```javascript
import { createStore, combineReducers } from 'redux';
import { createFormReducer } from 'redux-form';
const reducers = {
  // ... your other reducers here ...
  createFormReducer('contacts', ['name', 'address', 'phone'])
}
const reducer = combineReducers(reducers);
const store = createStore(reducer);
```

`reduxForm()` creates a Higher Order Component that expects a `dispatch` prop and a slice of the Redux store where its data is stored as a `form` prop. These should be provided by [React Redux](https://github.com/gaearon/react-redux)'s `connect()` function.

Let's look at an example:

Then, on your form component, add the `@reduxForm('contacts')` decorator.

```javascript
import React, {Component, PropTypes} from 'react';
import {connect} from 'react-redux';
import reduxForm from 'redux-form';
import contactValidation from './contactValidation';

class ContactForm extends Component {
  // you don't need all to define all these props,
  // only the ones you intend to use
  static propTypes = {
    data: PropTypes.object.isRequired,
    dirty: PropTypes.bool.isRequired,
    errors: PropTypes.object.isRequired,
    handleBlur: PropTypes.func.isRequired,
    handleChange: PropTypes.func.isRequired,
    initializeForm: PropTypes.func.isRequired,
    invalid: PropTypes.bool.isRequired,
    pristine: PropTypes.bool.isRequired,
    resetForm: PropTypes.func.isRequired,
    touch: PropTypes.func.isRequired,
    touched: PropTypes.object.isRequired,
    touchAll: PropTypes.func.isRequired,
    untouch: PropTypes.func.isRequired,
    untouchAll: PropTypes.func.isRequired,
    valid: PropTypes.bool.isRequired
  }
  
  render() {
    const {
      data: {name, address, phone},
      errors: {name: nameError, address: addressError, phone: phoneError},
      touched: {name: nameTouched, address: addressTouched, phone: phoneTouched},
      handleChange
    } = this.props;
    return (
      <form>
        <label>Name</label>
        <input type="text" value={name} onChange={handleChange('name')}/>
        {nameError && nameTouched ? <div>{nameError}</div>}
        
        <label>Address</label>
        <input type="text" value={address} onChange={handleChange('address')}/>
        {addressError && addressTouched ? <div>{addressError}</div>}
        
        <label>Phone</label>
        <input type="text" value={phone} onChange={handleChange('phone')}/>
        {phoneError && phoneTouched ? <div>{phoneError}</div>}
      </form>
    );
  }
}

// ------- HERE'S THE IMPORTANT BIT -------
function mapStateToProps(state) {
  return { form: state.contacts };
}
export default 
  connect(mapStateToProps)
  (reduxForm('contacts', contactValidation)
  (ContactForm))
```

Notice that we're just using vanilla `<input>` elements there is no state in the `ContactForm` component. I have left handling `onSubmit` as an exercise for the reader. Hint: your data is in `this.props.data`.

### ES7 Decorator Sugar

Using ES7 decorators, the example above could be written as:

```javascript
@connect(state => ({ form: state.contacts }))
@reduxForm('contacts', contactValidation)
export default class ContactForm extends Component {
```

Much nicer, don't you think?

## Validation

You may optionally supply a validation function, which is in the form `({}) => {}` and takes in all your data and spits out error messages. For example:

```javascript
function contactValidation(data) {
  const errors = {};
  if(!data.name) {
    errors.name = 'Required';
  }
  if(data.address && data.address.length > 50) {
    errors.address = 'Must be fewer than 50 characters';
  }
  if(!data.phone) {
    errors.phone = 'Required';
  } else if(!/\d{3}-\d{3}-\d{4}/.test(data.phone)) {
    errors.phone = 'Phone must match the form "999-999-9999"'
  }
  return errors;
}
```
You get the idea.

## API

Each form has a `sliceName`. That's the key in the Redux store tree where the data will be mounted.

### createFormReducer(sliceName:string, fields:Array&lt;string&gt;, config:Object)

##### -`sliceName` : string

> the name of your form and the key to where your form's state will be mounted in the Redux store

##### - fields : Array&lt;string&gt;

> a list of all your fields in your form.

##### - config: Object [optional]

> some control over when to mark fields as "touched" in the form:

###### config.touchOnBlur : boolean [optional]

> marks fields to touched when the blur action is fired. defaults to `true`

###### config.touchOnChange : boolean [optional]

> marks fields to touched when the change action is fired. defaults to `false`

### @reduxForm(sliceName:string, validate:Function)

##### -`sliceName` : string

> the name of your form and the key to where your form's state will be mounted in the Redux store

##### - validation : Function [optional]

> your [validation function](#validation)

### props

The props passed into your decorated component will be:

##### -`data:Object`

> The form data, in the form `{ field1: <string>, field2: <string> }`

##### -`dirty:boolean`

> `true` if the form data has changed from its initialized values. Opposite of `pristine`.

##### -`errors:Object`

> All the errors, in the form `{ field1: <string>, field2: <string> }`

##### -`handleBlur(field:string) : Function`

> Returns a `handleBlur` function for the field passed.

##### -`handleChange(field:string) : Function`

> Returns a `handleChange` function for the field passed.

##### -`initializeForm(data:Object) : Function`

> Initializes the form data to the given values. All `dirty` and `pristine` state will be determined by comparing the current data with these initialized values.

##### -`invalid:boolean`

> `true` if the form has validation errors. Opposite of `valid`.

##### -`pristine:boolean`

> `true` if the form data is the same as its initialized values. Opposite of `dirty`.

##### -`resetForm() : Function`

> Resets all the values in the form to the initialized state, making it pristine again.

##### -`touch(...field:string) : Function`

> Marks the given fields as "touched" to show errors.

##### -`touched:Object`

> the touched flags for each field, in the form `{ field1: <boolean>, field2: <boolean> }`

##### -`touchAll() : Function`

> Marks all fields as "touched" to show errors. should be called on form submission.

##### -`untouch(...field:string) : Function`

> Clears the "touched" flag for the given fields

##### -`untouchAll() : Function`

> Clears the "touched" flag for the all fields

##### -`valid:boolean`

> `true` if the form passes validation (has no validation errors). Opposite of `invalid`.


## Running Example

Check out the [react-redux-universal-hot-example project](https://github.com/erikras/react-redux-universal-hot-example) to see `redux-form` in action.

This is an extremely young library, so the API may change. Comments and feedback welcome.
