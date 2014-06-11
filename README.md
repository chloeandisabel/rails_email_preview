# Rails Email Preview [![Build Status][travis-badge]][travis] [![Gem Version][gem-badge]][gem] [![Code Climate][codeclimate-badge]][codeclimate]

A Rails Engine to preview plain text and html email in your browser. Compatible with Rails 3 and 4.

Preview:
![screenshot][rep-show-screenshot]

List:
![screenshot][rep-nav-screenshot]

REP can use the application styles and comes with [a bootstrap 3 theme][rep-show-default-screenshot] by default (default markup for [twbs/bootstrap](twbs/bootstrap) 3).
*These screenshots are from [Zuigo](http://zuigo.com/), a platform to organize & discover events; using REP default markup and app styles*

## How to

Add to Gemfile

```ruby
gem 'rails_email_preview', '~> 0.2.19'
```

REP handles setup for you:

```ruby
# adds initializer and route:
rails g rails_email_preview:install

# generates preview classes and method stubs in app/mailer_previews/:
rails g rails_email_preview:update_previews
```

This last generator will add a stub for each of your emails, then you populate the stubs with mock data:

```ruby
# app/mailer_previews/user_mailer_preview.rb:
class UserMailerPreview
  # preview methods should return Mail objects, e.g.:
  def invitation        
    UserMailer.invitation mock_user('Alice'), mock_user('Bob')
  end
        
  def welcome                
    UserMailer.welcome mock_user                            
  end
  
  private
  # You can put all your mock helpers in a module
  # or you can use your factories / fabricators, just make sure you are not creating anythin
  def mock_user(name = 'Bill Gates')
    fake_id User.new(name: name, email: "user#{rand 100}@test.com")
  end
  
  def fake_id(obj)
    # overrides the method on just this object
    obj.define_singleton_method(:id) { 123 + rand(100) }
    obj
  end
end
```


## Routing

You can access REP urls like this:

```ruby
# engine root:
rails_email_preview.rep_root_url
# list of emails (same as root):
rails_email_preview.rep_emails_url
# email show:
rails_email_preview.rep_email_url('user_mailer-welcome')
```

## Send Emails

You can send emails via REP. This is especially useful when testing with limited clients (Blackberry, Outlook, etc.).
This will use the environment's mailer settings, but the handler will `perform_deliveries`.
Uncomment this line in the initializer to disable sending test emails:

```ruby
config.enable_send_email = false
```

## Email editing

You can use [comfortable_mexican_sofa](https://github.com/comfy/comfortable-mexican-sofa) for storing and editing emails.
REP comes with an integration for it -- see [CMS Guide](https://github.com/glebm/rails_email_preview/wiki/Edit-Emails-with-Comfortable-Mexican-Sofa).

[![screenshot](https://raw.github.com/glebm/rails_email_preview/master/doc/img/rep-edit-sofa.png)](https://github.com/glebm/rails_email_preview/wiki/Edit-Emails-with-Comfortable-Mexican-Sofa)


## Premailer

[Premailer](https://github.com/alexdunae/premailer) automatically translates standard CSS rules into old-school inline styles. Integration can be done by using the <code>before_render</code> hook.

To integrate Premailer with your Rails app you can use either [actionmailer_inline_css](https://github.com/ndbroadbent/actionmailer_inline_css) or [premailer-rails](https://github.com/fphilipe/premailer-rails).
Simply uncomment the relevant options in [the initializer](https://github.com/glebm/rails_email_preview/blob/master/config/initializers/rails_email_preview.rb). *initializer is generated during `rails g rails_email_preview:install`*

## I18n

REP expects emails to use current `I18n.locale`:

```ruby
# current locale
AccountMailer.some_notification.deliver     
# different locale
I18n.with_locale('es') { InviteMailer.send_invites.deliver }

```
When linking to REP pages you can pass `email_locale` to set the locale for rendering:

```ruby
# will render email in Spanish:
rails_email_preview.root_url(email_locale: 'es')
```


If you are using `Resque::Mailer` or `Devise::Async`, you can automatically add I18n.locale information when the mail job is scheduled 
[with this initializer](https://gist.github.com/glebm/5725347).

REP displays too many locales? Make sure to set `config.i18n.available_locales`, since it defaults to *all* locales in Rails.

User interface is available in English and German (Danke, @baschtl).
You can set the language in `config.to_prepare` section of the initializer, default is English.

```ruby
# config/initializers/rails_email_preview.rb
RailsEmailPreview.locale = :de
```

## Views

You can render all REP views inside your app layout (this will need styling to look nice if you don't use bootstrap):

```ruby
Rails.application.config.to_prepare do
  # Use admin layout with REP (this will also make app routes accessible within REP):
  RailsEmailPreview.layout = 'admin'
end
```

REP allows you to customize some of the element classes via `RailsEmailPreview.style`:

```ruby
{
  btn_default_class:     'btn btn-default',
  btn_active_class:      'btn btn-primary active',
  btn_group_class:       'btn-group',
  list_group_class:      'list-group',
  list_group_item_class: 'list-group-item',
  panel_class:           'panel',
  row_class:             'row',
  column_class:          'col-%{n}'
}
```

E.g., to change column class from `col-` to `column-` everywhere where REP uses columns:

```ruby
RailsEmailPreview.style[:column_class] = 'column-%{n}'
```

You can `//= require 'rails_email_preview/layout'` REP-specific styles (`@import 'rails_email_preview/layout'` for SASS).

You can also override any individual view by placing a file with the same path in your project's `app/views`,
e.g. `app/views/rails_email_preview/emails/index.html.slim`. *PRs accepted* if you need hooks. 

## Authentication & authorization

You can specify the parent controller for REP controller, and it will inherit all before filters.
Note that this must be placed before any other references to REP application controller in the initializer (and before `layout=` call):

```ruby
RailsEmailPreview.parent_controller = 'Admin::ApplicationController' # default: '::ApplicationController'
```

Alternatively, to have custom rules just for REP you can:

```ruby
Rails.application.config.to_prepare do
  RailsEmailPreview::ApplicationController.module_eval do
    before_filter :check_rep_permissions
  
    private
    def check_rep_permissions
       render status: 403 unless current_user && can_manage_emails?(current_user)
    end
  end
end 
```


This project rocks and uses MIT-LICENSE.

  [rep-nav-screenshot]: https://raw.github.com/glebm/rails_email_preview/master/doc/img/rep-nav.png "Email List Screenshot"
  [rep-show-screenshot]: https://raw.github.com/glebm/rails_email_preview/master/doc/img/rep-show.png "Show Email Screenshot"
  [rep-show-default-screenshot]: https://raw.github.com/glebm/rails_email_preview/master/doc/img/rep-show-default.png "Show Email Screenshot (default styles)"


[travis]: http://travis-ci.org/glebm/rails_email_preview
[travis-badge]: http://img.shields.io/travis/glebm/rails_email_preview.svg
[gem]: https://rubygems.org/gems/rails_email_preview
[gem-badge]: http://img.shields.io/gem/v/rails_email_preview.svg
[codeclimate]: https://codeclimate.com/github/glebm/rails_email_preview
[codeclimate-badge]: http://img.shields.io/codeclimate/github/glebm/rails_email_preview.svg
[coveralls]: https://coveralls.io/r/glebm/rails_email_preview
[coveralls-badge]: http://img.shields.io/coveralls/glebm/rails_email_preview.svg
