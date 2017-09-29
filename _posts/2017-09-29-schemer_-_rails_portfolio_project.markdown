---
layout: post
title:  "Schemer - Rails Portfolio Project"
date:   2017-09-29 04:46:40 +0000
---


Schemer is a Ruby on Rails web app that lets you browse, create, and favorite color schemes.

I'll walk through some of the features starting from the signup page. For signup and authentication I just used Bcrypt and rails' has_secure_password macro on my User model as I didn't need anything complex like Devise. Here's my user controller logic to create a new user:

```
def create
    user = User.find_by_email(params[:email])
    if user && user.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to root_path
    else
      flash[:loginerror] = "Invalid Email or Password"
      render :new
    end
  end
```

And to handle logging in and logging out I had a sessions controller with the following:

```
def create
    user = User.find_by_email(params[:email])
    if user && user.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to root_path
    else
      flash[:loginerror] = "Invalid Email or Password"
      render :new
    end
  end
	
	...
	
def destroy
	session[:user_id] = nil
	flash[:success] = "You have logged out"
	redirect_to login_path
end
```

I also gave users the option of logging in with GitHub using OmniAuth. Here's the sessions controller logic:

```
def create_with_github
	user = User.find_or_create_by(uid: auth['uid']) do |u|
		u.email = auth['info']['email']
		u.password = u.password_confirmation = SecureRandom.urlsafe_base64(n=6)
	end
	session[:user_id] = user.id
	redirect_to root_path
end
```

Once you're logged in, you're greeted with a home page that shows all the color schemes you've created and all of your favorite color schemes. From there you can browse through color schemes, create new color schemes, and favorite color schemes. 

To select your colors when you're creating a color scheme I used a JavaScript color picker plugin ( https://tovic.github.io/color-picker/ ).

For the favoriting system, I set up a favorite_color_schemes join model / table with the following associations in my User model:

```
has_many :favorite_color_schemes
has_many :favorites, through: :favorite_color_schemes, source: :color_scheme
```

and, in my ColorScheme model: 

```
has_many :favorite_color_schemes, dependent: :destroy
has_many :favorited_by, through: :favorite_color_schemes, source: :user
```

And then to favorite and unfavorite color schemes I had a Favorites Controller with the following logic:

```
def update
    @color_scheme = ColorScheme.find(params[:id])
    type = params[:type]
    if type == "favorite"
      current_user.favorites << @color_scheme
      flash[:success] = "Favorited!"
      redirect_back(fallback_location: user_favorites_path)
    elsif type == "unfavorite"
      current_user.favorites.delete(@color_scheme)
      flash[:success] = "Unfavorited!"
      redirect_back(fallback_location: user_favorites_path)
    else
      flash[:error] = "Something went wrong"
      redirect_back(fallback_location: user_favorites_path)
    end
  end
```

And my "favorite" / "unfavorite" links looked like this:

```
<%= link_to "Favorite", user_favorite_path(current_user, color_scheme, type: "favorite"), method: :put %>
<%= link_to "Unfavorite", user_favorite_path(current_user, color_scheme, type: "unfavorite"), method: :put %>
```

Color schemes that have been favorited the most show up on a popular color schemes page. To achieve that I used the following class method in my FavoriteColorScheme model:

```
def self.most_popular
    fav_schemes = self.group('color_scheme_id').order('color_scheme_id asc').limit(20)
    fav_schemes.map do |fs|
      ColorScheme.find_by(id: fs.color_scheme_id)
    end
  end
```

And then I just iterate through the result of that method to display each scheme on the associated view page.

As my first Ruby on Rails app, I feel pretty good about this project overall and I'm happy with how it turned out. It was a lot of fun to make and I'm looking forward to my project review to see where I can improve!





