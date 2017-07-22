# Sanitizing User Inputs

Right now our code has no logic for handling the situation when the user enters an incorrect password confirmation. It just fails silently, redirecting the user to the links page. In the controller, the `user.id` will be `nil` because datamapper won't save the record if the validations fail.

```ruby
  post '/users' do
    user = User.create(email: params[:email],
                       password: params[:password],
                       password_confirmation: params[:password_confirmation])
    # the user.id will be nil if the user wasn't saved
    # because of password mismatch
    session[:user_id] = user.id
    redirect to('/links')
  end
```

Let's extend the test to ensure we are not redirected if the passwords don't match.

```ruby
scenario 'with a password that does not match' do
  expect { sign_up(password_confirmation: 'wrong') }.not_to change(User, :count)
  expect(current_path).to eq('/users') # current_path is a helper provided by Capybara
  expect(page).to have_content 'Password and confirmation password do not match'
end
```

This test expects the client to stay at `/users`, instead of being directed to the links page as they would on successful signup.

Instead of redirecting the user, we'll send a response back to the browser showing the same form again.

```ruby
post '/users' do
  # we just initialize the object
  # without saving it. It may be invalid
  user = User.new(email: params[:email],
                  password: params[:password],
                  password_confirmation: params[:password_confirmation])
  if user.save # #save returns true/false depending on whether the model is successfully saved to the database.
    session[:user_id] = user.id
    redirect to('/links')
    # if it's not valid,
    # we'll render the sign up form again
  else
    erb :'users/new'
  end
end
```

This is a fairly common pattern for handling errors in the model. Instead of creating the model straight away, you initialize it then test if it saves successfully.

Is the test passing?

We need to tell the user why their sign-up attempt failed!  Fortunately, inserting additional useful messages into our pages is a common requirement and there is a common pattern we can use to do this.  Let's display a **flash message** at the top of the page which notifies the user of the error(s).

* :white_check_mark: Add and require the gem [sinatra-flash](https://github.com/SFEley/sinatra-flash). *Important!* We are using the 'Sinatra::Base' style so you must follow the instructions on github (see link) under 'Setting Up' to configure sinatra-flash. Now add the flash line to `app.rb`:

```ruby
  if @user.save
    session[:user_id] = @user.id
    redirect to('/links')
  else
    flash.now[:notice] = "Password and confirmation password do not match"
    erb :'users/new'
  end
```

Add the following code to `layout.erb` - ideally before the `yield` - to present the flash message on the page.

```html
<% if flash[:notice] %>
  <div id='notice'><%= flash[:notice] %></div>
<% end %>
```

It will be displayed on top of the page that was re-rendered (note the `/users` path).

Do the tests now pass?

Rackup your app and try signing up for yourself with a mismatched password confirmation.  Is there something about the UX that's really annoying?

Let's suppose we have a longer sign-up form. A user fills out 20 fields but makes a mistake in `password_confirmation`. There is a danger here that we lose all the *valid* information entered by the user, as it never makes it to the database (try to confirm this in IRB). Fortunately, the data is available temporarily in the unsaved `user` variable.

Can this data (in our case the email the user entered) make its way from the user object to the re-rendered form? Let's make the `user` an instance variable and update the view.

```ruby
post '/users' do
  @user = User.new(email: params[:email],
                  password: params[:password],
                  password_confirmation: params[:password_confirmation])
  if @user.save
    session[:user_id] = @user.id
    redirect to('/')
  else
    flash.now[:notice] = "Password and confirmation password do not match"
    erb :'users/new'
  end
end
```

```html
Email: <input name='email' type='text' value='<%= @user.email %>'>
```

Now the email will be part of the form when it's rendered again.

Because the view now expects `@user` instance variable, we must make sure that it's available in the `/users/new` route as well.

```ruby
get '/users/new' do
  @user = User.new
  erb :'users/new'
end
```

A new instance of the user will simply return nil for `@user.email`.



![alt text](https://dchtm6r471mui.cloudfront.net/hackpad.com_jubMxdBrjni_p.52567_1380105990218_Screen%20Shot%202013-09-25%20at%2011.46.01.png "bookmark manager")


Finally, our tests pass.

[ [Next Stage](bookmark_manager_stage_5.md) ]

[ [Return to outline](bookmark_manager.md) ]
