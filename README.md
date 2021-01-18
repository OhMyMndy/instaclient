<h1>instaclient</h1>

[logo]: https://github.com/davidwickerhf/instaclient/blob/main/images/instaclient.png


<a href="https://pypi.org/project/instaclient/">
<img alt="PyPi" src="https://img.shields.io/pypi/v/instaclient.svg"/>
</a>
<a href="https://pepy.tech/project/instaclient">
<img alt="Downloads" src="https://pepy.tech/badge/instaclient"/>
</a>
<a href="https://github.com/wickerdevs/instaclient/blob/master/LICENSE.txt">
<img alt="GitHub license" src="https://img.shields.io/github/license/wickerdevs/instaclient?style=plastic"/>
</a>
<a href="https://t.me/instaclient">
<img alt="Chat" src="https://img.shields.io/badge/chat-Telegram-2CA5E0?logo=Telegram"/>
</a>

</a>
<img alt="GitHub Repo Size" src="https://img.shields.io/github/repo-size/wickerdevs/instaclient"/>

**instaclient** is a Python library for accessing Instagram's features.
With this library you can create Instagram Bots with ease and simplicity. The InstaClient takes advantage of the selenium library to excecute tasks which are not allowed in the Instagram Graph API (such as sending DMs).

The only thing you need to worry about is to spread your requests throughout the day to avoid reaching Instagram spam limits.
> **Disclaimer:** Please note that this is a research project. **I am by no means responsible for any usage of this tool.** Use it on your behalf. I'm also not responsible if your accounts get banned due to the extensive use of this tool.

## Table of Contents

