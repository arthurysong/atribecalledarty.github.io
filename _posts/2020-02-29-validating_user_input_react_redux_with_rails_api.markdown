---
layout: post
title:      "Validating User Input (React Redux with Rails API)"
date:       2020-02-29 21:05:22 -0500
permalink:  validating_user_input_react_redux_with_rails_api
---


Hey, in this blog post I'm going to go over how to validate form input against models in Rails.

So, I have a React application that loads users from Rails API and stores in redux state.
I also am using thunk so I can pass my redux reducer a function.

I want to create a new user in the Rails database using form inputs on the front end.
And I want to validate user input against validations I have in the models in the Rails API RATHER than validating the inputs on the front end.

This was important because I wanted some attributes like usernames to be unique for each user.

I wanted the front end to send a post request to the api to create a new user. If the created user was not valid, I wanted the API to return errors, which would then be displayed by the front end. If user was valid, then send the new saved user, which would be added to redux state.

This is how I decided to validate using the backend models.

SETUP:

I have a User model in Rails API with some validations.

```
class User < ApplicationRecord
    has_secure_password
    validates :username, presence: true
    validates :first_name, presence: true
    validates :last_name, presence: true
    validates :email, presence: true
    validates :email, uniqueness: true
    validates :username, uniqueness: true
    validates :phone_number, format: { with: /\d{3}-\d{3}-\d{4}/, message: "needs to be in ###-###-#### format" }
end
```

This is my component CreateUser with state controlled forms.
This component is connected to redux store and has access to state and dispatch.
When form is submitted it will send post request to Rails API with the state.

```
class CreateUser extends React.Component {
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
        this.props.addUser(this.state);
    }

    render() {
        return(
            <div>
                <FormErrors errors={this.props.errors}/>
                <form onSubmit={this.submitHandler}>
                    <h3>New User</h3>
                    <label htmlFor="username">Username: </label>
                    <input 
                        onChange={this.changeHandler} 
                        type="text" id="username" 
                        name="username" 
                        placeholder="harry_potter"
                    />
                   <input type="submit" value="Add User"/>
                </form>
            </div>
        )
    }
}

const mapDispatchToProps = dispatch => {
    return {
        addUser: state => dispatch(postNewUser(state))
    }
}

const mapStateToProps = state => {
    return {
        errors: state.errors
    }
}


export default connect(mapStateToProps, mapDispatchToProps)(CreateUser);
```

(I only am showing username input to keep code short.)

So upon submission I call this.props.addUser, which calls: state => dispatch(postNewUser(state)).

```
 submitHandler = event => {
        event.preventDefault();
        this.props.addUser(this.state);
    }
		
const mapDispatchToProps = dispatch => {
    return {
        addUser: state => dispatch(postNewUser(state))
    }
}
```

This is what the postNewUser action looks like.

```
export const postNewUser = (state) => {
    return dispatch => {
        dispatch({ type: 'LOADING_RESOURCE' }) //<=== ignore this

        const body = JSON.stringify(state);
        fetch('http://localhost:3002/users', {
            method: "POST",
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            body
        })
            .then(resp => resp.json())
            .then(user => {
                if (user.errors) {
                    dispatch({ type: 'ADD_ERRORS', user })
                } else {
                    dispatch({ type: 'ADD_NEW_USER', user })
                }
            })
    }
}
```

When dispatch(postNewUser(state)) gets called in CreateUser, the following happens. The state (form inputs of new user) gets sent to Rails API using fetch request. THEN, Rails User Controller does one of two things. If the inputs are valid and the new user created saved to the database, it will send back the JSON for that user. IF user is NOT valid, the API will send JSON with key 'errors'.

This is my User create action that handles post request to /users.

```
    def create
        user = User.new(user_params)
        if user.save
            render json: UserSerializer.new(user).to_serialized_json //<=== UserSerializer customizes json of the user object
        else 
            render json: { errors: user.errors.full_messages }
        end
    end
```

So now when response is received if there are errors, it will dispatch 'ADD_ERRORS' action, and if there is normal user sent back with no errors, it will dispatch 'ADD_NEW_USER'.

```
//in postNewUsers fetch action
if (user.errors) {
    dispatch({ type: 'ADD_ERRORS', user })
} else {
    dispatch({ type: 'ADD_NEW_USER', user })
}
```

This is my reducer that handles those actions:

```
export default function manageResources (state = { units: [], users: [], loading: false, errors: [] }, action) {

    switch(action.type) {
        case 'ADD_NEW_USER':
            return {
                ...state,
                units: [ ...state.units ],
                users: [ ...state.users, action.user ],
                loading: false, //<== don't worry about this
                errors: []
            }
        case 'ADD_ERRORS':
            return {
                ...state,
                units: [ ...state.units ],
                users: [ ...state.users ],
                loading: false, //<== don't worry about this
                errors: [ ...action.user.errors ]
            }
        default:
            return state;
    }
}
```

**
When Rails API cannot save a user to the database it will return errors.
This will prompt the ADD_ERRORS action to dispatch.
The dispatched action will save the errors to the state of redux store.
And finally I have a component that will display the errors.
**

Inside CreateUser component, it displays the FormErrors component which receives the errors from the state and displays them.

```
 render() {
        return(
            <div>
						
			*********************************************
                <FormErrors errors={this.props.errors}/>	
			*********************************************
								
                <form onSubmit={this.submitHandler}>
                    <h3>New User</h3>
                    <label htmlFor="username">Username: </label>
                    <input 
                        onChange={this.changeHandler} 
                        type="text" id="username" 
                        name="username" 
                        placeholder="harry_potter"
                    />
                   <input type="submit" value="Add User"/>
                </form>
            </div>
        )
}

const mapStateToProps = state => {
    return {
        errors: state.errors
    }
}
```

And finally this is the FormErrors component that will display errors.

```
const FormErrors = ({ errors }) => {
    const displayErrors = errors.map((error, i) => <li key={i}>{error}</li>)

    return (
        <div>
            {displayErrors}
        </div>
    )
}
```

So that is basically how I used Rails model validations to validate the inputs and display any errors. And if there are no errors it will dispatch the ADD_NEW_USER action, and add the proper user to the redux state.

In conclusion, I needed to
1. add an errors array to my redux state to store any errors from fetch requests
2. in my user controller, send back either errors if user cannot save or a valid saved user
3. in my action generator for dispatch, have if statement for when API sends errors
4. finally display errors

