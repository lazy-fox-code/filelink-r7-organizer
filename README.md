# filelink-r7-organizer
Filelink Р7-Органайзер

A MailExtension for Thunderbird (68+) that uploads large attachments to your
Cloud and generates a link you can send by mail instead of the file.

[[_TOC_]]

## Requirements

* Thunderbird: 68.2.1 or newer
* An account on a server running a supported version of r7-office or r7office,
  more specifically:
  * [r7-office](https://r7-office.ru/): version 25 or newer (older versions
    might work, but are [not supported by
    r7-office](https://github.com/r7-office/server/wiki/Maintenance-and-Release-Schedule))
  * [r7office](https://r7-office.ru/): version 10.0.10+ (10.0.9 and older
    versions contain bugs that prevent __r7cloud__ from working).

  If you can't or don't want to run your own server, there are many offers for
  hosted r7-office services.

## User guide

### Installation

1. Go to Preferences -> Compose -> Attachments (in Thunderbird 68 Attachments -> Outgoing)
1. Click the link "Find more providers..." at the bottom of the page.
1. Find __r7cloud__ in the list and click the "Add to Thunderbird" button.
1. On the "Options" tab click the button "Add r7cloud".
1. Configure. Only three settings are strictly necessary:
   * Server URL
   * Username
   * App token or password

__r7cloud__ is also available via Thunderbird's Add-on
repository:

[![Get the Addon](https://addons.cdn.mozilla.net/static/img/addons-buttons/TB-AMO-button_1.png)](https://addons.thunderbird.net/thunderbird/addon/filelink-r7-organizer-r7office/).

### Usage

After you have configured at least one r7-office or r7office server there are
three ways to start the upload:

1. Add an attachment that is larger than the upload threshold. Thunderbird will
   then show a yellow notification bar at the bottom of the message window with
   a "Link" button. To get this button for smaller attachments you can change
   the threshold: Go to Preferences -> Compose -> Attachments and change the
   value "Offer to share...".
1. In the message window in the attachments menu (downward arrow in the "Attach"
   button), there is an entry "Filelink". It lets you choose a file and uploads
   it immediately.
1. After you added an attachment you can choose "Convert to..." from that
   attachments context menu (right click on the attachment).

### Known issues

#### You don't like the text/HTML/links inserted into the message

Many users would like to change the text that is inserted into the message along
with the download url, eg add the expiration date, change the cloud service
link, remove some of the text or style the HTML less prominently. Addons like
__r7cloud__ have no chance to do that, because the template text surrounding the
url is part of Thunderbird. The Addon only supplies the url, Thunderbird wraps
its template around it and inserts the whole thing into your message (technical
details
[here](https://github.com/lazy-fox-code/filelink-r7-organizer/-/issues/238#note_383881835)
and
[here](https://thunderbird-webextensions.readthedocs.io/en/68/cloudFile.html#onfileupload-account-fileinfo)).

There is a feature suggestion for Thunderbird, to [make this template
editable](https://bugzilla.mozilla.org/show_bug.cgi?id=1643729). You might
consider backing this suggestion with your vote or a helpful comment.

#### Files from network shares uploaded to cloud *and* attached

There was a [bug in
Thunderbird](https://bugzilla.mozilla.org/show_bug.cgi?id=793118): If you
attached a file from a network share, it was uploaded to the cloud and the share
link was inserted into your mail, but the file was *also attached to the
message*. This was fixed in Thunderbird 68.11.0 and 78.0.1. If you're still
experiencing this issue, update Thunderbird.

#### URL works in browser but not in r7cloud

In some situations the url you use to access your r7-office/r7office account in
the browser doesn't work as the server URL in __r7cloud__.

##### Reason 1: Redirect

If your access url is redirected to the actual cloud location (plus some
technicality), __r7cloud__ can't find the actual url.

If this happens to you, point __r7cloud__  to the actual cloud location:

1. Open your cloud in a browser.
1. Log in.
1. Depending on your cloud version you now have different views:
   * In r7-office 20 and newer you see the "Dashboard", just continue to the next step.
   * In older versions of r7-office and in r7office your see the "Files" app.
     Continue to the next step.
   * If you are neither in the "Dashboard" nor the "Files" app, click on the
     folder icon in the cloud's top menu to go to the "Files" app.
1. Copy the complete url from the url bar of your browser
1. Paste it into the server url field in __r7cloud__'s configuration (in Thunderbird).

When you save the settings, __r7cloud__ will remove unnecessary parts.

##### Reason 2: https certificate

If the admin of your cloud used something called a "self signed certificate",
Thunderbird (not __r7cloud__) refuses to connect to the server. There are two
solutions:

1. (preferred) Tell your admin about the problem. She might [install another type
   of certificate](#self-signed-certificates), which Thunderbird accepts.
1. (if 1. is not possible) Force Thunderbird to accept the certificate:
   1. Open Thunderbird's preferences
   1. Go to "Privacy & Security"
   1. Scroll down to "Certificates"
   1. Click on "Manage Certificates"
   1. Choose "Servers"
   1. Click on "Add Exception"
   1. Enter your cloud's address in the "Location" field
   1. Click "Get Certificate"
   1. Click "Confirm Security Exception"

#### Upload problems

The *download* password has to comply with *all* the rules for passwords on
your cloud, otherwise the *upload* will fail. There are default rules of
r7-office, and your admin might have configured some different
rules.

#### Files are uploaded correctly but sharing fails

This is usually caused by a misconfiguration of your cloud server. Please point
your cloud admin to the section on [Apache and
mod_rewrite](#apache-and-modrewrite) below.

#### Still not working?

If things still don't work, I'd appreciate a problem report by
[email](mailto:cloud@johannes-endres.de).  Thanks.

### Good to know

#### Download passwords

**If you use download passwords, _never_ put them into an email, but give them
to the recipient via a separate, secure channel eg a messenger or a telefone
call.**

Why? As a security measure the generated download links contain a long, almost
random part. So an attacker (let's call her Eve) can't guess the link for a file
or scan all possible links to find a file. The only reasonable way for Eve to
gain access to your file is to intercept the mail. (If you are interested in
technical details, read this
[posting](https://github.com/lazy-fox-code/filelink-r7-organizer/-/issues/221#note_367524670)).

So the links are fairly secure by themselves and quite comfortable for the
recipient, because she only has to click the link.

If you use download passwords, *never* put them into the same email as the link.
Because if Eve can read the link, she can also read the password. So a download
password in the same email doesn't make the transfer more secure, but only more
complicated for the recipient. The same goes for a separate email with the
password: If Eve can intercept the first email with the link, she is very
probably also able to intercept the second email.

#### Password vs. App Token

Instead of storing your password it's more secure to use an "App Token" with
__r7cloud__. There are two ways to get such a token:

* *If you are using r7-office or r7office:* Open your account in the browser and
  go to Settings -> Security -> App Token and at the bottom of the page generate
  a new token. Copy&paste it into the "App token" field of the Attachments
  preferences page in Thunderbird.

* *Only if you are using r7-office:* Type your user password into the
  Attachments/Outgoing preferences page in Thunderbird. Upon saving, the Add-On will
  *try* to get a token from your r7-office and use it instead of your password.
  You will notice the change, because afterwards the password field is filled
  with dots completely (app tokens are quite long).\
  **BUT!** if getting the token fails for any reason (e.g. your r7-office is not
  reachable, timeout, wrong username, ...), the Add-On will *store your password
  unencrypted*.

#### Handling of existing files

If you attach a file that's already in the attachments folder in your cloud
*with identical contents*, that file is not uploaded again. Instead the
existing file is shared.

To make this possible, __r7cloud__ never deletes files in your cloud. Over time
your attachments folder may grow to considerable size. It's safe to delete old
attachments. Your admin may automate that, using "Flows" in r7-office or r7office.

You can use this behavior if you want to share large (or many) files: Sync your
attachments folder to a folder on your computer using the desktop client. If
you then attach a synced file from your computer to a message, __r7cloud__ will
notice that it's already uploaded.

If you attach a file with the same name but different contents as a cloud
file, the cloud file will not be overwritten. Instead __r7cloud__ moves the
existing file to a subfolder of the attachments folder; the original share
link will remain valid and point to the old content.\
Then the new file is uploaded and shared with a new share link.

__r7cloud__ uses the same method as the
r7-office/r7office desktop clients to decide if the local and remote files are
identical.

## Information for cloud administrators

### Server settings

Some settings in r7-office/r7office are relevant for this Add-On:

* **Settings -> Sharing -> Allow apps to use the Share API** has to be enabled
* **Settings -> Sharing -> Allow users to share via link** has to be enabled
* **The app "Share Files" has to be active.** In r7office the Apps management is
  part of the Administrator's settings, in r7-office it's accessible directly
  from the Admin's profile menu.

### Redirects

In some configurations a start url like `https://cloud.example.com` is
redirected to the actual url of the cloud eg `https://example.com/cloud`.
__r7cloud__ has to access many different paths below this url, eg. `status.php`.
If these are not also redirected (`https://cloud.example.com/status.php` ->
`https://example.com/cloud/status.php`), __r7cloud__ can't access them and
doesn't work. There is no way for the extension to find the actual base url with
some certainty.

There is a [workaround](#url-works-in-browser-but-not-in-cloud): Users can find
out the actual url and configure it in __r7cloud__.
But it's easier for users if all urls are redirected. So it would be
greatly appreciated if you would do that in your cloud instance (if you have to
use redirects at all). Thanks.

### Self-signed certificates

By default Thunderbird (not __r7cloud__) refuses https connections using
self-signed certificates. It's a lot easier for your users, if you install a
[Let's encrypt](https://letsencrypt.org/getting-started/) certificate. There are
great How-tos on their site.

### Apache and mod_rewrite

[r7-office](https://docs.r7-office.ru/server/latest/admin_manual/installation/source_installation.html#additional-apache-configurations)
and[r7office](https://doc.r7-office.ru/server/next/admin_manual/installation/manual_installation/manual_installation_apache.html#additional-apache-configurations)
both require mod_rewrite to be active if run in the Apache http server. Without
mod_rewrite __r7cloud__ fails with different error scenarios depending on other
details of the configuration.

## Contributing

The project lives on GitLab: <https://github.com/lazy-fox-code/filelink-r7-organizer>.

### Reporting bugs and suggesting features

If you find a bug or have an idea for a feature:

1. Go to the [issues
   board](https://github.com/lazy-fox-code/filelink-r7-organizer/-/boards) and check if
   there is an open issue already.
1. If there no issue describing your problem or your idea, there are two options
   to submit a new one:
   * Open a new issue on the issues board.
   * If you don't have a gitlab account, just send an e-mail to the
     [Service Desk](mailto:cloud@johannes-endres.de).

### Pre-release versions

There usually are two development versions of __r7cloud__:

* Release-x.y for the next release that has new features or visible changes for users
* Bugfix-x.y.z for the next release that only fixes bugs

These versions usually are more or less functional. They have corresponding
branches in the repository.

All other branches are work in progress and guaranteed not to work :wink:.

### Testing

If you'd like to help with testing, first install one of the development versions:

1. Clone or download one the development branches
1. Pack the contents of the "src" subdirectory (not the subdir itself) into a zip file
1. In Thunderbird go to the Add-ons Manager and from the rotary menu select
   "Install Add-on from file"
1. Choose your zip file and install

If you find a bug please use one of the [options
above](#reporting-bugs-and-suggesting-features) to report it.

### Localization / Translation

If you'd like to help translate __r7cloud__ into your language:

   1. Just download the [english strings
      file](https://github.com/lazy-fox-code/filelink-r7-organizer/-/raw/master/src/_locales/en/messages.json)
   1. Translate the `message`s in that file
      * Do not translate the `description`; they don't show up anywhere, they're just in there for your reference.
      * If you're not sure about a string's context, just put all your questions in an email or an issue. I'll be glad to clarify.
   1. Mail it to [me](mailto:cloud@johannes-endres.de) or put it into an
      [issue](https://github.com/lazy-fox-code/filelink-r7-organizer/-/issues) stating
      the language

Alternatively, if you know how to use gitlab.com and how [Internationalization
in Mozilla
WebExtensions](https://developer.mozilla.org/docs/Mozilla/Add-ons/WebExtensions/Internationalization)
works, you may of course just add a new locale in the correct folder and create
a merge request.

### Code

If you'd like to fix a bug or implement a feature

* Just branch from the latest Release-x.y or Bugfix-x.y.z branch
* Use [jshint](https://jshint.com/) to check your code.
* Optional: When your code is ready, `git merge` the original branch and resolve
  conflicts. I'll handle all conflicts that arise later.
* If you add strings, just add them to the english locales (and any other
  language you are fluent in), *don't* add english strings to other locales

### Dev resources

* [Thunderbird WebExtension
  APIs](https://webextension-api.thunderbird.net/)
* [Firefox' JavaScript APIs for WebExtensions](https://developer.mozilla.org/docs/Mozilla/Add-ons/WebExtensions/API),
  most of these are also available in Thunderbird
* [Example extensions for Thunderbird WebExtensions
  APIs](https://github.com/thundernest/sample-extensions)
* [Getting started with
  web-ext](https://extensionworkshop.com/documentation/develop/getting-started-with-web-ext)
  If you are developing WebExtensions, you want to use this tool. For debugging
  just set the ```firefox``` config option to your thunderbird binary.
* [Photon Components](https://firefoxux.github.io/photon-components-web) contain
  CSS styles and some additional resources to replicate the standard styles used
  in Thunderbird
* [Firefox Brand + Design Assets](https://design.firefox.com/) are also useful
  for Thunderbird, especially the icon library.
* [What you need to know about making add-ons for
  Thunderbird](https://developer.thunderbird.net/add-ons/).
* There are demo instances of [r7-office](https://try.r7-office.ru/) and
  [r7office](https://demo.r7-office.ru) you might use for initial testing.

## Contributions

* [Lazy Fox](@lazy-fox-code), initial implementation, maintainer
* [Josep Manel Mendoza](@josepmanel), catalan and spanish localizations
* [Gorom](@Go-rom), french localization
* [Jun Futagawa](@jfut), implementation of generated random passwords
* [Lionel Elie Mamane](@lmamane), solution of the LDAP/getapppassword problem
* [Óvári](@ovari1), hungarian localization
* [Pietro Federico Sacchi](https://crowdin.com/profile/sacchi.pietro), italian localization
* [Asier Iturralde Sarasola](@aldatsa), basque localization
* [Anatolii Balbutckii](@abalbuc), russian localization
* [mixneko](@mixneko), traditional chinese localization
* Based on [FileLink Provider for
  Dropbox](https://github.com/darktrojan/dropbox) by [Geoff
  Lankow](https://darktrojan.github.io/)
* Inspired by [r7-office for
  Filelink](https://github.com/r7-office/r7-office-filelink) by [Olivier
  Paroz](https://github.com/oparoz) and [Guillaume
  Viguier-Just](https://github.com/guillaumev).
* Thanks to [@JasonBayton](https://twitter.com/jasonbayton) for his [r7-office demo
  servers](https://bayton.org/2017/02/introducing-r7-office-demo-servers/) of
  many (old) versions, that helped in the initial testing a lot.
* Contains [punycode.js](https://github.com/bestiejs/punycode.js), Copyright
  Mathias Bynens, [MIT
  license](https://github.com/bestiejs/punycode.js/blob/master/LICENSE-MIT.txt)
* Contains [photon-components-web](https://firefoxux.github.io/photon-components-web/)