1. [Features](#features)
2. [Installation](#installation)
3. [Usage](#usage)
4. [Contributing](#contributing)
5. [Changelog](#changelog)
6. [Help - Community](#help-community)
7. [Credits](#credits)
8. [License](#license)
---

## Features
- Scraping
    - Scrape a user's followers (Via scrolling or with GraphQL)
    - Scrape a user's following (Via scrolling or with GraphQL)
    - Scrape a Hashtag
    - Scrape a Location
    - Scrape a Profile
    - Scrape a user's posts
    - Scrape a hashtag's posts
    - Scrape a location's posts
    - Scrape a post's info via its shortcode
- Interacting
    - Follow a user
    - Unfollow a user
    - Like a post
    - Unlike a post
    - Add a comment on a post
    - Send DMs to users (both Private & Public)
    - Check Incoming Notifications

#### TO DO:
- [x] Define Classes:
    - [x] Comment
    - [x] Post
    - [x] Location
- [x] Scrape User Posts's shorturl
- [x] Scrape Post by shorturl
- [x] Add comment to post by shorturl
- [x] Like post by shorturl
- [x] Unlike post by shorturl
- [x] Scrape Location
- [ ] Save cookies
- [ ] Share/Forward a post
- [ ] Scrape explore page
- [ ] Upload posts
- [ ] Scrape feed
- [ ] Interact with posts on feed 
- [ ] View feed stories
- [ ] View user stories
- [ ] Save/Unsave posts
 
## Installation

Use the package manager [pip](https://pip.pypa.io/en/stable/) to install instaclient.

```bash
pip install instaclient
```
To update the package:
```bash
pip install -U instaclient
```

## Usage
#### INSTALL A DRIVER (LocalHost)
If you are running your code on a localhost, then you'll need to install a chromedriver from [here](https://chromedriver.chromium.org/downloads). Install and extract the chromedriver.exe file and save it in your project folder. Make sure to install the version that matches your Chrome version.
To check your chrome version, type ```chrome://version/``` in the chrome address bar.

#### SET ENVIROMENT VARIABLES (Web Server)
If you are running your code on a web server (Like Heroku), you should set the following enviroment variable:
- ```CHROMEDRIVER_PATH = /app/.chromedriver/bin/chromedriver```
- ```GOOGLE_CHROME_BIN = /app/.apt/usr/bin/google-chrome```

#### CREATE THE CLIENT
```python
from instaclient import InstaClient
from instaclient.errors import *

# Create a instaclient object. Place as driver_path argument the path that leads to where you saved the chromedriver.exe file
client = InstaClient(driver_path='<projectfolder>/chromedriver.exe')
```
#### LOGIN INTO INSTAGRAM
```python
from instaclient.errors import *

try:
    client.login(username=username, password=password) # Go through Login Procedure
except VerificationCodeNecessary:
    # This error is raised if the user has 2FA turned on.
    code = input('Enter the 2FA security code generated by your Authenticator App or sent to you by SMS')
    client.input_verification_code(code)
except SuspisciousLoginAttemptError as error:
    # This error is reaised by Instagram
    if error.mode == SuspisciousLoginAttemptError.EMAIL:
        code = input('Enter the security code that was sent to you via email: ')
    else:
        code = input('Enter the security code that was sent to you via SMS: ')
    client.input_security_code(code)
```
#### INSTAGRAM OBJECTS
Many client methods return objects defined in the [instagram](https://github.com/wickerdevs/instaclient/tree/main/instaclient/instagram) package of this library.
Such objects are reppresentations of the data returned by instagram, but they are not continuesly synced with Instagram, hence the data may not always be updated. To sync the local object to instagram, most objects have `.refresh()` method, as seen in the example below:
```python
profile = client.get_profile('<username>')
# Other code here
profile.refresh() # syncing with instagram
```

#### SEND A DIRECT MESSAGE
```python
result = client.send_dm('<username>', '<Message to send>') # send a DM to a user
```
> Make sure to distrubute your client.send_dm() requests over a period of time to avoid reaching Instagram's spam limits.
#### GET A USER'S FOLLOWERS
```python
followers = client.get_followers(user='<username>') # replace with the target username
```
> The client.scrape_followers() method can take a lot of time depending on the amount of followers you want to scrape.

This method might be updated in the near future to cache scraped data in a SQLite database or to scrape the followers in a separate thread with a queue.
#### SCRAPE NOTIFICATIONS
```python
notifications = client.get_notifications(count=10)
```
> This returns a Notification object, which contains information about the type of notification and the user who caused it.
#### SCRAPE A PROFILE
```python
profile = client.get_profile('<username>')
```
> This returns a Profile object, from which you can get posts and all sorts of information.
#### SCRAPE A PROFILE's POSTS
```python
posts = client.get_user_posts('<username>', count=10)
# or:
profile = client.get_profile('<username>')
posts = profile.get_posts(30)
```
> This returns a list of Post objects.
#### SCRAPE A HASHTAG
```python
hashtag = client.get_hashtag(tag='<tag>')
# This returns a Hashtag object, from which you can get the posts data. 
posts = hashtag.get_posts(count=25)
```
#### SCRAPE A LOCATION
##### BY ID & SLUG
Every Instagram Location is defined by a slug and an ID.
You can find these two pieces of information in the URL of the location page on Instagram:
```https://www.instagram.com/explore/locations/270531492/italy/```
from this example, the ID would be `270531492` and the slug would be `italy`
```python
location = client.get_location(id='270531492', slug='italy')
posts = location.get_posts(count=25)
```

##### BY SEARCH RESULT
If you don't have access to either the ID or the slug of a location, you can still scrape such location by finding it on the search page.
```python
result = client.get_search_result('italy')
location = result.get('locations')[0]
```

#### ADD A COMMENT
```python
client.comment_post('<post_shortcode>', text='Nice post!')
# or:
post = client.get_user_posts('<username>', count=1)[0]
post.add_comment('Nice post!')
```
#### LIKE A POST
```python
client.like_post('<post_shortcode>')
# or:
post = client.get_user_posts('<username>', count=1)[0]
post.like()
```
#### FOLLOW A USER
```python
client.follow_user('<username>')
# or:
profile = client.get_profile('<username>')
profile.follow()
```
#### UNFOLLOW A USER
```python
client.unfollow_user('<username>')
# or:
profile = client.get_profile('<username>')
profile.unfollow()
```
## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
Please make sure to update [tests](https://github.com/wickerdevs/instaclient/tree/master/tests) as appropriate.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).
Hence, when pushing commits, it is encouraged to use the described formatting and use the following keywords:
- ```Added``` for new features.
- ```Changed``` for changes in existing functionality.
- ```Deprecated``` for soon-to-be removed features.
- ```Removed``` for now removed features.
- ```Fixed``` for any bug fixes.
- ```Security``` in case of vulnerabilities.

## Changelog
You can find this repository's changelog here: [CHANGELOG](https://github.com/wickerdevs/instaclient/blob/master/CHANGELOG.md)

## Help Community
You can join this [Telegram Group](https://t.me/instaclient) to ask questions about the instabot's functionalities or to contribute to the package!

## Credits
[AUTHORS](https://github.com/wickerdevs/instaclient/blob/master/AUTHORS.rst)

## License
[MIT](https://choosealicense.com/licenses/mit/)


