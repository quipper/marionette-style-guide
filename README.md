# Marionette + Backbone Style Guide

This style guide for Marionette and Backbone was written so that our development team would be on the same page for coding standards, best practices, patterns and anti-patterns when writing Marionette apps in CoffeeScript, especially since there was no formal style guide for Marionette app development.

It is made available to the developer community for sharing and improvement. Contribution is welcome and encouraged.

## CoffeeScript

We use CoffeeScript for writing our code, instead of vanilla JavaScript, allowing us to focus our attention on problem-solving.

This document does not define a style guide for CoffeeScript itself, as there are plenty of good ones already written. Use [polarmobile's CoffeeScript Style Guide](https://github.com/polarmobile/coffeescript-style-guide) for reference.

## Modules

### Grouping

Keep your app modular. Instead of putting all views in a single "Views" namespace (and so on with models etc), use modules to break your app into components, giving each one its necessary views and models.

```coffeescript
# Bad
App.module 'Views', (Views) ->
  class Views.UserView extends Marionette.ItemView
  class Views.UsersView extends Marionette.CollectionView
  class Views.SettingView extends Marionette.ItemView

App.module 'Models', (Models) ->
  class Models.User extends Backbone.Model
  class Models.Setting extends Backbone.Model

App.module 'Collections', (Collections) ->
  class Collections.User extends Backbone.Collection

# Good
App.module 'Users', (Users) ->
  class Users.UserView extends Marionette.ItemView
  class Users.UsersView extends Marionette.CollectionView
  class Users.User extends Backbone.Model
  class Users.Users extends Backbone.Collection

App.module 'Settings', (Settings) ->
  class Settings.SettingView extends Marionette.ItemView
  class Settings.Setting extends Backbone.Model
```

### Shared models

If a model is considered very "global" such that it is shared between many modules, then it is best to define it in its own namespace.

### Naming

A module should be named after its main concern. If the concern relates to an object such as a primary model, then prefer to pluralise the name.

```coffeescript
# Bad
App.module 'UserPage', (UserPage) ->
  class UserPage.UserPageView extends Marionette.ItemView
  
# Good
App.module 'Users', (Users) ->
  class Users.UserView extends Marionette.ItemView
```

Keep areas that are related to a higher-level modules as submodules of it.

```coffeescript
# Bad
App.module 'UsersSettings', (UserSettings) ->
  class UserSettings.SettingsView extends Marionette.ItemView

# Good
App.module 'Users.Settings', (Settings) ->
  class Settings.SettingsView extends Marionette.ItemView
```

### File structure

Although models and views should be grouped within the module that they refer to, it can be useful to split the directory structure into `views` and `models`.

```
modules/
  users/
    views/
      user_view.js.coffee
      users_view.js.coffee
    models/
      user.js.coffee
      users.js.coffee
    router.js.coffee
    controller.js.coffee
```

## Routers

A module should only ever need at most one router, defining the routes necessary for that module to function.

```coffeescript
# Bad
App.module 'Routes', (Routes) ->
  class Routes.Router extends Marionette.Router
    appRoutes:
      # five million routes defined here

# Good
App.module 'Users', (Users) ->
  class Users.Router extends Marionette.Router
    appRoutes:
      "users/:id" : "show"
      "users" : "index"

App.module 'Settings', (Settings) ->
  class Settings.Router extends Marionette.Router
    appRoutes:
      "settings" : "index"
```

## Controllers

A module should only ever need at most one controller. Modules with multiple routes and actions can organise themselves better by splitting into submodules, with each submodule defining its own controller to handle the request.

```coffeescript
# OK
# modules/users/controller
App.module 'Users', (Users) ->
  class Users.Controller
    show: ->
      app.pageRegion.show(new Users.UserView)
      
    edit: ->
      app.pageRegion.show(new Users.EditUserView)

# Better
# modules/users/controller.js.coffee
App.module 'Users', (Users) ->
  class Users.Controller
    show:  -> new App.Users.Show.Controller().show()
    edit:  -> new App.Users.Edit.Controller().edit()

# modules/users/show/controller.js.coffee
App.module 'Users.Show', (Show) ->
  class Show.Controller
    show: ->
      app.pageRegion.show(new Show.UserView)

# modules/users/edit/controller.js.coffee
App.module 'Users.Edit', (Edit) ->
  class Edit.Controller
    edit: ->
      app.pageRegion.show(new Edit.UserView)
```

## Views

### Naming

All views have the class suffix `View`, but should never be called simply `View` even if they are the only or primary view of a module.

```coffeescript
# Bad
App.module 'Simple', (Simple) ->
  class Simple.View extends Marionette.ItemView

# Good
App.module 'Simple', (Simple) ->
  class Simple.SimpleView extends Marionette.ItemView
```

### Callbacks

Favour using `onRender` instead of `onShow` if you want the callback to run every time the view is rendered with `view.render()`. This is particularly useful during isolated tests where you may not be rendering the view within a region.

```coffeescript
# OK
class ThingView extends Marionette.ItemView
  onShow: ->
    @$('.thing-i-must-hide').hide()

view = new ThingView()
view.render().onShow() # because we have no region


# Better
class ThingView extends Marionette.ItemView
  onRender: ->
    @$('.thing-i-must-hide').hide()

view = new ThingView()
view.render()
```


## Regions

### Naming

Name your regions with the suffix `Region` so that they do not get confused with other fields.

```coffeescript
# Bad - @name could get confusing
@name.show(@nameView)

# Good - @nameRegion is obviously a region
@nameRegion.show(@nameView)
```
