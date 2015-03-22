---
layout: post
title: "Dangerous attribute auto assignment by cancan that Rails developers should be awared of!"
date: 2015-03-22
image: /images/cancan_note.png
tags: cancan cancancan rails ruby
---

***Cancan can save you a lot of time but sometime it makes you careless enough to produce hard-to-find bugs.***

In web development, an application usually has multiple user roles such as admin, customer... Each role has a particular permission on some actions and objects. Permission management has become a well-known problem.

As a Rails developer, I use [cancancan - community version of cancan](https://github.com/CanCanCommunity/cancancan) to manage user permissions and I know a lot of Rails developers out there are using it too. It's a handy gem, easy to use and provides permission checking on Model objects out of the box.

In controller, `cancan` allows you to load corresponding resource and check permission of `current_user` (if you are using `devise`). If `current_user` doesn't have permission on particular action (such as `update`), it immediately stop the code and does something like a redirect or whatever you want it to do. With `cancan`, your code is more or less like this:

You will define users' permission in ability model:

```ruby
class Ability
  include CanCan::Ability

  def initialize(user)
    # initialize user object if user is nil
    user ||= User.new

    # admin can do whatever he wants on every object
    if user.admin?
      can :manage, :all
    else
    # other users can manage only their posts
      can :manage, Post, user_id: user.id
    end
  end
end
```

In your controller, you are likely to load the resource and authorize it:

```ruby
class PostsController < ApplicationController
  # get the post with params[:id] and assign it to @post
  # after that, check if @post.user_id is equal to current_user.id or not
  # if it is, the code runs normally, otherwise redirect to root or something
  # else you want.
  load_and_authorize_resource :post

  def create
    # @post is created by cancan before
    if @post.save
      redirect_to @post, notice: 'Your post has been created.'
    else
      render :new
    end
  end

  private
  def post_params
    params.require(:post).permit(:text)
  end

end
```

## The cancan mysterious assignment

As you can see in the code above, I didn't assign the user to that post. Really? This is so wrong. Let's run the test.

```ruby
describe AdUnitsController do
  describe "POST create" do
    it "creates a post with current_user associated" do
      user = FactoryGirl.create(:user, role: "customer")
      # log the user in
      sign_in user
      valid_attributes = {
        text: "This is a test post body"
      }
      post :create, post: valid_attributes

      # get the post created inside controller
      created_post = assigns(:post)

      expect(created_post.persisted?).to be true
      # the post's user is expected to be the logged in user
      expect(created_post.user_id).to eq user.id
    end
  end
end
```

I ran this test and it passed. HAHA. So I didn't have to assign user to @post myself, it was done by `cancan` transparently. It's amazing. Is it? Might be. I feel a little bit confused and doubtful here.

## The problem

After I ran the test, I thought that my code works properly but I wanted to check it one last time on the browser. This time, I already signed under an admin user, I tried to create a post and it was unable to save the post.

What? Why? Being confused, I started running the test repeatedly with stupid print-outs and wondered ***Why did the test pass when it actually doesn't work correctly?***

It turns out that `user_id` was never be assigned on `@post`, it was nil so it didn't pass the validation. But wait, `user_id` is assigned in the test, why isn't it here?

***How the hell does cancan know that I want to assign user to current_user? What causes this inconsistency?***

After a while reading `cancan` source code, I found that it automatically assigns some attributes to reasonable values without notice to developers. In my case, it assigns `user_id` to `current_user.id`. Why? Because I defined a rule to check `current_user` permission on this `@post`, the rule says `:user_id => user.id` so cancan thinks that I want to assign `user_id` to `current_user.id` on `@post` and it's right. I do want that.

But that rule is applied only when the `current_user` is not admin. On the other casewhere `current_user` is an admin, `cancan` does no pre-assignment so `user_id` is not set on the fly. This is hard for developers to notice the difference because it's done silently by `cancan`.

## Test agaist that problem

To make a failing test for this, you have to test the action on both admin and customer role like this:

```ruby
describe AdUnitsController do
  describe "POST create" do
    context "customer" do
      it "creates a post with current_user associated" do
        user = FactoryGirl.create(:user, role: "customer")
        test_post_create(user)
      end
    end

    context "admin" do
      it "creates a post with current_user associated" do
        user = FactoryGirl.create(:user, role: "admin")
        test_post_create(user)
      end
    end

    def test_post_create(user)
      # log the user in
      sign_in user
      valid_attributes = {
        text: "This is a test post body"
      }
      post :create, post: valid_attributes

      # get the post created inside controller
      created_post = assigns(:post)

      expect(created_post.persisted?).to be true
      # the post's user is expected to be the logged in user
      expect(created_post.user_id).to eq user.id
    end
  end
end
```

## Conclusion

  * `cancan` might pre-assign attributes it needs for permission checking to the comparing values. If you don't assign them, they **might** be assigned by `cancan` and **might be not**. Your test **might** pass and **might not** pass.
  * Always manually assign object attributes to expected values instead of depending on `cancan`.
  * It's good to have tests on the same action with different user roles.
