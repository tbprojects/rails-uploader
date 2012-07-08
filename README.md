# HTML5 File uploader for rails

This gem use https://github.com/blueimp/jQuery-File-Upload for upload files.

![Uploader in use](http://img39.imageshack.us/img39/2206/railsuploader.png)

## Install

In Gemfile:

  gem "rails-uploader"

In routes:  

``` ruby
  mount Uploader::Engine => '/uploader'
```

## Usage

Architecture to store uploaded files (cancan integration):

``` ruby
  class Asset < ActiveRecord::Base
    include Uploader::Asset
    
    def uploader_create(params, request = nil)
      ability = Ability.new(request.env['warden'].user)
      
      if ability.can? :create, self
        self.user = request.env['warden'].user
        super
      else
        errors.add(:id, :access_denied)
      end
    end
    
    def uploader_destroy(params, request = nil)
      ability = Ability.new(request.env['warden'].user)
      
      if ability.can? :delete, self
        super
      else
        errors.add(:id, :access_denied)
      end
    end
  end
  
  class Picture < Asset
    mount_uploader :data, PictureUploader
    
    validates_integrity_of :data
    validates_filesize_of :data, :maximum => 2.megabytes.to_i
  end
```

For example user has one picture:

``` ruby
  class User < ActiveRecord::Base
    has_one :picture, :as => :assetable, :dependent => :destroy
    
    fileuploads :picture
  end
```

Find asset by foreign key or guid:

``` ruby
  @user.fileupload_asset(:picture)
```

### Include assets

Javascripts:

  //= require uploader/application

Stylesheets:

  *= require uploader/application  
  
### Views

``` ruby
  <%= uploader_field_tag :article, :photo %>
```

or FormBuilder:

``` ruby
  <%= form.uploader_field :photo %>
```

### Formtastic

``` ruby
  <%= f.input :picture, :as => :uploader %>
```

### SimpleForm

``` ruby
  <%= f.input :picture, :as => :uploader %>
```

Copyright (c) 2012 Aimbulance, released under the MIT license