---
layout: post
title:      "React LifeCycle Methods"
date:       2020-04-10 14:52:37 -0400
permalink:  react_lifecycle_methods
---


Hi! I just got done with my React Redux project for Flatiron, my final project.

The project is called iManage. It was inspired by the web application I use to make payments for my rent. Every month I can log in to my apartment's portal and submit a payment for my rent. It is very convenient.

iManage allows the manager, Luna, to create and assign new users to a unit in her apartment complex. The application will calculate the users' balances from the move-in date and all previous payments. Users can log in to see their previous payments and submit new payments.

The project consists of a React front end with a Rails API for data persistence. One thing that I that I understood better after working through a project was React components, specifically lifecycle methods. During Learn, it wasn't really clear to me when I would need to use lifecycle hooks, like `componentDidMount()`, `componentDidUpdate()`, and `componentWillUnmount()`.

### ComponentWillUnmount()

**componentWillUnmount() was very useful in cleaning up components that are reused throughout the application.**

In my project I have many components that utilize a form. If a user tries to submit a form with invalid user data, my FormErrors component will display any errors.



```
//in Login Component
<FormErrors errors={this.props.errors} clearErrors={this.props.clearErrors}/>
```

```
//FormErrors component
import React from 'react';

class FormErrors extends React.Component {
    componentWillUnmount() {
        this.props.clearErrors();
    }

    displayErrors = () => {
        return (
            this.props.errors.map((error, i) => <li className='error' key={i}>{error}</li>)
        )
    }

    render() {
            return (
            <ul id="error-list">
                {this.displayErrors()}
            </ul>
        )
    }
}

export default FormErrors;
```

When user submits invalid inputs, it will display the red error messages. 

![image](https://i.imgur.com/jak68K6.png)

I use FormErrors component in many different forms including NewPaymentForm, NewUserForm, NewResidentForm, Login, and SignupForm. The problem is when a user submits a bad form and decides to navigate to a different form, that other form will display the same errors that were created from the original form. So for example, error messages generated from the Login component will display when user switches to SignupForm.

![image2](https://i.imgur.com/fFpThR5.png)

Using the lifecycle method `componentWillUnmount()` can dispel these problems.

```
//in FormErrors component
componentWillUnmount() {
        this.props.clearErrors();
}
```

*`componentWillUnmount()` is the last method that is executed before the component is removed from the DOM*. This is the perfect place to clean up any generated errors from a component. Any form that unmounts should clean up the errors, which is stored in Redux state, so that any other components that use FormErrors will have a 'new' FormErrors component.

`this.props.clearErrors()` is mapped to a dispatch action that will clear the errors key in my redux state. So now anytime the FormErrors component unmounts, the errors in my redux state will be deleted, and when I navigate to a different form there will be no more errors.

### ComponentDidMount() & ComponentDidUpdate()
**`componentDidMount()` and `componentDidUpdate()` were very useful for directing the webflow of the application.** 

For example, when a user successfully creates his account, he should be redirected again to their account page. 

For my project I used Rails sessions to keep track of user authentication, and stored the `isLoggedIn` status of the application and logged in `user` in Redux state. For more information about authetication check out my other [blog post](https://atribecalledarty.github.io/how_i_authenticated_users_react_application_w_rails_api) about authenticating and authorizing users.

This is my form SignupForm.

```
import React from 'react';
import Form from 'react-bootstrap/Form';

class NewUserForm extends React.Component {
    state = {
        username: "",
        first_name: "",
        last_name: "",
        email: "",
        phone_number: "",
        password: ""
    }

    changeHandler = event => {
        this.setState({
            [event.target.name]: event.target.value
        })
    }

    submitHandler = event => {
        event.preventDefault();
        this.props.addUser(this.state, this.props.isManager);
    }

    componentDidUpdate(){
        if (this.props.isLoggedIn) {
            this.props.history.push(`/users/${this.props.user.id}`)
        }
    }

    render() {
        return(
             <Form onSubmit={this.submitHandler}>
                 <h5>Create New Account</h5>
                 // state controlled inputs
             </Form>
        )
    }
}

export default NewUserForm;
```

*`componentDidUpdate()` will fire any time an update, a change to state or props, occurs.*
This includes when my form inputs change and also when any props that may be connected to Redux state changes.

This component has access to the prop `isLoggedIn.` When a user successfully creates the account, user will be logged in, setting `isLoggedIn` to true, which will prompt the `componentDidUpdate()` method. In `componentDidUpdate()`, we specify if `isLoggedIn` is true, redirect the url to `/users/${this.props.user.id}`.

Now when a user successfully creates an account, `componentDidUpdate()` will be invoked, redirecting the website!

For more in depth about logging in and authentication, please check out the other [blog post](https://atribecalledarty.github.io/how_i_authenticated_users_react_application_w_rails_api). 

Hope you found this helpful! This is the [link](https://github.com/atribecalledarty/i-manage) to my project's GitHub.
