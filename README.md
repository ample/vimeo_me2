# VimeoMe2

[![Travis badge](https://travis-ci.org/bo-oz/vimeo_me2.svg?branch=master)](https://travis-ci.org/bo-oz/vimeo_me2)

A very basic wrapper for the Vimeo API. OAuth2 is not included in the code. You can easily write your own OAuth2 workflow with a gem like [OAuth2](https://github.com/intridea/oauth2). All you need is a Vimeo access token and easily make calls to their API through this gem.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'vimeo_me2', :git => "https://github.com/bo-oz/vimeo_me2.git"
```

And then execute:

    $ bundle


## Usage

This gem consists of two classes that can access respectively a user or a video object from Vimeo. Using the gem starts with obtaining an access token from [vimeo.com](https://developer.vimeo.com). When calling the classes, put in the access token and access the various methods.

### Making any call to Vimeo
Use utility methods to make any call against Vimeo. You can see the full list of eindpoints in the Vimeo documentation on [vimeo.com](https://developer.vimeo.com).

```ruby
# Set-up the basis for making calls to Vimeo
vimeo = VimeoMe2::VimeoObject.new('12345hjhsjdshasd')

# Make any get request, by providing only the API endpoint
vimeo.get('/me')

# Make any post request, including a body, don't forget
# to set the expected repsonse code
# You can also set additional header
body = "whatever"
vimeo.post('/videos/12344', body:body, headers:{'Content-Type': 'video/mp4'}, code:201)

# Or delete items
vimeo.delete('/videos/12344', code:204)

```

### Accessing a User
Take a look at the files under /lib/vimeo_me2/user/ for available methods.

```ruby
# Access your own user object
vimeo_user = VimeoMe2::User.new('12345hjhsjdshasd')

# Access someones user object
vimeo_user = VimeoMe2::User.new('12345hjhsjdshasd','username')
```

#### Like
Get a list of the likes, like and unlike video's.

```ruby
# Fetch for the user
vimeo_user.view_all_likes

# Like a specific video
vimeo_user.like_video 1234455

# Check if a video is liked
vimeo_user.check_if_liked 1234455 #returns true

# Unlike the video
vimeo_user.unlike_video 1234455

# Check if a video is unliked
vimeo_user.check_if_liked 1234455 #returns false

```
### Uploading a video
At this moment uploading a video only works with a ActionDispatch::Http::UploadedFile object.

```ruby
# Set-up a model like this for instance:

class Videofile < ActiveRecord::Base
  attr_accessor :video_file

  before_create :upload_to_vimeo

  def upload_to_vimeo
    # connect to Vimeo as your own user, this requires upload scope
    # in your OAuth2 token
    vimeo_client = VimeoMe2::User.new('12345hjhsjdshasd')
    # upload the video by passing the ActionDispatch::Http::UploadedFile
    # to the upload_video() method. The data_url in this model, stores
    # the location of the uploaded video on Vimeo.
    self.data_url = vimeo_client.upload_video(self.video_file)
    return true
  rescue VimeoMe2::RequestFailed => e
    errors.add(:video_file, e.message)
    return false
  end

end

```

### Accessing a video
Take a look at the files under /lib/vimeo_me2/video/ for available methods.

```ruby
# Get access to a video (both options are valid)
vimeo_video = VimeoMe2::Video.new('12345hjhsjdshasd','196277011')
vimeo_video = VimeoMe2::Video.new('12345hjhsjdshasd','/videos/196277011')

# Or with an uploaded video like with the model above
#in the previous code fragment
videofile = Videofile.last
vimeo_video = VimeoMe2::Video.new('12345hjhsjdshasd',videofile.data_url)

# Get comments on the video
vimeo_video.comments

# Get the name of the video
vimeo_video.name

# Set the name of the video
vimeo_video.name = "New name"

# Update the video
vimeo_video.update

# Delete the video (if you have access to do that)
vimeo_video.destroy
```

At this moment the gem only returns the raw JSON response received from Vimeo. I do plan on extending this to also include a player embed method. But this is still work in progress.

## TODO

* Write tests
* Include all parameters in the various API calls
* Write methods for every Vimeo API endpoint

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/bo-oz/vimeo_me2. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
