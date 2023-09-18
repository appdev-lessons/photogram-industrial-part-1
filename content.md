# Photogram Industrial Part 1: Devise accounts and photos scaffold

## Walkthrough video

<div class="bg-red-100 py-1 px-5" markdown="1">
**Please note**, the video is from a previous iteration of the project, so there are some differences:

- The audio quality is somewhat poor on this video, but it is _much better_ on the rest of the videos in this project series. The text below should guide you through the lesson if you can't make out the audio at times.
- I am using Gitpod as my cloud editor, so the interface looks a bit different.
- Anything contained in the project "README" is now contained in this Lesson
- I use `bin/server` to start my live app preview, _you_ should use `bin/dev`
- I use `rails db:migrate`, _you_ should use `rake db:migrate`
</div>

Did you read the differences above? Good! Then [here is a walkthrough video for this project.](https://share.descript.com/view/iJTnIgxaV0n)

**As you watch the video, pause it frequently, read the associated text, and type out the code.**

## Getting started

This project does include a few basic automated tests, so click on this button to get started:

LTI{Load Photogram Industrial assignment}(https://grades.firstdraft.com/launch)[S9ymPy6WCsn18gLbByVbZQ7k]{vfdtzJb5bLYqYwuqgeRKpc5d}(10)[Photogram Industrial Project]

The current project `photogram-industrial` covers everything in this lesson and the next three lessons: _Photogram Industrial Parts 2, 3, and 4_. Keep the project open when you move onto those lessons in order to build on your progress through the series.

Here is a rough target to work towards:

[pg-industrial-2-final.herokuapp.com](https://pg-industrial-2-final.herokuapp.com/).

This time around, Photogram will be _industrial grade_ — the kind of code you could charge money for. We'll use database indexes and constraints, advanced association accessors, scopes, validations, view helper methods like `link_to` and `form_with` everywhere, partials to DRY up code judiciously, the Devise gem for authentication and password reset emails, authorizing access to each action explicitly, and many other industrial-strength upgrades.

This is like finishing school. We're going to learn how to level up to write a codebase that we can onboard professional developer to.

Launch the codespace for your forked project and get the live preview running with `bin/dev`.

Also, go to the settings of your forked repository on `github.com/YOUR_USERNAME/photogram-industrial` and add your instructors as collaborators ("Settings" tab, then "Manage Access").

We're going to start leaving  feedback for you in the form of comments on your pull requests. You're going to start adopting the professional git workflow, where you submit pull requests for your branches, and receive line-by-line comments on your code.

[Here is a cheat sheet for our git workflow.](https://learn.firstdraft.com/lessons/196-git-cli)

We're going to practice the workflow for each feature that we're working on of creating a branch, committing to it, and merging it back to `main`.

To remind you, here is the data model from Photogram:

![](/assets/pg-erd.png)

Importantly, there's the `FollowRequest` table, which keeps track of who's following whom. We have a status column in the `FollowRequest` because this is going to be a permissioned social network. When somebody sends a `FollowRequest`, we're going to start it off as "pending", and the recipient of that request has to update that to "accept" it before the follower can actually see their posts.

## User accounts with Devise

Let's begin in our blank app by adding accounts with Devise. Open your `Gemfile` in the root directory and look for the `gem "devise"` line. 

If it's not there, add this gem now. Don't put it in one of the `:development` or `:test` group code blocks, put it outside of these blocks. We want the Devise gem available everywhere, not just when we are in development or test environments.

For example, any gems that you put in the `:development` group are just things we use while developing:

```ruby
# Gemfile

group :development do
  gem "annotate"
  gem "better_errors"
  gem "binding_of_caller"
  gem "grade_runner"
  gem "pry-rails"
  gem "rails_db"
  gem "rails-erd"
  gem "rufo"
  gem "specs_to_readme"
  gem "web_git"
end
```

These are things like our `better_errors` page for debugging, `annotate` to add column information on the models, etc. We don't want these gems to be loaded in the `:production` environment when we deploy our app to users. It saves memory not to have these loaded in production. That's why we have these gem `groups`. It allows us to specify gems that we want to use in production versus development versus all of the time. 

With the `gem "devise"` line added _outside_ of any group, we can go to a terminal tab and run the usual commands to install gems and Devise.

(Consider [clearing your terminal](https://learn.firstdraft.com/lessons/31#clear-terminal) before you run any of these commands to clear old output, so you can clearly see any instructions or error messages when the command runs.)

```
bundle install
```

Then:

```
rails generate devise:install
```

If you are asked to overwrite the file when you run these commands, then you can say yes (`Y` or `a` for "yes to all"). 

The `devise:install` command outputs a list of instructions in the terminal that you should follow. One item on the list is defining a root route in your controller. We don't have any resources yet, but soon `photos#index` will work, so add that:

```ruby{4}
# config/routes.rb

Rails.application.routes.draw do
  root "photos#index"
  # ...
```

Once you make all of the Devise changes, you can make a:

  - `git add -A`, 
  - then a `git commit -m "install devise and add root"`, 
  - and possibly even a `git push` 
  
in succession at the terminal now to save your work on your GitHub repo fork. (The first two commands can be shortened to just one command `g acm "install devise and add root"`, then `g p` for pushing using [the shortcut aliases that we setup for you](https://learn.firstdraft.com/lessons/196-git-cli#git-aliases).)

We just made that commit and push on the `main` branch! Oops! We want to get in the habit of branching and merging, which is the proper git workflow.

Before we go on, let's make our first git branch to get into the habit of our new workflow before we add anything else to the app.

Create a branch at the terminal bash prompt by running (replace `<your-initials>` with your initials, e.g. `rb-create-database`):

```
git checkout -b <your-initials>-create-database
```

(The shortcut for that is just `g cob <branch-name>`.)

Now we will be switched to our new feature branch to work on, commit to, push to GitHub, and eventually merge to `main`.

We can begin by generating the `users` table at the terminal:

```
rails g devise user username private:boolean likes_count:integer comments_count:integer
```

As a reminder:

 - `g` is short for `generate` in the `rails` command above, like `c` is short for `console`.
 - I dropped `:string` after `username` because `string` is the default datatype.

Why not a commit to get things started: `g acm "generated users with devise"`

(Long version: 

`git add -A` then 

`git commit -m "generated users with devise`.)

## Users migration file

Before we migrate the `users` table to our database, let's open the migration file and explore some values:

```ruby
# db/migrate/<date-time-of-migration>-devise-create-users.rb

# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""
# ...
```

This is the long migration that Devise wrote on our behalf when we generated the `users`. It's adding email and encrypted password columns for us automatically. If you look down the file, there are also columns that it uses internally for handling the forgotten password flow:

```ruby
# ...
      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at
# ...
```

There's even some optional columns (commented out by default) to make the users `# Trackable` `# Confirmable`, and `# Lockable`, very nice!

All the way at the bottom, we can find the columns that _we_ specified in the generation:

```ruby
# ...
      t.string :username
      t.boolean :private
      t.integer :likes_count
      t.integer :comments_count
# ...
```

Before I run this migration, I want to make a couple of changes. First, I want to set default values with these columns of zero:

```ruby{4:(29-40),5:(32-43)}
# ...
      t.string :username
      t.boolean :private
      t.integer :likes_count, default: 0
      t.integer :comments_count, default: 0
# ...
```

Whenever you generate a model or scaffold, it's a good idea to come into the migration file in `db/migrate/` and think about default values on each column. Usually, for numerical columns, like `likes_count` or `comments_count`, a starting value of 0 makes sense (rather than the default of `nil` if we don't set anything when we instantiate a new instance).

Another nice thing we can do is shown on these lines at the bottom of the migration file:

```ruby{2,3}
# ...
    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```

Devise added these `add_index ...` lines. This is a very important concept in database design. The index is how we speed up lookups of records. It's just like the index in a long book! Without it, Rails has to scan through every "page" of the database to find the record. With the index, the database creates a separate record keeping area with the indexes that we specify, so we can look up all of those very quickly. 

This is important for primary keys, so primary keys are automatically indexed on the database. No need to add any lines to the migration file. If you have any other columns that you plan to look up records by (usually things like `email` or `username`), then it's a good idea to add indexes to those columns (although you can always add them later if you notice slowness). The `email` index was already added for us, but let's add one for our `username`s:

```ruby{6}
# ...
    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
    add_index :users, :username,             unique: true
  end
end
```

In addition, we (and Devise) have used the `unique: true` option for these columns. This will add a _database constraint_ enforcing uniqueness within the column at the database level, which is a stronger guarantee than an `ActiveRecord` validation. 

<aside markdown="1">
An `ActiveRecord` model validation still allows for ["race conditions"](https://en.wikipedia.org/wiki/Race_condition). Hence, a database constrain on uniqueness is important to add here.
</aside>

Devise knows that we want both an index and a uniqueness constraint for `email` since that's what we uniquely identify and look up accounts by. In this app, we decided to have a `username` column that we're probably going to be using similarly. Therefore, _we_ (_not_ Devise) had to add an index and a uniqueness constraint for it.

An advanced optimization that we can make, is to use a case-insensitive column for `username`. That way, when doing lookups, we won't have to worry about `RaGhU` not matching `raghu`, or normalizing by downcasing or upcasing before every lookup; the database will take care of it for us. [You can read more here.](https://mikecoutermarsh.com/storing-email-in-postgres-rails-use-citext/)

We can enable this feature by adding a line at the very top of the `change` method in the migration file, then we can change the `email` and `username` column to use it:

```ruby{5,7:(9-14),12:(9-14)}
# ...
class DeviseCreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      enable_extension("citext")
      ## Database authenticatable
      t.citext :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""
      
      # ...
      
      t.citext :username
      t.boolean :private
      t.integer :likes_count, default: 0
      t.integer :comments_count, default: 0
#...
```

This is an example of a database-specific feature. Previously, we used a lightweight database called **SQLite** that did not have `citext` column support. **PostgreSQL**, the professional database that we are using now, has many other excellent features (JSON datatype, range datatype, ordering by geographic distance, full-text search), and Rails provides first-class support for many of them; [see this Rails Guide for a rundown](https://guides.rubyonrails.org/active_record_postgresql.html).

<aside markdown="1">
Pretty much every device in the world has SQLite installed on it, including hardware devices. Probably if you have any smart device in your home, even a light bulb, it has SQLite installed on it. This database is universal, and a nice way for us to get started, but we're upgrading now to PostgreSQL. This is a very powerful database and we're going to use it now in development mode, as well as production, so that we have access to all of the features.
</aside>

When you're satisfied with your migration, `rake db:migrate` and `git commit`, perhaps using our shortcut. The message should be something succinct that describes the incremental change, like "generated users" or "edited and migrated devise users".

<div class="bg-red-100 py-1 px-5" markdown="1">

If you get any error messages during the database migration, then you can: `rake db:drop` your database (if `rake db:rollback` isn't working), then run `rake db:create` before doing any migrations to set the database up. 

Don't worry, if you get error messages trying to migrate, then read the message and you should get a clue to this step.
</div>

## Photos resource

Now it's time to start generating the rest of our data model. Typically, I generate my `users` first because I'm going to have a lot of associations between `users` and everything else in almost every application. 

Let's revisit the ERD:

![](/assets/pg-erd.png)

The question you have to answer now is: for each of these tables, do you want to generate a `scaffold` or do you just want to generate a `model`? How do we figure that out? 

My usual rule of thumb:

 - If I will need routes and controller/actions for users to be able to CRUD records in the table, then I probably want to generate `scaffold`. (At least some of the generated routes/actions/views will go unused. I need to remember to go back and at least disable the routes, and eventually delete the entire RCAVs, at some point; or I risk introducing security holes.)
 - If the model will only be used on the backend, e.g. by other models, then I probably want to generate `model`. For example, a `Month` model where I will create all twelve records once in `db/seeds.rb` does not require routes, `MonthsController`, `app/views/months/`, etc.

In this case, since users will be CRUDing all of the remaining resources, we'll `scaffold` them all. 

Let's generate the photos resource first: 

```
rails g scaffold photo image comments_count:integer likes_count:integer caption:text owner:references
```

Notice that I used `owner:references` as the foreign key column name and datatype, instead of what you might have been expecting, `owner_id:integer`. (An alias for `owner:references` is `owner:belongs_to`. They mean the same thing.) Go take a look at the generated migration file. First, be sure to add some default values to the `_count` columns:

```ruby{7:(32-43),8:(29-40),10}
# db/migrate/<date-time-of-migration>-create-photos.rb

class CreatePhotos < ActiveRecord::Migration[7.0]
  def change
    create_table :photos do |t|
      t.string :image
      t.integer :comments_count, default: 0
      t.integer :likes_count, default: 0
      t.text :caption
      t.references :owner, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```

If we ran this migration as-is,

 - Even though it says `t.references` instead of the usual `t.integer`, the datatype would be `integer` (or whatever the default datatype is for primary keys for the database you are using; [I commonly use UUIDs these days](https://pawelurbanek.com/uuid-order-rails)).
 - The column name would be `owner_id` rather than `owner`, since `t.references` knows the convention we want to follow.
 - A database constraint would be added preventing the column from being blank. If you want to allow this foreign key column to be blank, which is sometimes the case, then you should delete the `null: false` option.
 - We _don't_ need to add an `index: true` option to the `owner_id` column, because Rails adds this lookup index by default. That's good, because we'll often look up photos by their `owner_id`, or filter the photo table by `owner_id`.

Go ahead and try to `rake db:migrate` now.

Uhoh! Can you spot the helpful error message?

```
... relation "owners" does not exist
```

That's because we departed from conventional naming here! Our table that we are associating with `photos` is `users`, but we are associating it is as `owner`. 
 
If you head over to `app/models/photo.rb`, you'll notice that a `belongs_to :owner` association accessor was automatically added: 

```ruby{4}
# app/models/photo.rb

class Photo < ApplicationRecord
  belongs_to :owner
end
```

That association isn't quite right, is it? Because the other model name is `User`, not `Owner`; we just chose to use a more descriptive foreign key column name than `user_id`.

So, the generator tried to be helpful, but couldn't know that we went off-convention with our foreign key column name. Update the association accessor to be correct:

```ruby{4:(20-39)}
# app/models/photo.rb

class Photo < ApplicationRecord
  belongs_to :owner, class_name: "User"
end
```

And while we're at it with association accessors, we should add the `has_many` side to the `User` model (noting all the nice Devise additions):

```ruby{9}
# app/models/user.rb

class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  
  has_many :own_photos, class_name: "Photo", foreign_key: "owner_id"
end
```

Similarly, we need to update the migration file to point the foreign key to the correct table:

```ruby{6:(41-73)}
# db/migrate/<date-time-of-migration>-create-photos.rb

class CreatePhotos < ActiveRecord::Migration[7.0]
  # ...
      t.text :caption
      t.references :owner, null: false, foreign_key: { to_table: :users }
  # ...
```

The `to_table:` key in the hash allows us to supply the table name that `owner` should point to.

When you're satisfied, `rake db:migrate`. Then commit with a `g acm "Generated photos"` at the terminal. And you could even push the changes up to your repo with a `g p` (a.k.a. `git push`).
