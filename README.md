# Selections

Selection list management and form and view helpers.

##Key Features

* Manages one table to hold all selections items/ dropdown lists ( tree )
* Dynamic lookup to find parent or children ( eg. Selection.priorities )
* Form helper to display lists ( eg. f.selections :priorities )
* Model helpers for joining tables ( eg. belongs_to_selection :priority )
* Handling of archived items ( displaying if selected only )
* Ordering of lists based on alpha or numbered
* Default item handling

## Installation

Add this line to your application's Gemfile:

    gem 'selections'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install selections

## Usage

First, you need to configure your selection model. We typically use `Selection` for this (although you
can change the name), and should be generated such as:

```ruby
rails generate model Selection name parent_id:integer system_code position_value:integer is_default:boolean is_system:boolean archived_at:datetime
```

### Selection Model 
And next, edit this class to look like:

```ruby
class Selection < ActiveRecord::Base
  selectable
end
```

Selections table can contain one or many lists, it uses the acts_as_tree so each root is a new list
meta example: (see below for example YAML file).

* priority
 - high
 - medium
 - low
* user_role
 - admin
 - owner
 - user

From the rails console it is be possible to access these items directly using dynamic lookups:

```ruby
Selection.priority => returns the parent row
Selection.user_role
```

Dynamic lookups support pluralization and will then return the children:

```ruby
Selection.priorities  -> [high,med,low] records
```

#### Form Helper

if we had a controller for Ticket model with fields of:

* name
* priority_id

within the _form.html.erb just use the selections helper method:

```ruby
<%= form_for(@ticket) do |f| %>

  <div>
    <%= f.label :name %><br />
    <%= f.text_field :name %>
  </div>

  <div>
    <%= f.label :priority %><br />
    <%= f.selections :priority %>
  </div>

  <div>
    <%= f.submit %>
  </div>
<% end %>
```

### Form Helper Options

```ruby
  f.selections :fieldname, options = {}, html_options = {}
```

The selections method excepts all the standard Ruby on Rails form helper options and html formatting - http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-select

#### System Code Override

If you have a selection named differently to the foreign key eg. the foreign key is variety_id, you can use a system_code option.

```ruby
<%= f.selections :variety, :system_code => :category %>
```

### Scoped System Code

If you have naming conflicts/duplicates system_codes (parent codes) eg. category_id field for user and ticket models

Selections Data example

* user_category
 - civilian
 - programmer
* ticket_category
 - high
 - low

The fieldname within the user and ticket form can both be :category as the form helper will search for
either 'category' or 'user_category' in the selections model.

users/_form.html.erb

```ruby
  <%= f.selections :category %>
```

This will automatically look up the 'user_category' selections if 'category' does not exist

## Related Models

Next, you need to tell models that have a selectable association. These are `belongs_to` associations
which pull their values from the selections model. Assuming you have a `Ticket` model with a foreign key of priority_id,
you can set this up like so:

```ruby
class Ticket < ActiveRecord::Base

  belongs_to_selection :priority

end
```

From an instance of a ticket within a view the belongs_to selection will return string (to_s).
eg: show.html.erb

```ruby
<p>
  <b>Priority:</b>
  <%= @ticket.priority %>  # Instead of this @ticket.priority.try(:name)
</p>
```

## Automatic Features
#### Include Blank

In a new form the selections list will have blank top row unless a default item is set in the selections eg. Medium Priority, then there
will be no blank row and the default item will be selected.

When editing a form, by default the blank row will not be displayed. Use the options :include_blank => true option to overrided.

#### Archived Item

On a new form items that are archived in Selections will not appear in the selection list. When editing an existing record the list will only
contain un-archived items, unless the item selected is archived it will then appear.

eg. A ticket has a priority set to high, later the high selection list item was archived, when we edit the ticket record again the item will show in the list even though it is archived.

#### Order

The list will by default display in alphanumeric order, unless you add position values and then it will display in the values.

#### system_code generation
On creation of a new item in Selections the system_code field will be automatically generated based on parent and item name if left blank.

## Configuration

If you use a class name other than `Selection` as your selection model, you must
tell selections so by adding the following to a new file, `config/initializers/selections.rb`:

```ruby
Selections.model { YourSelectionModel }
```

# TODO

* Add model generators
* Add selections management scaffold/generator
* Add Radio Button support

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## example selections.yml to import into DB

```yaml
priority:
  name: Priorities
  parent:
  position_value:
  system_code: priority
  is_system: false
priority_high:
  name: High
  parent: priority
  position_value: 30
  system_code: $LABEL
  is_system:
priority_medium:
  name: Medium
  parent: priority
  position_value: 20
  system_code: $LABEL
  is_system:
priority_low:
  name: Low
  parent: priority
  position_value: 10
  system_code: $LABEL
  is_system:
  is_default: true


contact_category:
  name: Contact Categories
  parent:
  position_value:
  system_code: contact_category
  is_system: false
contact_category_junior:
  name: Junior Tech
  parent: contact_category
  position_value:
  system_code: $LABEL
  is_system:
contact_category_senior:
  name: Senior Tech
  parent: contact_category
  position_value:
  system_code: $LABEL
  is_system:
contact_category_manager:
  name: Manager
  parent: contact_category
  position_value:
  system_code: $LABEL
  is_system:
  archived_at: <%= Time.now %>
contact_category_ceo:
  name: CEO
  parent: contact_category
  position_value:
  system_code: $LABEL
  is_system:
```
