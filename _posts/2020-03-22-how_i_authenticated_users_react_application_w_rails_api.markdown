---
layout: post
title:      "How I Authenticated Users (React Application w Rails API)"
date:       2020-03-22 19:54:22 +0000
permalink:  how_i_authenticated_users_react_application_w_rails_api
---


How do you authenticate users in a React application using user information that is stored in a Rails API?
This was the method I found least complicated. I didn't want to use plugins that I didn't understand.
You just need basic understanding of React, React routing, Rails, Rails sessions, and Redux (You don't have to use redux).

*You also need to know how to use bcrypt to store user passwords and authenticate users in Rails
*

I will be going over how to login a user to your React application, and how to direct the web flow of a logged in user.

Alejandro's guide here was what I went off of the most => [Link](https://medium.com/how-i-get-it/react-with-rails-user-authentication-8977e98762f2)

Here is the GitHub repo to how I set this up in my own project => [GitHub](https://github.com/atribecalledarty/i-manage)

**Before we get started, 1. make sure you configure CORS so React application can communicate with Rails. 
And also 2. make sure your Rails API is not configured to API only.**

   (1) Configure CORS
a. Add gem 'rack-cors' to your gem file and run `bundle install`

```
#Gemfile

gem 'rack-cors'
```

b. Allow requests from your React application
In `config/initializers` folder create folder called `cors.rb`, and add following.
**Make sure you have `credentials:true` because we will be passing in credentials in the HTTP request so that Rails can set and read the cookies on the browser.**

```
Rails.application.config.middleware.insert_before 0, Rack::Cors do 
  allow do
    origins 'http://localhost:3000'
  
    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head],
      credentials: true
  end
end
```

c. Configure handling of cookies
In `config/initializers` create new file called `session_store.rb`, and following
```
if Rails.env === 'production' 
  Rails.application.config.session_store :cookie_store, key: '_your-app-name', domain: 'your-frontend-domain'
else
  Rails.application.config.session_store :cookie_store, key: '_your-app-name' 
end
```

To read more about configuring CORS, check out Alejandro's [guide](https://medium.com/how-i-get-it/react-with-rails-user-authentication-8977e98762f2) 

   (2) Make sure in `config/application.rb` the following line is commented out (this only applies if you generated your Rails application as an API. 

```
module VillaBugambilias
  class Application < Rails::Application
    # config.api_only = true <=== comment out this line 
  end
end
```

This needs to be commented so Rails can make use of Rails Sessions.

# Logging In

Now, that you have that set up, THIS IS BASICALLY THE PROCESS OF LOGGING IN:
1. User submits login form in React Client
2. React sends user params to RAILs API
3. If user and password correct, Rails will create new session
4. Rails also sends JSON with **isLoggedIn: true** and **user** that logged in
5. React stores this info in state (I did this in redux, it doesn't matter whether you use redux or not so don't worry)

Now these are the different parts of my project.
This is my Login page with state controlled forms.

```
class Login extends Component {
    state = {
        email: '',
        password: ''
    }

    changeHandler = event => {
        this.setState({
            [event.target.name]: event.target.value
        })
    }

    submitHandler = event => {
        event.preventDefault();
        this.props.loginUser(this.state);
    }

    render() {
        return(
            <div>
                <form onSubmit={this.submitHandler}>
                    <label htmlFor="email">Email </label>
                    <input type="text"
                        name="email" 
                        id="email" 
                        onChange={this.changeHandler} 
                        value={this.state.email}/>
                    <br />
                    
                    <label htmlFor="password">Password </label>
                    <input type="password" 
                        name="password" 
                        id="password" 
                        onChange={this.changeHandler} 
                        value={this.state.password}/>
                    <br/>

                    <input type="submit" value="Sign In"/>
                </form>
            </div>
        )
    }
}
```

This is my Redux store state

```
state = {  
        isLoggedIn: false,
        user: {},
        errors: []
    }
```

*If you are not using Redux state, store the state of isLoggedIn and user in suitable React component.
*

This is my sessions controller, which is responsible for creating new session upon request from React, and also for sending back the loggedIn status and loggedIn User to React.

```
class SessionsController < ApplicationController

  def create // <========================= post /login routes to this action
    @user = User.find_by(email: session_params[:email])
  
    if @user && @user.authenticate(session_params[:password])
      login!
      render json: {
        logged_in: true,
        user: @user
      }
    else
      render json: { errors: ["Invalid combination of email and password. Try again"] }
    end
  end

  def is_logged_in? // <=================== get /logged_in routes to this action
    if logged_in? && current_user
      render json: {
        logged_in: true,
        user: current_user
      }
    else
      render json: {
        logged_in: false
      }
    end
  end

  def destroy // <============================= delete /logout routes to this action
      logout!
      render json: {
        status: 200,
        logged_out: true
      }
  end

  private
    
    def session_params
      params.require(:user).permit(:username, :email, :password)
    end
    
    def login!
      session[:user_id] = @user.id
    end

    def logged_in?
      !!session[:user_id]
    end

    def current_user
      @current_user ||= User.find(session[:user_id]) if session[:user_id]
    end

    def authorized_user?
      @user == current_user
    end

    def logout!
      session.clear
    end


  end
```

So when user submits form with their credentials, I call function *this.props.loginUser(this.state)*, which will send a post request to my Rails API, with the user credentials. Here are those codes..

```
//Login component
submitHandler = event => {
        event.preventDefault();
        this.props.loginUser(this.state);
				//this.props.loginUser is mapped using redux and thunk to: user => dispatch(loginUser(user))
				//if you are not using redux and thunk don't worry
    }
```

```
//exported redux thunk action
export const loginUser = user => {
    return dispatch => {
        axios.post('http://localhost:3002/login', { user }, {withCredentials: true})
            .then(resp => {
                if (resp.data.logged_in) {
                    dispatch({ type: 'LOGIN', user: resp.data.user })
                } else {
                    dispatch({ type: 'ADD_ERRORS', errors: resp.data.errors })
                }
            })
    }
}
```

This is loginUser function. I'm using thunk middleware so I can pass in a function to my dispatch action. If you're not using redux don't worry about this. All you need to know is this part. *Make sure when user submits form, this request is sent to Rails API.*

```
axios.post('http://localhost:3002/login', { user }, {withCredentials: true})
            .then(resp => {
                if (resp.data.logged_in) {
                    dispatch({ type: 'LOGIN', user: resp.data.user })
                } else {
                    dispatch({ type: 'ADD_ERRORS', errors: resp.data.errors })
                }
            })
```

I am using axios package to send post request to my API. For axios, run `npm install axios` in your application and `import axios from 'axios'` in whichever react component you need. **It's important you use axios, so that Rails can set and read the cookies on your browser. Make sure to have { withCredentials: true } in args.** 

POST `http://localhost:3002/login` routes to this controller action.

```
//sessions_controller.rb
def create
    @user = User.find_by(email: session_params[:email])
  
    if @user && @user.authenticate(session_params[:password])
      login!
      render json: {
        logged_in: true,
        user: @user
      }
    else
      render json: { errors: ["Invalid combination of email and password. Try again"] }
    end
  end
	
//helper method
    def login!
      session[:user_id] = @user.id
    end
```

If user credentials are valid it will set the Rails `session[:user_id]` to the valid user's id, and return JSON of `logged_in` status of Rails application and the valid `user` back to client. And if not valid, Rails will return errors.

```
//in exported redux thunk action
axios.post('http://localhost:3002/login', { user }, {withCredentials: true})
            .then(resp => {
                if (resp.data.logged_in) {
                    dispatch({ type: 'LOGIN', user: resp.data.user })
                } else {
                    dispatch({ type: 'ADD_ERRORS', errors: resp.data.errors })
                }
            })
```

Then all we are doing is dispatching either the 'LOGIN' action or 'ADD_ERRORS' action depending on what API sent back.
'LOGIN' action will set `isLoggedIn: true` and `user: resp.data.user` in our redux state.
'ADD_ERRORS' action will add returned errors to `errors` array in state. 

This is my redux store reducer.

```
export default function manageResources (
    state = { 
        errors: [], 
        isLoggedIn: false,
        user: {} 
    }, action) {

    switch(action.type) {
        case 'ADD_ERRORS':
            return {
                ...state,
                isLoggedIn: state.isLoggedIn,
                user: state.user,
                errors: [ ...action.errors ]
            }
        case 'LOGIN':
            })
            return {
                ...state,
                isLoggedIn: true,
                user: action.user,
                errors: [ ]
            }
        case 'LOGOUT':
            return {
                ...state,
                isLoggedIn: false,
                user: {},
                errors: [ ]
            }
        default:
            return state;
    }
}
```

If you are not using redux, use `this.setState()` to set state of whichever React component.

i.e.
```
this.setState({
   isLoggedIn: true,
   user: resp.data.user
})
```

Essentially, using React form data and a Rails database, we have logged in the user in the Rails "API", and also logged in the user in the React application.

# Directing Web Flow

Now we will be going over *keeping track of the logged in status of the Rails API* and also *using the `isLoggedIn` status to authorize users*.

As soon as user enters website, we want to check in if user has logged in to application using with their browser.
We will use **componentDidMount()** lifecycle method, to check logged in status upon first loading the application.

```
class App extends React.Component {  
  componentDidMount(){
    this.props.setLoginStatus();
  }
	
	render(){
	...
	}
}

//this.props.setLoginStatus is mapped to () => dispatch(setLoginStatus()) using react-redux.
```

```
//this is the setLoginStatus imported action
export const setLoginStatus = () => {
    return dispatch => {
        axios.get('http://localhost:3002/logged_in',{
            withCredentials: true
        })
            .then(resp => {
                if (resp.data.logged_in) {
                    dispatch({ type: 'LOGIN', user: resp.data.user })
                } else {
                    dispatch({ type: 'LOGOUT' })
                }
            })
    }
}
```

Again this is using redux and thunk.
Below is the part that you need to know.
**Make sure you are using axios, with { withCredentials: true } so Rails has access to browser cookies.**

```
axios.get('http://localhost:3002/logged_in',{
            withCredentials: true
        })
            .then(resp => {
                if (resp.data.logged_in) {
                    dispatch({ type: 'LOGIN', user: resp.data.user })
										// if you're not using redux, use this.setState({
										//   isLoggedIn: true,
										//	 user: resp.data.user
										//	 })
                } else {
                    dispatch({ type: 'LOGOUT' })
										// this.setState({
										//    isLoggedIn: false,
										//    user: {}
										//    })
                }
            })
```

A get request to `http://localhost:3002/logged_in` routes to this controller action in `sessions_controller.rb`.

```
def is_logged_in?
    if logged_in? && current_user
      render json: {
        logged_in: true,
        user: current_user
      }
    else
      render json: {
        logged_in: false
      }
    end
  end
```

Helper methods `logged_in?` and `current_user`:

```
def logged_in?
      !!session[:user_id]
end

def current_user
      @current_user ||= User.find(session[:user_id]) if session[:user_id]
end
```

I think this is pretty self explanatory, if `logged_in?` Rails API will return `logged_in: true` and the `current_user`, else it will return `logged_in: false`. And React will either dispatch action `'LOGIN'` or `'LOGOUT'`, which will set the `isLoggedIn` and `user` state of the React application.

So if user was logged into the API, our state would update to:

```
state = {  
        isLoggedIn: true,
        user: {
           id: 1,
           username: "luna_lovegood",
           email: "itslunaaa@owl.com",
           ...
        },
        errors: []
    }
```

Now anytime application is loaded we will have set the correct logged in status for React.

Next, any components that need to be redirected if user has or has not logged in should use **componentDidUpdate()** to redirect the url.
Here are some routes I have set up, in my application.
My redux states, `isLoggedIn` and `user` are mapped to `this.props.isLoggedIn` and `this.props.user`.

```
//App Component
class App extends React.Component {  
  componentDidMount(){
    this.props.setLoginStatus();
  }

  render() {
    return (
      <Router >
        <h1>Welcome to Luna's Tavern</h1>
        
        <Route exact path="/" render={routerProps => 
          <Home 
            {...routerProps}
            user={this.props.user}
            isLoggedIn={this.props.isLoggedIn} />}/>
        <Route path={`/auth_user/:userId`} render={() => 
          <UserShow  
            user={this.props.user} />
      </Router>
	)}
}
```

When user goes to root, `http://localhost:3000/`, `Home` component will be rendered. Home component is passed `this.props.user`, `this.props.isLoggedIn`, and also `routerProps` so it has access to history.

```
//Home component
class Home extends React.Component {
    componentDidUpdate() { //<==================Home component should be redirected if logged in*******
        if (this.props.isLoggedIn) {
            this.props.history.push(`/auth_user/${this.props.user.id}`)
        }
    }

    render() {
        return (
            <p>Please Login/Register to continue!</p>
        )
    }
}
```

```
//UserShow presentational component
const UserShow = ({ user }) => {
    return (
        <div>
            <h3>{user.first_name} {user.last_name}</h3>
            <p>
                {user.email}<br/>
                {user.phone_number}
            </p>
        </div>
    )
}
```

**You want to use componentDidUpdate lifecycle hook, to redirect the webflow.** When the state of the application updates (user is logged in to the react application) the Home component will rerender to update the isLoggedIn prop. If `isLoggedIn` is equal to true, we will redirect to the show page of authorized user. 

```
//Home Component
componentDidUpdate() { 
        if (this.props.isLoggedIn) {
            this.props.history.push(`/auth_user/${this.props.user.id}`)
        }
    }
```

Basically, any components, where we might need to redirect the webpage, we want to pass in the `isLoggedIn` state as well as the **router props**, which contain the history. **I use** `this.props.history.push(`/auth_user/${this.props.user.id}`)` **to redirect the url to different route.**

Now we can set the logged in status upon application start as well as authorize users for access to different web pages!

Thanks for reading, hope you found this helpful. Any feedback is appreciated.




