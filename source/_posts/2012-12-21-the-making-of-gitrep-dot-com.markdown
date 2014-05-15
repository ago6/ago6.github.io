---
layout: post
title: "The Making of GitRep.com"
date: 2013-02-03 14:40
comments: true
categories: tech
---

For the longest time I had an annoying problem; I found it difficult to find and compare open source software. What's the best jQuery library for x, or a more updated version of gem y?

After discovering I was not alone in this torture, I spent a few weeks building and launching [GitRep](http://GitRep.com). Read on to learn a little about some decisions, both technical and business, made in its making and its launch.

<!--more-->

##The Need

With Ruby it's less difficult to find popular gems thanks to the makeup of the community and existing tools, however I found this type of discovery to be more difficult when it came to other languages. And still in Ruby, I wasn't able to explore and compare various solutions as easily as I would have liked.

I didn't think much about the problem, thinking that as I become more ingrained with open source software I will become more knowledgeable of the tools available. But as time went on, I would notice in tweets, posts, and questions complaints that were similar to mine. This lead me to realize that maybe I wasn't the only one sparring with this issue, and maybe "waiting to travel the road to know what is around" isn't the best way to become aware of your options. Maybe this should be addressed, it seems a solution could be of value to people other than just myself.

##Back of the Napkin Design

So I began thinking of what a first stab at a solution for this would look like. One approach I've seen is to place gems/libraries/plugins/packages/etc. (referred to simply as "repos" going forward) into categories. The issue I had with the approach of categorizing repos is that it would be almost impossible for me to sit down and think of all the appropriate categories that repos could fall into. As I wanted the site to be as community-driven as possible, and that community-driven tagging seemed to fit the bill of being flexible, I felt that that was a good place to start.

Another feature I thought would be helpful would be the ability to see repos that are similar to any given repo. That way one could easily compare and discover different options. There were two "repo discovery" methods I wanted to implement; repos similar according to their tags and descriptions, and "users who starred this repo also starred these other repos" type associations. I felt that these two methods were a good way to discover other relevant repos. Currently only the former is implemented, the latter should be implemented in the next few weeks.

It goes without saying that it must have good searching functionality. I want to be able to search by tags and text descriptions, and be able to easily compare results by how popular they are (star count), and how active development is (created date, updated date, fork count).

Another consideration is that it must be low-friction and easy to use. One click sign-in with your existing GitHub account, one click to tag items, tag suggestions, clicking on tags to see other items with the same tag, etc.. Get the site out of the user's way as much as possible.

I thought that this was a good design to start with. I didn't want to make a huge monolithic app in isolation with every single feature imaginable. Since I am one of the billions of people who read Lean Startup, I knew it was important to build a rough idea and get it in the hands of the community and see if and where they find value and iterate accordingly.

##Building, and the Technology

Now that I had an idea of what I wanted to build, I needed to decide how I was going to build it. Most of my stack was going to be pretty conventional [Rails "Prime" Stack](http://words.steveklabnik.com/rails-has-two-default-stacks); Ruby on Rails with haml to cut down on typing, Unicorn for its performance (and letting me squeeze the most out of each Heroku dyno), Postgresql for it's maturity, Heroku for it's ease of use.

For monitoring, New Relic was a pretty easy choice, especially on Heroku where it is popular. Installation is easy, and it gives me insight to memory usage, slow queries, downtime, etc.. It's an easy win, and can't yet think of a reason one wouldn't use it.

Twitter Bootstrap was another easy and common choice. In [GitRep.com](http://GitRep.com)'s case, it is used manually for scaffolding, inputs, and javascript stuff. jQuery was another obvious front end choice.

For signing in with your GitHub account, I turned to Devise with omniauth-github. As I didn't want just anyone or any GitHub user altering public tags, I needed user authorization. For this I turned to CanCan and Rolify. After implementing CanCan, I now have the following `ability.rb` file under `models` that allows only GitHub users older than a month to update the public tags of a repo.
{% codeblock ability.rb %}
class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new

    can :read_public_tags, Repo

    can :manage, User, id: user.id if user.id

    # if they are a github authorized user and have been for a month, they can edit public tags
    if user.github_oauth_token && user.github_created_at <= 1.month.ago
      can :update_public_tags, Repo
    end
  end
end
{% endcodeblock %}

And then check the user's abilities in the view and controller. With an example of checking in the controller something like...

{% codeblock tags_controller.rb %}
class TagsController < ApplicationController

  def update_tag_list
    @repo = Repo.find params[:repo_id]
    @user = current_user
    authorize! :update_public_tags, @repo # Makes sure the user can do this action on this Repo
    @tag_names = params[:name].split(',')
    Tag.update_tag_list({
      repo: @repo,
      user: @user,
      type: params[:type]
    }, @tag_names)

    render json: @repo.tags_public
  end
end
{% endcodeblock %}

Authorizing various actions of objects and classes for different types of users is pretty much that simple!


Throughout your codebase, and sometimes unique to some environments, there are values that make sense to centralize in a settings file instead of throughout the code. To address this I use a gem called [settingslogic](http://github.com/somelink). I just define a yaml file, and address it throughout the codebase, for example;

{% codeblock application.yml %}
defaults: &defaults
  app_name: 'GitRep'
  similar_tag_limit: 7
development:
  <<: *defaults
  host: 'localhost:5000'
  analytics:
    google_property_id: 'UA-xxxxxxx-DEVELOPMENT'
production:
  <<: *defaults
  host: 'gitrep.com'
  analytics:
    google_property_id: 'UA-31403396-2'
{% endcodeblock %}

{% codeblock app_settings.rb %}
class AppSettings < Settingslogic
  source "#{Rails.root}/config/application.yml"
  namespace Rails.env
end
{% endcodeblock %}

{% codeblock example.rb %}
AppSettings.analytics.google_property_id # this is equal to 'UA-31403396-2' in production
{% endcodeblock %}

This makes settings things like project name and other settings easy. I would also recommend looking into [Figaro](https://github.com/laserlemon/figaro) for handling sensitive things such as API keys and other things you'd want to keep in an environment variable (and out of the codebase and version control) in a similar manner as settingslogic.

Several features, such as adding and updating a user's starred repos, are done asynchronously. For asynchronous jobs I turned to sidekiq. I find it more performant than Resque, with a simple to use API that is still powerful and feature rich. For example when a user signs in I use the following user_login service;

{% codeblock user_login.rb %}
class UserLogin
  def initialize(auth)
    @auth = auth
  end

  def login
    return false unless @auth

    user = User.from_github_omniauth(@auth)

    if user.save
      UpdateUserStarredRepos.perform_async(user.id)

      user
    else
      false
    end
  end
end
{% endcodeblock %}

which then executes

{% codeblock update_user_starred_repos.rb %}
class UpdateUserStarredRepos
  include Sidekiq::Worker

  def perform(id)
    user = User.find(id)
    starred_repos = user.github_starred_repos
    Star.where(stargazer_login: user.login).delete_all

    starred_repos.each do |raw_repo|
      repo = Repo.from_github(raw_repo)
      repo.save
      Star.create(admired_id: repo.id, stargazer_login: user.login)
    end
  end
end
{% endcodeblock %}

And boom, an asynchronous job is created.

Also since I am bootstrapping this, I aimed to fit the entire application, including background jobs, into a single Heroku dyno. In my case with unicorn and sidekiq, I achieved this by starting sidekiq once with the following config file for unicorn;

{% codeblock unicorn.rb %}
worker_processes 2
timeout 30
preload_app true

@sidekiq_pid = nil

before_fork do |server, worker|
  @sidekiq_pid ||= spawn("bundle exec sidekiq -C ./config/sidekiq.yml")
end

after_fork do |server, worker|
  ActiveRecord::Base.establish_connection
  Sidekiq.configure_server do |config|
    config.redis = {
      url: AppSettings.data_store.redis,
      namespace: 'sidekiq',
      size: 7
    }
  end
end
{% endcodeblock %}

So far, this unicorn setup has been humming along.

I use caching liberally throughout the app, both in the form of view fragment caching, and good old (and awesome) [Rails.cache.fetch](http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html#method-i-fetch). For this I use memcache with the dalli gem. This is the first project where I made good use of Rails.cache.fetch, and I am a huge fan. An example from the codebase below (Ignore the Feature Envy for now, it is a candidate for refactoring);
{% codeblock tag.rb %}
class Tag
  # ....

  def self.public_tags(options = {})
    tags = if options[:cache] == false
      Repo.tag_counts_on(:public_tags)
    else
      Rails.cache.fetch(['Tag', 'public_tags'], expires_in: 10.minutes) { Repo.tag_counts_on(:public_tags) }
    end
    options[:tag_class] ? tags : tags.map { |t| t.name }
  end
{% endcodeblock %}

And something just as awesome, [key-based fragment caching](http://37signals.com/svn/posts/3113-how-key-based-cache-expiration-works)!;
{% codeblock _widget_profile_info.html.haml %}
- cache ['v1', 'users/_widget_profile_info', user] do
  %p
    Here is some cool stuff on this user that is cached until the user is updated!
  = user.name
  = image_tag(user.gravatar_url(:s => 300), :alt => user.name)
{% endcodeblock %}

Using these techniques has kept load time lower, but I still have a ways to go to optimize performance.

For testing I use rspec, machinist (may switch back to FactoryGirl), and [vcr](https://github.com/vcr/vcr) for stubbing web requests. If you use external APIs or web resources at all I would definitely recommend taking a look at [vcr](https://github.com/vcr/vcr), it makes mocking external requests a breeze.

Finally for search I am using [Elastic Search](http://www.elasticsearch.org/) with the [tire gem](https://github.com/karmi/tire). My initial approach was to load up into Elastic Search the IDs of each repo, and each repo's updated date, created date, and tags. I would then do a search against Elastic Search on those items and be returned IDs that I would use to pull the actual objects from the database and present to the user with the additional information that having the object provides. Then I happened upon a better idea; Why don't I put everything I'm interested in into Elastic Search (and keep it updated) so I don't have to go back to my database to get that information? Now, I place the ID, repo name and owner name, description, star count, fork count, created date, updated date, and tags into Elastic Search. When a user does a search, all of those items are returned from Elastic Search, which is everything I need to display to the user. No need to bug the database for additional information! Unless the search contains personal tags from a logged in user (since I don't store private tag information in Elastic Search), which is a very small percentage of searches, the database is not hit at all! This was a huge boost to performance and memory usage.

Also, using Elastic Search's filter and boolean features, finding similar repos based on the tags and descriptions is a snap.

During development, [Pry](http://pryrepl.org/) was a big asset. If you have never heard of Pry, I'd recommend checking it out as it is a powerful alternative to the standard IRB shell for Ruby.

##Launch

After I had the initial feature set completed and launched, it was time to get it in the hands of actual users to see if this solves the problem I set out to address and how to move forward. I was excited to get people using the app that I spent several weeks building in the hopes that it'd be useful. I tweeted a bit, posted a discussion on LinkedIn, and made a small post on Reddit. In the first hour, I got a few dozen users. Awesome! People were using it! I was being asked questions! People liked it! The past few weeks weren't a complete waste of time!

Then another hour or two passed, and several hundred people were using it. I was blown away. I was getting real traffic! I'm not the only person who had this problem and thought my approach was useful! This was huge for me, my project actually had legs!

Then thousands of people were using it. Now I was getting nervous; I felt like I wasn't ready for this kind of attention yet. My plan was to get some people using it, see what their thoughts and input were, and improve accordingly. Once I was able to address the input from other devs and launch a "v1.2", I'd then throw everything I had at getting it in the hands of devs. Clearly that plan went up in smoke and they were coming *now*.

There were more conversations springing up and I knew it was important to get involved. I wanted to explain what I was trying to achieve, address the critics, and solicit input, and now was the best time, possibly the only time, to do this. I started engaging as many people and points as possible. At the same time, a bottleneck or two that I missed became apparent as I was dealing with hundreds of requests per minute and adding tens of thousands of repos. User experience was paramount. If the site started to respond poorly, crash, throw errors, or do anything other than perform with flying colors, I'd be stifling possibly my largest opportunity to get real traction. While I was engaging as many conversations as humanly possible, I was also trying to elegantly and quickly address any issues that arose. It was a frantic day to say the least.

####On [GitRep](http://gitrep.com)'s first full day live, 1,493 user accounts, 72,503 repos, and 409 unique tags were created.
I honestly could not have hoped for a better first day!

##Post-Launch

Now that it is finally out the door, I am going to continue to watch how people use the site (using analytics such as Google Analytics and Mixpanel) and listen to direct feedback from UserVoice. Between the two I should get a good idea of where people are finding value and how to iterate and improve accordingly.

So far next steps include adding badges to help incentivize the user base and adding a blog to cover updates and posts of interest. I also want to invest in getting the word out. The more people using it, the more opportunity to make [GitRep](http://GitRep.com) useful!

Thanks for taking the time to read through all this! It turned out a little longer than I had expected. As always, I'm eager for any input or hearing from anyone wanting to get involved. I'm always available at adrian.r.artiles at gmail dot com!
