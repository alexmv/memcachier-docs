<a href="https://github.com/memcachier/docs"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://github-camo.global.ssl.fastly.net/e7bbb0521b397edbd5fe43e7f760759336b5e05f/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f677265656e5f3030373230302e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_right_green_007200.png"></a>

# Getting Started with MemCachier

MemCachier is an implementation of the
[Memcache](http://memcached.org) in-memory key/value store used for
caching data. Memcache a key technology in modern web applications for
scaling and reducing server loads. MemCachier manages and
scales clusters of memcache servers so you can focus on your app. Tell
us how much memory you need and get started for free instantly. Add
capacity later as you need it.

Below is our user guide for using MemCachier, you can also find answers
in our <a href="/faq">FAQ</a>.

<h2 id="toc">Table of Contents</h2>

1. [Supported Protocols: ASCII & Binary](#protocols)
2. [Ruby](#ruby)
3. [Rails 3 & 4](#rails3)
4. [Rails 2](#rails2)
5. [Ruby Puma Webserver](#ruby-puma-webserver)
6. [Rack::Cache](#rack)
7. [Python](#python)
8. [Django](#django)
9. [PHP](#php)
10. [WordPress](#wordpress)
11. [CakePHP](#cakephp)
12. [Symfony2](#symfony2)
13. [Laravel](#laravel)
14. [Node.js](#node.js)
15. [Java](#java)
16. [Supported client libraries](#clients)
17. [Example applications](#sample-apps)
18. [Local usage](#local)
19. [MemCachier analytics](#analytics)
20. [Advanced analytics](#advanced-analytics)
21. [Analytics API](#analytics-api)
22. [New Relic integration](#newrelic)
23. [Credentials](#credentials)
24. [Disabled caches](#disabled-caches)
25. [Encrypted Connections (TLS)](#tls)
26. [Changing plans](#upgrading)
27. [Usage Documentation](#using)
28. [Key-Value size limit](#1mb-limit)
29. [Errors connecting to localhost](#localhost-errors)
30. [Getting support](#support)


<h2 id="protocols">Supported Protocols: ASCII &amp; Binary</h2>

MemCachier supports both the memcache binary and ASCII protocols. Our preferred
method is to use the binary protocol as it natively supports user
authentication and improves the performance slightly over the ASCII protocol.
All our documentation is written for using the binary protocol.

The ASCII protocol does not natively support user authentication. For
MemCachier, where all connections require authentication, we extend the ASCII
protocol in a very simple way. Once the connection is established, you should
send a `set` command with your username as the key and your password as the
value. For example:

```text
$ telnet 35865.1e4cfd.us-east-3.ec2.prod.memcachier.com 11211
>
> set 15F38e 0 0 32
> 52353F9F1C4017CC16FD348B982ED47D
> STORED
```

You'll need to do this quickly! As we timeout connections that don't
authenticate within a few seconds. Once authenticated, the full ASCII protocol
is available to you.


<h2 id="ruby">Ruby</h2>

Start by adding the [dalli](https://github.com/mperham/dalli) gem to your
Gemfile.

```ruby
gem 'dalli'
```

Then bundle install:

```text
$ bundle install
```

`Dalli` is a Ruby memcache client. Once it is installed you can start writing
code. The following is a basic example showing get and set.

```ruby
cache = Dalli::Client.new(<MEMCACHIER_SERVERS>.split(","),
                    {:username => <MEMCACHIER_USERNAME>,
                     :password => <MEMCACHIER_PASSWORD>
                     :failover => true,
                     :socket_timeout => 1.5,
                     :socket_failure_delay => 0.2,
                     :down_retry_delay => 60
                    })
```

The values for `<MEMCACHIER_SERVERS>`, `<MEMCACHIER_USERNAME>`, and
`<MEMCACHIER_PASSWORD>` are listed on your [cache overview
page](https://www.memcachier.com/caches).

From here you can use the following code examples to use the cache in your Ruby
app:

```ruby
cache.set("foo", "bar")
puts cache.get("foo")
```

You can also get an insight into your cache usage (number of keys stored and
bytes) with the `stats` command:

```ruby
cache.stats
=> {"memcachier.example.net:11211" => {"cur_items" => "49982", "bytes" => "89982234"} }
```

We’ve built a small Ruby example using Sinatra here: [MemCachier Sinatra Sample
App](https://github.com/memcachier/examples-sinatra).


<h2 id="rails3">Rails 3 & 4</h2>

Start by adding the [dalli](https://github.com/mperham/dalli) gem to your Gemfile.

```ruby
gem 'dalli'
```

Then bundle install:

```text
$ bundle install
```

`Dalli` is a Ruby memcache client. Once it is installed you’ll want to
configure the Rails cache_store appropriately. Modify
`config/environments/production.rb` with the following:

```ruby
config.cache_store = :dalli_store, <MEMCACHIER_SERVERS>.split(","),
                    {:username => <MEMCACHIER_USERNAME>,
                     :password => <MEMCACHIER_PASSWORD>
                     :failover => true,
                     :socket_timeout => 1.5,
                     :socket_failure_delay => 0.2,
                     :down_retry_delay => 60
                    }
```

The values for `<MEMCACHIER_SERVERS>`, `<MEMCACHIER_USERNAME>`, and
`<MEMCACHIER_PASSWORD>` are listed on your [cache overview
page](https://www.memcachier.com/caches).

<p class="alert alert-info">
In your development environment, Rails.cache defaults to a simple
in-memory store and so it doesn't require a running memcached.
</p>

From here you can use the following code examples to use the cache in your
Rails app:

```ruby
Rails.cache.write("foo", "bar")
puts Rails.cache.read("foo")
```

We’ve built a small Rails example here: [MemCachier Rails sample
app](https://github.com/memcachier/examples-rails).


<h2 id="rails2">Rails 2</h2>

Start by adding the [dalli](https://github.com/mperham/dalli) gem to your
Gemfile. You will need to use dalli **v1.0.5** as later versions of Dalli don't
support Rails 2.

```ruby
gem 'dalli', '~>1.0.5'
```

Then bundle install:

```text
$ bundle install
```

`Dalli` is a Ruby memcache client. Once it is installed you’ll want to
configure the Rails cache_store appropriately. Modify
`config/environments/production.rb` with the following:

```ruby
require 'active_support/cache/dalli_store23'
config.cache_store = :dalli_store, <MEMCACHIER_SERVERS>.split(","),
                    {:username => <MEMCACHIER_USERNAME>,
                     :password => <MEMCACHIER_PASSWORD>
                     :failover => true,
                     :socket_timeout => 1.5,
                     :socket_failure_delay => 0.2,
                     :down_retry_delay => 60
                    }
```

The values for `<MEMCACHIER_SERVERS>`, `<MEMCACHIER_USERNAME>`, and
`<MEMCACHIER_PASSWORD>` are listed on your [cache overview
page](https://www.memcachier.com/caches).

<p class="alert alert-info">
In your development environment, Rails.cache defaults to a simple
in-memory store and so it doesn't require a running memcached.
</p>

Also modify `config/environment.rb` to contain:

```ruby
config.gem 'dalli'
```

From here you can use the following code examples to use the cache in your
Rails app:

```ruby
Rails.cache.write("foo", "bar")
puts Rails.cache.read("foo")
```

We’ve built a small Rails (3 & 4) example here: [MemCachier Rails sample
app](https://github.com/memcachier/examples-rails).


<h2 id="ruby-puma-webserver">Ruby Puma Webserver</h2>

If you are using the [Puma](http://puma.io/) webserver for your Ruby app (Rails
or otherwise), then you should take some additional steps due to the
multi-threaded runtime being used. This applies to all threaded webservers for
Ruby, not just Puma.

First, please refer to the documentation on [Rails](#rails-3-and-4) or
[Ruby](#ruby) above appropriately, and then take these additional steps.

Dalli by default uses a single connection to each server. This works fine
normally, but can be come a bottleneck in a multi-threaded environment and
limit performance. In this case, Dalli support connection pooling, where
multiple connections are created to MemCachier's servers. To use this, start by
adding the `connection_pool` gem to your Gemfile:

```ruby
gem 'connection_pool'
```

Next, you'll need to set the `:pool_size` configuration option when setting up
Dalli. For example, in Rails 3 & 4 your configuration would become:

```ruby
config.cache_store = :dalli_store,
                    (ENV["MEMCACHIER_SERVERS"] || "").split(","),
                    {:username => ENV["MEMCACHIER_USERNAME"],
                     :password => ENV["MEMCACHIER_PASSWORD"],
                     :failover => true,
                     :socket_timeout => 1.5,
                     :socket_failure_delay => 0.2,
                     :down_retry_delay => 60,
                     :pool_size => 5
                    }
```

Where the number 5 should be chosen according to how many threads you will be
running and the available concurrency on the machines running your webserver.


<h2 id="rack">Rails Rack::Cache</h2>

Rails can use a middle-ware component of the Rack web server architecture
called Rack::Cache. This provides caching of static assets in Rails and is a
simple alternative to use a full CDN.

Please see [this
article](https://devcenter.heroku.com/articles/rack-cache-memcached-rails31#configure-rails-cache-store)
for information.


<h2 id="python">Python</h2>

<p class="alert alert-info">
We support the <code>pylibmc</code> memcache client as it has great performance
and Python 3 support. However, it can sometimes be difficult to install locally
as it relies on the C <code>libmemcached</code> library. If you prefer, you can
try a pure python client, <a
href="https://github.com/jaysonsantos/python-binary-memcached">python-binary-memcached</a>.
</p>

Here we explain how you setup and install MemCachier with Python.

MemCachier has been tested with the `pylibmc` memcache client. This
client relies on the C libmemcached library. This should be fairly
straight-forward to install with your package manager on Linux or
Windows. We also have a
[blog post](http://blog.memcachier.com/2014/11/05/ubuntu-libmemcached-and-sasl-support/)
for Ubuntu users on how to do this. Once it's installed, then install
`pylibmc`:

```term
$ pip install pylibmc
```

Be sure to update your `requirements.txt` file with these new requirements
(note that your versions may differ than what’s below):

```text
pylibmc==1.5.1
```

<p class="alert alert-info">
<b>Heroku Users:</b> The above <code>pylibmc</code> requirements must be added
directly to your <code>requirements.txt</code> file. They shouldn't be placed
in an included pip requirement file. The Heroku Python buildpack checks the
<code>requirements.txt</code> file and only that file for the presence of
<code>pylibmc</code> to trigger bootstrapping <code>libmemcached</code>, which
is prerequisite for installing <code>pylibmc</code>.
</p>


Next, configure your settings.py file the following way:

```python
import pylibmc

servers = os.environ.get('MEMCACHIER_SERVERS', '').split(',')
user = os.environ.get('MEMCACHIER_USERNAME', '')
pass = os.environ.get('MEMCACHIER_PASSWORD', '')

mc = pylibmc.Client(servers, binary=True,
                    username=user, password=pass,
                    behaviors={
                      # Faster IO
                      "tcp_nodelay": True,

                      # Keep connection alive
                      'tcp_keepalive': True,

                      # Timeout for set/get requests
                      'connect_timeout': 2000, # ms
                      'send_timeout': 750 * 1000, # us
                      'receive_timeout': 750 * 1000, # us
                      '_poll_timeout': 2000, # ms

                      # Better failover
                      'ketama': True,
                      'remove_failed': 1,
                      'retry_timeout': 2,
                      'dead_timeout': 30,
                    })
```

After this, you can start writing cache code in your Python app:

```python
mc.set("foo", "bar")
print mc.get("foo")
```

<p class="alert alert-info">
A confusing error message you may get from <code>pylibmc</code> is
<b>MemcachedError: error 37 from memcached_set: SYSTEM ERROR (Resource
temporarily unavailable)</b>. This indicates that you are trying to store a
value larger than 1MB. MemCachier has a hard limit of 1MB for the size of
key-value pairs. To work around this, either consider sharding the data or
using a different technology. The benefit of an in-memory key-value store
diminishes at 1MB and higher.
</p>


<h2 id="django">Django</h2>

<p class="alert alert-info">
We support the <code>pylibmc</code> memcache client as it has great performance
and Python 3 support. However, it can sometimes be difficult to install locally
as it relies on the C <code>libmemcached</code> library. If you prefer, you can
try a pure python client, <a
href="https://github.com/jaysonsantos/python-binary-memcached">python-binary-memcached</a>.
You'll also need the <a
href="https://github.com/jaysonsantos/django-bmemcached">django-bmemcached</a>
package.
</p>

MemCachier has been tested with the `pylibmc` memcache client. This is a great
client, fully-featured, high-performance and Python 2 & 3 support. Sadly, the
Django integration of `pylibmc` and other memcache clients doesn't work
out-of-the-box with MemCachier as they don't expose the authentication
mechanism (SASL). This is easily solved by using the `django-pylibmc` package.

The `pylibmc` client relies on the C `libmemcached` library. This should be
fairly straight-forward to install with your package manager on Linux or
Windows. We also have a [blog
post](http://blog.memcachier.com/2014/11/05/ubuntu-libmemcached-and-sasl-support/)
for Ubuntu users on how to do this.

Once `libmemcached` is installed, then install `pylibmc` and `django-pylibmc`:

```text
$ pip install pylibmc django-pylibmc
```

Be sure to update your `requirements.txt` file with these new requirements
(note that your versions may differ than what’s below):

```text
pylibmc==1.5.1
django-pylibmc==0.6.1
```

<p class="alert alert-info">
<b>Heroku Users:</b> The above <code>pylibmc</code> requirements must be added
directly to your <code>requirements.txt</code> file. They shouldn't be placed
in an included pip requirement file. The Heroku Python buildpack checks the
<code>requirements.txt</code> file and only that file for the presence of
<code>pylibmc</code> to trigger bootstrapping <code>libmemcached</code>, which
is prerequisite for installing <code>pylibmc</code>.
</p>

Next, configure your settings.py file the following way:

```python
os.environ['MEMCACHE_SERVERS'] = os.environ.get('MEMCACHIER_SERVERS', '').replace(',', ';')
os.environ['MEMCACHE_USERNAME'] = os.environ.get('MEMCACHIER_USERNAME', '')
os.environ['MEMCACHE_PASSWORD'] = os.environ.get('MEMCACHIER_PASSWORD', '')

CACHES = {
    'default': {
        # Use pylibmc
        'BACKEND': 'django_pylibmc.memcached.PyLibMCCache',

        # Use binary memcache protocol (needed for authentication)
        'BINARY': True,

        # TIMEOUT is not the connection timeout! It's the default expiration
        # timeout that should be applied to keys! Setting it to `None`
        # disables expiration.
        'TIMEOUT': None,

        'OPTIONS': {
            # Enable faster IO
            'tcp_nodelay': True,

            # Keep connection alive
            'tcp_keepalive': True,

            # Timeout settings
            'connect_timeout': 2000, # ms
            'send_timeout': 750 * 1000, # us
            'receive_timeout': 750 * 1000, # us
            '_poll_timeout': 2000, # ms

            # Better failover
            'ketama': True,
            'remove_failed': 1,
            'retry_timeout': 2,
            'dead_timeout': 30,
        }
    }
}
```

The values for `<MEMCACHIER_SERVERS>`, `<MEMCACHIER_USERNAME>`, and
`<MEMCACHIER_PASSWORD>` are listed on your [cache overview
page](https://www.memcachier.com/caches). Note that Django expects
<MEMCACHIER_SERVERS> to be semicolon-delimited (while we provide it
comma-eliminated).

Finally, we also *strongly* recommend that you place the following
code in your `wsgi.py` file to correct a serious performance bug
([#11331](https://code.djangoproject.com/ticket/11331)) with Django
and memcached. The fix enables persistent connections under Django,
which by default uses a new connection for each request:

```python
# Fix django closing connection to MemCachier after every request (#11331)
from django.core.cache.backends.memcached import BaseMemcachedCache
BaseMemcachedCache.close = lambda self, **kwargs: None
```

After this, you can start writing cache code in your Django app:

```python
from django.core.cache import cache
cache.set("foo", "bar")
print cache.get("foo")
```

We’ve built a small Django example here: [MemCachier Django sample
app](https://github.com/memcachier/examples-django).

You may also be interested in the
[django-heroku-memcacheify](http://github.com/rdegges/django-heroku-memcacheify)
pip, which fully configures MemCachier with one line of code for any Django app
the pip supports.

<p class="alert alert-info">
A confusing error message you may get from <code>pylibmc</code> is
<b>MemcachedError: error 37 from memcached_set: SYSTEM ERROR (Resource
temporarily unavailable)</b>. This indicates that you are trying to store a
value larger than 1MB. MemCachier has a hard limit of 1MB for the size of
key-value pairs. To work around this, either consider sharding the data or
using a different technology. The benefit of an in-memory key-value store
diminishes at 1MB and higher.
</p>

<h2 id="php">PHP</h2>

We recommended you use the [PHP Memcached
client](http://www.php.net/manual/en/book.memcached.php) to connect with
MemCachier. It supports the full protocol and has great performance. We also
recommend that you use the [composer dependency
manager](https://getcomposer.org/) for PHP, although that is up to you.

It can be difficult to get the memcached client to work as it requires that you
build it against a version of libmemcached (a C library that the PHP client
relies upon) that support SASL authentication, which often isn't enabled by
default. If you have trouble, please open a [support
ticket](http://support.memcachier.com/) with us. Alternatively, you could use a
[pure PHP client](#php-memcachesasl) that MemCachier supports, instructions on
how are [here](#php-memcachesasl).

First, if using composer, you'll need to modify your `composer.json` file to
include the module:

```js
{
    "require": {
        "php": ">=5.3.2",
        "ext-memcached": "*"
    }
}
```

Then, you can connect to MemCachier using the client:

```php
require 'vendor/autoload.php';

// create a new persistent client
$m = new Memcached("memcached_pool");
$m->setOption(Memcached::OPT_BINARY_PROTOCOL, TRUE);

// some nicer default options
// - nicer TCP options
$m->setOption(Memcached::OPT_TCP_NODELAY, TRUE);
$m->setOption(Memcached::OPT_NO_BLOCK, FALSE);
// - timeouts
$m->setOption(Memcached::OPT_CONNECT_TIMEOUT, 2000);    // ms
$m->setOption(Memcached::OPT_POLL_TIMEOUT, 2000);       // ms
$m->setOption(Memcached::OPT_RECV_TIMEOUT, 750 * 1000); // us
$m->setOption(Memcached::OPT_SEND_TIMEOUT, 750 * 1000); // us
// - better failover
$m->setOption(Memcached::OPT_DISTRIBUTION, Memcached::DISTRIBUTION_CONSISTENT);
$m->setOption(Memcached::OPT_LIBKETAMA_COMPATIBLE, TRUE);
$m->setOption(Memcached::OPT_RETRY_TIMEOUT, 2);
$m->setOption(Memcached::OPT_SERVER_FAILURE_LIMIT, 1);
$m->setOption(Memcached::OPT_AUTO_EJECT_HOSTS, TRUE);

// setup authentication
$m->setSaslAuthData( <MEMCACHIER_USERNAME>
                   , <MEMCACHIER_PASSWORD> );

// We use a consistent connection to memcached, so only add in the
// servers first time through otherwise we end up duplicating our
// connections to the server.
if (!$m->getServerList()) {
    // parse server config
    $servers = explode(",", <MEMCACHIER_SERVERS>);
    foreach ($servers as $s) {
        $parts = explode(":", $s);
        $m->addServers($parts[0], $parts[1]);
    }
}
```

The values for `<MEMCACHIER_SERVERS>`, `<MEMCACHIER_USERNAME>`, and
`<MEMCACHIER_PASSWORD>` are listed on your [cache overview
page](https://www.memcachier.com/caches).

You should look at the PHP [Memcached client
documentation](http://www.php.net/manual/en/book.memcached.php) for a list of
API calls you can make against MemCachier.

We’ve built a small PHP example here: [MemCachier PHP sample
app](https://github.com/memcachier/examples-php).

<h3 id="php-session">PHP Session Support</h3>

You can configure PHP to store sessions in MemCachier as follows.

First, start by configuring an appropriate `.user.ini` in your document root.
It should contain the following:

```php
session.save_handler=memcached
memcached.sess_binary=1
session.save_path="PERSISTENT=myapp_session <MEMCACHIER_SERVERS>"
memcached.sess_sasl_username=<MEMCACHIER_USERNAME>
memcached.sess_sasl_password=<MEMCACHIER_PASSWORD>
```

In your code you should then be able to run:

```php
// Enable MemCachier session support
session_start();
$_SESSION['test'] = 42;
```

<h3 id="php-memcachesasl">Alternative PHP Client -- MemcacheSASL</h3>

This is not our recommended client for using MemCachier from PHP. We recommend
the [php memcached](#php) client. However, it is an easier client to use as
it's a pure PHP implementation while the [recommended php client](#php)
requires a C extension to be installed with
[SASL](http://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer)
support. It doesn't support multiple proxy servers like the memcached client
but is otherwise quite good.

You should first install the
[PHPMemcacheSASL](https://github.com/memcachier/PHPMemcacheSASL) client. You
can either grab the code directly or use [composer](https://getcomposer.org/)
for package management. We suggest composer.

First, if using composer, you'll need to modify your `composer.json` file to
include the module:

```js
{
    "require": {
        "php": ">=5.3.2",
        "memcachier/php-memcache-sasl": ">=1.0.1"
    }
}
```

Then, you can connect to MemCachier using the client:

```php
require 'vendor/autoload.php';
use MemCachier\MemcacheSASL;

// Create client
$m = new MemcacheSASL();
$servers = explode(",", <MEMCACHIER_SERVERS>);
foreach ($servers as $s) {
    $parts = explode(":", $s);
    $m->addServer($parts[0], $parts[1]);
}

// Setup authentication
$m->setSaslAuthData( getenv("MEMCACHIER_USERNAME")
                   , getenv("MEMCACHIER_PASSWORD") );

$m->add("foo", "bar");
echo $m->get("foo");
```

The values for `<MEMCACHIER_SERVERS>`, `<MEMCACHIER_USERNAME>`, and
`<MEMCACHIER_PASSWORD>` are listed on your [cache overview
page](https://www.memcachier.com/caches).

We’ve built a small PHP example here: [MemCachier PHP sample
app](https://github.com/memcachier/examples-php).


<h2 id="wordpress">WordPress</h2>

We support WordPress through two different options. The first you can find
[here](https://github.com/memcachier/wordpress-cache), and uses the binary
memcache protocol and is supported by us. It should be easy to install, simply
follow the instructions on the Git repo.

We also have a community support alternative approach that uses the ASCII
memcache protocol, which is far easier to get running on your own servers if
you aren't using a hosted platform like Heroku. You can find it on GitHub
[here](https://github.com/hubertnguyen/memcachier). Please follow the
instructions on the repo page.


<h2 id="cakephp">CakePHP</h2>

The CakePHP framework has excellent support for caching and can be easily used
with MemCachier as the provider. To setup CakePHP with MemCachier, you'll need
to edit the file `app/Config/bootstrap.php` and add the following lines:

```php
Cache::config('default', array(
    'engine' => 'Memcached',
    'prefix' => 'mc_',
    'duration' => '+7 days',
    'servers' => explode(',', <MEMCACHIER_SERVERS>),
    'compress' => false,
    'persistent' => 'memcachier',
    'login' => <MEMCACHIER_USERNAME>,
    'password' => <MEMCACHIER_PASSWORD>,
    'serialize' => 'php'
));
```

After that, you should be able to use caching throughout your application like so:

```php
class Post extends AppModel {

    public function newest() {
        $model = $this;
        return Cache::remember('newest_posts', function() use ($model){
            return $model->find('all', array(
                'order' => 'Post.updated DESC',
                'limit' => 10
            ));
        }, 'longterm');
    }
}
```

The above will fetch the value associated with the key `newest_posts` from the
cache if it exists. Otherwise, it will execute the function and SQL query,
storing the result in the cache using the `newest_posts` key.

You can find much more information on how to use caching with CakePHP
[here](http://book.cakephp.org/2.0/en/core-libraries/caching.html).


<h2 id="symfony2">Symfony2</h2>

The [Symfony2](http://symfony.com/) framework is a great choice with
MemCachier. It supports caching and storing sessions in MemCachier.

First, start by configuring an appropriate `.user.ini` in your document root.
It should contain the following:

```php
session.save_handler=memcached
memcached.sess_binary=1
session.save_path="PERSISTENT=myapp_session <MEMCACHIER_SERVERS>"
memcached.sess_sasl_username=<MEMCACHIER_USERNAME>
memcached.sess_sasl_password=<MEMCACHIER_PASSWORD>
```


<h2 id="laravel">Laravel</h2>

We have community provided support for the Laravel PHP webframework. You can
find a caching backend with MemCachier support on Github
[here](https://github.com/b3it/laravel-memcached-plus). Please follow the
instructions on the repo page.


<h2 id="node.js">Node.js</h2>

For Node.js we recommend the use of the
[memjs](https://github.com/alevy/memjs) client library. It is written
and supported by MemCachier itself! To install, use the [node package
manager (npm)](https://npmjs.org/):

```text
npm install memjs
```

Using it is straight-forward as memjs understands the `MEMCACHIER_SERVERS`,
`MEMCACHIER_USERNAME` and `MEMCACHIER_PASSWORD`
environment variables that the MemCachier add-on setups. For example:

```javascript
var memjs = require('memjs')
var mc = memjs.Client.create()
mc.get('hello', function(val) {
    alert(val)
})
```

We’ve built a small Node.js example here: [MemCachier Node.js sample
app](http://github.com/memcachier/examples-node).


<h2 id="java">Java</h2>

For Java we recommend using the
[SpyMemcached](https://code.google.com/p/spymemcached/) client. We also
recommend using the [Apache Maven](https://maven.apache.org/) build manager for
working with Java applications. If you aren't using `maven` and are instead
using [Apache Ant](https://ant.apache.org/) or your own build system, then
simply add the `spymemcached` jar file as a dependency of your application.

For `maven` however, start by configuring it to have the proper `spymemcached`
repository:

```xml
<repository>
  <id>spy</id>
  <name>Spy Repository</name>
  <layout>default</layout>
  <url>http://files.couchbase.com/maven2/</url>
  <snapshots>
    <enabled>false</enabled>
  </snapshots>
</repository>
```

Then add the `spymemcached` library to your dependencies:

```xml
<dependency>
  <groupId>spy</groupId>
  <artifactId>spymemcached</artifactId>
  <version>2.12.1</version>
  <scope>provided</scope>
</dependency>
```

Once your build system is configured, you can start adding caching to your Java
app:

```java
import java.io.IOException;
import net.spy.memcached.AddrUtil;
import net.spy.memcached.MemcachedClient;
import net.spy.memcached.ConnectionFactoryBuilder;
import net.spy.memcached.auth.PlainCallbackHandler;
import net.spy.memcached.auth.AuthDescriptor;

public class Foo {
  public static void main(String[] args) {
    AuthDescriptor ad = new AuthDescriptor(new String[] { "PLAIN" },
        new PlainCallbackHandler(<MEMCACHIER_USERNAME>,
            <MEMCACHIER_PASSWORD>));

    try {
      MemcachedClient mc = new MemcachedClient(new ConnectionFactoryBuilder()
          .setProtocol(ConnectionFactoryBuilder.Protocol.BINARY)
          .setAuthDescriptor(ad).build(), AddrUtil.getAddresses(<MEMCACHIER_SERVERS>));
      mc.set("foo", "bar");
      System.out.println(mc.get("foo"));
    } catch (IOException ioe) {
      System.err.println("Couldn't create a connection to MemCachier: \nIOException "
              + ioe.getMessage());
    }
  }
}
```

The values for `<MEMCACHIER_SERVERS>`, `<MEMCACHIER_USERNAME>`, and
`<MEMCACHIER_PASSWORD>` are listed on your [cache overview
page](https://www.memcachier.com/caches).

You may wish to look the `spymemcached`
[JavaDocs](https://dustin.github.com/java-memcached-client/apidocs/) or some
more [example code](https://code.google.com/p/spymemcached/wiki/Examples) to
help in using MemCachier effectively.

We’ve built a small Java example here, using Jetty: [MemCachier Java Jetty
sample app](https://github.com/memcachier/examples-java).


<h2 id="clients">Client library support</h2>

MemCachier will work with any memcached binding that supports [SASL
authentication](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer)
and the [binary
protocol](https://github.com/memcached/memcached/wiki/BinaryProtocolRevamped). We
have tested MemCachier with the following language bindings, although the
chances are good that other SASL binary protocol packages will also work.

<table class="table table-striped table-bordered">
<tbody>
<tr>
<th>Language</th>
<th>Bindings</th>
</tr>
<tr>
<td>Ruby</td>
<td><a href="https://github.com/mperham/dalli">dalli</a></td>
</tr>
<tr>
<td>Python</td>
<td>
  <a href="http://sendapatch.se/projects/pylibmc/">pylibmc</a>
  <b>or</b>
  <a href="https://github.com/jaysonsantos/python-binary-memcached">python-binary-memcached</a>
</td>
</tr>
<tr>
<td>Django</td>
<td>
  (
    <a href="http://sendapatch.se/projects/pylibmc/">pylibmc</a>
    <b>and</b>
    <a href="https://github.com/jbalogh/django-pylibmc">django-pylibmc</a>
  )
  <b>or</b>
  (
    <a href="https://github.com/jaysonsantos/python-binary-memcached">python-binary-memcached</a>
    <b>and</b>
    <a href="https://github.com/jaysonsantos/django-bmemcached">django-bmemcached</a>
  )
  <b>or</b>
  (
    <a href=https://github.com/memcachier/django-ascii>django-ascii</a>
  )
</td>
</tr>
<tr>
<td>PHP</td>
<td>
  <a href="http://www.php.net/manual/en/book.memcached.php">php-memcached</a>
  <b>or</b>
  <a href="http://github.com/ronnywang/PHPMemcacheSASL">PHPMemcacheSASL</a>
</td>
</tr>
<tr>
<td>Node.js</td>
<td>
  <a href="https://github.com/alevy/memjs">memjs</a>
</td>
</tr>
<tr>
<td>Java</td>
<td>
  <a href="http://code.google.com/p/spymemcached/">SpyMemcached</a>
  <b>or</b>
  <a href="https://code.google.com/p/xmemcached/">XMemcached</a>
</td>
</tr>
<tr>
<td>Go</td>
<td><a href="https://github.com/memcachier/mc">mc</a></td>
</tr>
<tr>
<td>Haskell</td>
<td><a href="http://hackage.haskell.org/package/memcache">memcache</a></td>
</tr>
</tbody>
</table>


<h2 id="sample-apps">Sample applications</h2>

We've built a number of working sample apps, too. Though keep in mind that
these apps are built for Heroku, so they expect `MEMCACHIER_SERVERS`,
`MEMCACHIER_USERNAME`, and `MEMCACHIER_PASSWORD` to be in the environment.

* [Sinatra Example](https://github.com/memcachier/examples-sinatra)
* [Rails Example](https://github.com/memcachier/examples-rails)
* [Rack::Cache Example](https://github.com/memcachier/examples-rack-cache)
* [Django Example](https://github.com/memcachier/examples-django)
* [PHP Example](https://github.com/memcachier/examples-php)
* [Node.js Example](https://github.com/memcachier/examples-node)
* [Java Example](https://github.com/memcachier/examples-java)

We also have a collection of [example
code](https://github.com/memcachier/example-code) that can be a useful
reference source.


<h2 id="local">Local usage</h2>

To test against your application locally, you will need to run a local
memcached process. MemCachier is only available from the datacenter you signed
up for. But because MemCachier and memcached speak the same protocol, you
shouldn’t have any issues testing locally. Installation depends on your
platform.

<p class="alert alert-info">The below examples will install memcached without
SASL authentication support. This is generally what you want as client code can
still try to use SASL auth and memcached will simply ignore the requests which
is the same as allowing any credentials. So your client code can run without
modification locally.</p>

On Ubuntu:

```text
$ sudo apt-get install memcached
```

Or on OS X (with Homebrew):

```text
$ brew install memcached
```

Or for Windows please refer to [these
instructions](http://www.heckler.com.br/blog/2013/05/10/memcached-on-windows/)

For further information and resources (such as the memcached source code)
please refer to the [Memcache.org homepage](http://memcached.org)

To run memcached simply execute the following command:

```text
$ memcached -v
```


<h2 id="analytics">MemCachier analytics</h2>

Our analytics dashboard is a simple tool that gives you more insight into how
you’re using memcache. Here's a screenshot of the dashboard:

<p style="text-align:center;">
<img style="width:80%;" src="/images/analytics.png" alt="Analytics dashboard">
</p>

To access your application's analytics dashboard login to your
[account](https://www.memcachier.com/caches) and view one of your caches.

The analytics displayed are:

* _Limit_ -- Your current cache size and memory limit. Once usage comes
  close to this amount you will start seeing evictions.
* _Live Connections_ -- Number of connections currently open to your
  cache.
* _Total connections_ -- Number of connections ever made to your cache.
  (So always larger than live connections).
* _Items_ -- Number of items currently stored in your cache.
* _Evictions_ -- Number of items ever evicted from your cache due to
  memory pressure. Items are evicted in an LRU order.
* _New Evictions_ -- Number of evictions that have occured since the
  last time we sampled your cache.
* _Hit Rate_ -- The ratio of `get` commands that return an item (hit)
  vs. the number that return nothing (miss). This ratio is for the
  period between now and when we last sampled your cache.
* _Set Cmds_ -- Number of times you have ever performed a set command.
* _Flush Cmds_ -- Number of times you have ever performned a flush
  command.

With the basic analytics dashboard we sample your cache once per hour.
With the advance dashboard we sample it once every 30 minutes.

<h3 id="advanced-analytics">Advanced analytics</h3>

We offer higher paying customers an advance version of our analytics
dashboard. Currently, this offers two primary advantages:

* _Higher Sample Rate_ -- We sample the cache for collecting analytics
  once every thirty minutes, twice the rate of the basic analytics
  dashboard. We don't sample more often than that as a higher
  granularity hasn't proven to be useful, it leads to more noise and
  less signal.
* _More Graphs_ -- We offer two additional graphs for the advanced
  analytics dashboard.
  * _Eviction Graph_ -- Your new evictions tracked over time.
  * _Connection Graph_ -- Your new connecions tracked over time.

Please note that our graph only allows two data sources to be selected at a
time. So if two are already selected, say "Usage" and "Hit Rate", to select a
different data source, say "Evictions", you'll need to deselect an existing
data source first before selecting the new one to display.

<h2 id="analytics-api">Analytics API</h2>

You can also access features available on the analytics dashboard via the API.
- [Authenticate](#api-auth)
- [Memcachier API ID](#api-id)
- [Stats](#api-stats)
- [History](#api-history)
- [Flush](#api-flush)
- [List Credentials](#api-list-cred)
- [Add Credentials](#api-create-cred)
- [Update Credentails](#api-update-cred)
- [Promote Credentials](#api-promote-cred)
- [Delete Credentials](#api-delete-cred)

<h3 id='api-auth'>Authentication</h3>

Memcachier uses credentials to allow access to the API. After you've created a cache, you can find your credentials on the [analytics dashboard](https://analytics.memcachier.com/). Only credentials that have the API capability will be allowed to use this API.

Memcachier expects for your credentials to be included in the header of all API requests.

```shell
curl "https://analytics.memcachier.com/api/v1/:memcachier_id/:action"
  --user CRED_USERNAME:CRED_PASSWORD
```

> Make sure to replace `CRED_USERNAME:CRED_PASSWORD` with your credential username and password found on the analytics dashboard.

<h3 id='api-id'>Memcachier API ID</h3>

All of the API paths include a `<memcachier_id>` variable. In order to find this ID, you'll need to use the `/login` path.
> This is not the same thing as the "Memcachier ID" listed on your analytics dashboard.

**HTTP Request**

`GET https://analytics.memcachier.com/api/login`

**Response**


<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>JSON response with memcachier ID</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>404</td>
<td>"No cache found."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error"</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/login"
  --user CRED_USERNAME:CRED_PASSWORD
```

**Returns**:

```json
{
    "cache_id": 1561
}
```

<h3 id='api-stats'>Stats</h3>

This endpoint retrieves all the statistics for your cache.

**HTTP Request**

`GET https://analytics.memcachier.com/api/v1/<memcachier_id>/stats`

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>JSON response with stats</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error: a.b.c.totallyaserver.com:1234,..."</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/stats"
  --user "CRED_USERNAME:CRED_PASSWORD"
```

**Returns:**

```json
{
    "a.b.c.totallyaserver.com:12345": {
        "auth_cmds": 19,
        "auth_errors": 0,
        "bytes": 0,
        "bytes_read": 960,
        "bytes_written": 22233,
        "cas_badval": 0,
        "cas_hits": 0,
        "cas_misses": 0,
        "cmd_delete": 0,
        "cmd_flush": 0,
        "cmd_get": 0,
        "cmd_set": 0,
        "cmd_touch": 0,
        "curr_connections": 0,
        "curr_items": 0,
        "decr_hits": 0,
        "decr_misses": 0,
        "delete_hits": 0,
        "delete_misses": 0,
        "evictions": 0,
        "expired": 0,
        "get_hits": 0,
        "get_misses": 0,
        "incr_hits": 0,
        "incr_misses": 0,
        "limit_maxbytes": 28835840,
        "time": 1500651085,
        "total_connections": 19,
        "total_items": 0,
        "touch_hits": 0,
        "touch_misses": 0
    }
}
```

<h3 id='api-history'>History</h3>

This endpoint retrieves the statistical history of a cache.

**HTTP Request**

`GET https://analytics.memcachier.com/api/v1/<memcachier_id>/history`

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>JSON response with historical stats</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error"</td>
</tr>
</tbody>
</table>


**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/history"
  --user "CRED_USERNAME:CRED_PASSWORD"
```

**Returns:**

```json
  [{
    "memcachier_id": "11",
    "server": "a.b.c.totallyaserver.memcachier.com",
    "stats": {
        "auth_cmds": 158,
        "auth_errors": 0,
        "bytes": 0,
        "bytes_read": 3840,
        "bytes_written": 178754,
        "cas_badval": 0,
        "cas_hits": 0,
        "cas_misses": 0,
        "cmd_delete": 0,
        "cmd_flush": 0,
        "cmd_get": 0,
        "cmd_set": 0,
        "cmd_touch": 0,
        "curr_connections": 2,
        "curr_items": 0,
        "decr_hits": 0,
        "decr_misses": 0,
        "delete_hits": 0,
        "delete_misses": 0,
        "evictions": 0,
        "expired": 0,
        "get_hits": 0,
        "get_misses": 0,
        "incr_hits": 0,
        "incr_misses": 0,
        "limit_maxbytes": 1153433600,
        "time": 1490731542,
        "total_connections": 158,
        "total_items": 0,
        "touch_hits": 0,
        "touch_misses": 0
    },
    "timestamp": "1490731541096"
    }, … ]
```

<h3 id='api-flush'>Flush</h3>

This endpoint will flush all of the data from the cache cache.

**HTTP Request**

`POST https://analytics.memcachier.com/api/v1/<memcachier_id>/flush`

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>""</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error: a.b.c.totallyaserver.com:1234,..."</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/flush" -X POST
  --user "CRED_USERNAME:CRED_PASSWORD"
```

> Certain credentials may not have permission to flush the cache, which will produce a 403 error.

<h3 id='api-list-cred'>List Credentials</h3>

The endpoint returns a list of all the credentials connected to the cache.

**HTTP Request**

`GET https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials`

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>JSON list of credentials</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error"</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials"
  --user "CRED_USERNAME:CRED_PASSWORD"
```

**Returns:**

```json
  [
    {
        "cache_id": 14,
        "flush_capability": false,
        "id": 44,
        "sasl_username": "CRED_USERNAME",
        "uuid": null,
        "write_capability": true,
        "api_capability": true,
        "primary": true,
    },
    {
        "cache_id": 14,
        "flush_capability": false,
        "id": 43,
        "sasl_username": "789101",
        "uuid": null,
        "write_capability": true,
        "api_capability": true,
        "primary": false,
    }, ...
]
```

<h3 id='api-create-cred'>Create Credentials</h3>

This endpoint creates a new set of credentials which can be used to connect to the cache.

**HTTP Request**

`POST https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials`

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>JSON of new credential set</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error"</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials" -X POST
  --user "CRED_USERNAME:CRED_PASSWORD"
```

**Returns:**

```json
{
    "cache_id": 14,
    "id": null,
    "sasl_password": "CRED_PASSWORD",
    "sasl_username": "CRED_USERNAME",
    "uuid": null,
    "primary": false,
    "flush_capability": true,
    "write_capability": true,
    "api_capability": true,
}
```

<h3 id='api-update-cred'>Update Credentials</h3>

This endpoint updates the capabilities of a specific set of credentials.

**HTTP Request**

`POST https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials/<cred_id>`

**Query Parameters**

<table class="table table-bordered">
<tbody>
<tr>
<th>Parameter</th>
<th>Default</th>
<th>Description</th>
</tr>
<tr>
<td>flush_capability</td>
<td>true</td>
<td>Authorize this set of credentials to flush the cache.</td>
</tr>
<tr>
<td>write_capability</td>
<td>true</td>
<td>Authorize this set of credentials to write to the cache.</td>
</tr>
<tr>
<td>api_capability</td>
<td>true</td>
<td>Authorize this set of credentials to use this API.</td>
</tr>
</tbody>
</table>

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>JSON of new credential properties</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error"</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials/<cred_username>" -X PATCH
  -d '{"flush_capability":"false"}'
  --user "CRED_USERNAME:CRED_PASSWORD"
```

**Returns:**

```json
{
    "flush_capability": false,
    "write_capability": true,
    "api_capability": true,
}
```

<h3 id='api-promote-cred'> Promote Credentials</h3>

This endpoint promotes a set of credentials to primary.

**HTTP Request**

`POST https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials/primary/<cred_id>`

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>JSON of promoted credential set</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error"</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials/primary/<cred_username>" -X POST
  --user "CRED_USERNAME:CRED_PASSWORD"
```

**Returns:**

```json
{
    "cache_id": 14,
    "id": null,
    "sasl_username": "CRED_USERNAME",
    "uuid": null,
    "primary": true,
    "flush_capability": false,
    "write_capability": true,
    "api_capability": true,
}
```

<h3 id='api-delete-cred'>Delete Credentials</h3>

This endpoint deletes a set of credentials

**HTTP Request**

`POST https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials/primary/<cred_id>`

**Response**

<table class="table table-bordered">
<tbody>
<tr>
<th>Status</th>
<th>Response</th>
</tr>
<tr>
<td>200</td>
<td>""</td>
</tr>
<tr>
<td>403</td>
<td>"You are not authorized to perform this action."</td>
</tr>
<tr>
<td>500</td>
<td>"Server error"</td>
</tr>
</tbody>
</table>

**Example:**

```shell
curl "https://analytics.memcachier.com/api/v1/<memcachier_id>/credentials/primary/<cred_username>" -X DELETE
  --user "CRED_USERNAME:CRED_PASSWORD"
```


<h2 id="newrelic">New Relic integration</h2>

MemCachier supports integration with your New Relic dashboard if you
happen to be a customer of both MemCachier and New Relic. Currently
this feature is only available to caches of *500MB* or larger. A blog
post showing the integration can be found
[here](http://blog.memcachier.com/2014/03/05/memcachier-and-new-relic-together/).

To setup the integration you will need to find your New Relic license key. This
can be done by going to your "Account Settings" page when logged in to New
Relic by click on your New Relic username in the top right corner. Then you
will find your license key in the right side information column. It should be
exactly 40 characters long. Please refer to the [blog
post](http://blog.memcachier.com/2014/03/05/memcachier-and-new-relic-together/)
for a visual walkthrough.

Once you have your New Relic licence key, it can be entered for your cache on
the analytics dashboard page. In the bottom right corner there is a button to
do this.


<h2 id="credentials">Credentials</h2>

In order to connect a memcache client to MemCachier, you use a
username and password listed on the [analytics dashboard](#analytics)
for your cache. Each cache can have multiple sets of credentials.
One of these sets of credentials is distinguished as
*primary*, meaning that, for hosted platforms like Heroku,
it is linked to the hosted platform MemCachier addon.

From the *Credentials* panel on the analytics dashboard, it
is possible to create new credentials, delete existing credentials and
promote secondary credentials to primary credentials. This makes it
possible to rotate credentials by creating a new set of secondary
credentials and promoting them to primary. For caches associated with
hosted platforms, promoting a set of secondary credentials to primary
causes the configuration variables on the hosted platform to be
updated. For example, rotating the credentials on a Heroku-associated
cache causes an update of the `MEMCACHIER_USERNAME` and
`MEMCACHIER_PASSWORD` configuration variables on your Heroku app and a
restart of your dynos to pick up the new values.

Each set of credentials for a cache can be given different
*capabilities*, in the sense that sets of credentials can be
restricted to read-only access to the cache, or prevented from
flushing the cache via the memcache API. These capabilities are
controlled by checkboxes on the *Credentials* panel of the analytics
dashboard. (The exact error conditions that a client will receive if
it attempts to perform an action for which it does not have the
capability depends on the details of the client library used. The
most common cases are likely to be for the Dalli Ruby library and the
pylibmc Python library. For both of these client libraries,
attempting to set a cache entry using credentials that do not have the
write capability will simply result in a "value not set" response from
the library.)

<h2 id="disabled-caches">Disabled Caches</h2>

Development caches are disabled after 30 days of inactivity. Disabled caches
can no longer be accessed (authentication will fail). To use the caches again
it can be re-enabled on the [analytics dashboard](#analytics) under the
*Settings* panel.

<h2 id="tls">Encrypted Connections (TLS)</h2>

<p class="alert alert-info">
We generally don't recommend using TLS to secure your connection. Why?
Memcache is normally only used when performance is important and so
low latency is critical. This means we expect your MemCachier cache
and your application that accesses it runs in the same datacenter, for
example the Amazon EC2 `us-east-1` datacenter. All your traffic are
running over, and only over, the internal datacenter network. This is
a <strong>highly secure</strong> network that can't be sniffed on or
tampered with. For example, your web application is probably speaking
HTTPS, but the HTTPS connection is very likely terminated at a load
balancer, and then unsecured HTTP used to talk between the load
balancer and your application.
</p>

It is possible to connect to MemCachier using TLS encrypted sockets.
While no existing clients support TLS connections natively, we provide
a [buildpack](https://github.com/memcachier/memcachier-tls-buildpack)
for Heroku customers that proxies the connection to MemCachier and
wraps it in a TLS connection. This can be useful for the extra
paranoid among us, or to securely access your cache from outside the
datacenter.

The buildpack installs and sets up
[stunnel](https://www.stunnel.org/index.html) on localhost listening
on port 11211. It configures stunnel to connect to the MemCachier
servers specified in your environment variable and to verify
certificates as signed by the [MemCachier Root
CA](https://www.memcachier.com/MemCachierRootCA.pem).

Use the buildpack in conjunction with another buildpack that actually
runs your app, using Heroku's [multiple
buildpack](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app)
feature:

```term
$ heroku buildpacks:add https://github.com/memcachier/memcachier-tls-buildpack.git
```

Finally, configure your app to connect to `localhost:11211` instead of
using the `MEMCACHIER_SERVERS` environment variable, but, **leave your
`MEMCACHIER_SERVERS` environment variable unchanged as the TLS
buildpack uses it to connect to MemCachier**.


<h2 id="upgrading">Upgrading and downgrading</h2>

Changing your plan, either by upgrading or downgrading, requires no code
changes. Your cache won't be lost, either. Upgrading and downgrading Just
Works™. Also, you are only ever charged by the hour for the time that you are
on a certain plan. So try experimenting with different cache sizes knowing that
you will only be charged for the hours you are on a plan, not for a whole
month.

Changing your plan, either by upgrading or downgrading, can be done easily at
any time through your
[account](https://www.memcachier.com/caches).

* No code changes are required.
* Your cache won't be lost or reset<strong>*</strong>.
* You are charged by the hour for plans, so try experimenting with
  different cache sizes with low cost.

<p class="alert alert-info">
<strong>*</strong> When moving between the development plan to a
production plan, you <strong>will</strong> loose your cache. This is
unavoidable due to the strong separation between the development and
production clusters.
</p>


<h2 id="using">Using MemCachier</h2>

Please refer to your client or framework documentation for how to use
MemCachier effectively.

MemCachier Guides:

* [Advanced Memcache Usage](https://devcenter.heroku.com/articles/advanced-memcache)
* [Building a Rails 3 App with MemCachier](https://devcenter.heroku.com/articles/building-a-rails-3-application-with-memcache)
* [Rails + Rack::Cache + MemCachier](https://devcenter.heroku.com/articles/rack-cache-memcached-rails31)
* [Django and MemCachier](https://devcenter.heroku.com/articles/django-memcache)

Framework and Client Documentation:

* [Dalli (Ruby Client) API](http://www.rubydoc.info/github/mperham/dalli/Dalli/Client)
* [Rails Caching Guide](http://guides.rubyonrails.org/caching_with_rails.html)
* [PHP Memcached client](http://www.php.net/manual/en/book.memcached.php)
* [CakePHP Caching Guide](http://book.cakephp.org/2.0/en/core-libraries/caching.html)
* [Pylibmc (Pytnon Client) API](http://sendapatch.se/projects/pylibmc/)
* [Django Caching Guide](https://docs.djangoproject.com/en/dev/topics/cache/)
* [MemJS (node.js client) API](http://amitlevy.com/projects/memjs/)
* [Spymemcached JavaDocs](http://dustin.github.com/java-memcached-client/apidocs/)


<h2 id="1mb-limit">Key-Value size limit (1MB)</h2>

MemCachier has a maximum size that a key-value object can be of
__1MB__. This applies to both key-value pairs created through a `set`
command, or existing key-value pairs grown through the use of an
`append` or `prepend` command. In the later case, the size of the
key-value pair with the new data added to it, must still be less than
1MB.

The 1MB limit applies to the size of the key and the value together. A
key of size 512KB with a value of 712KB would be in violation of the
1MB limit.

The reason for this has partially to do with how memory is managed in
MemCachier. A limitation of the high performance design is a
restriction on how large key-value pairs can become. Another reason is
that storing values larger than 1MB doesn't normally make sense in a
high-performance key-value store. The network transfer time in these
situations becomes the limiting factor for performance. A disk cache
or even a database makes sense for this size value.


<h2 id="localhost-errors">Errors about app trying to connect to localhost</h2>

By default, most memcache client look for the environment variables,
`MEMCACHE_SERVERS`, `MEMCACHE_USERNAME` and `MEMCACHE_PASSWORD` for
their configuration. These variables are used when the initialization
code for the memcache client doesn't specifically specify these values.

If these environment variables aren't set, clients generally default
to connecting to `127.0.0.1:11211` (i.e., localhost), with no username
and password.

The MemCachier add-on sets the `MEMCACHIER_SERVERS`,
`MEMCACHIER_USERNAME` and `MEMCACHIER_PASSWORD` environment variables.
So you need to either set the equivalent `MEMCACHE_*` variables from
these, or pass these values to your client when you create a new one
in your code.

For example, pseudo-code for the first approach is:

    env[MEMCACHE_SERVERS] = env[MEMCACHIER_SERVERS]
    env[MEMCACHE_USERNAME] = env[MEMCACHIER_USERNAME]
    env[MEMCACHE_PASSWORD] = env[MEMCACHIER_PASSWORD]

While pseudo-code for the second approach is:

    memClient = new MemcacheClient(ENV['MEMCACHIER_SERVERS'],
                                   ENV['MEMCACHIER_USERNAME'],
                                   ENV['MEMCACHIER_PASSWORD'])

Please be careful that you have setup the your client in all
locations. Many frameworks, such as Rails, use memcache in multiple
ways and may require you to setup initialization properly in a few
locations. Generally the first approach is preferred as it is global
while the second approach is local to each initialization.

For example, with Ruby on Rails, you'll need to setup `cache_store`,
`Rack::Cache` and the `session_store`.


<h2 id="support">Support</h2>

All MemCachier support and runtime issues should be submitted via email to <a
href="mailto:support@memcachier.com"><i class="icon-envelope"></i>
support@memcachier.com</a> or through our [support
site](http://support.memcachier.com). Please include your `MEMCACHIER_USERNAME`
with support tickets.

Any non-support related issues or product feedback is welcome via
email at: [support@memcachier.com](mailto:support@memcachier.com).

Any issues related to MemCachier service are reported at [MemCachier
Status](http://status.memcachier.com/).

Please also follow us on twitter, <a
href="https://twitter.com/MemCachier">@memcachier</a>, for status and product
announcements.
