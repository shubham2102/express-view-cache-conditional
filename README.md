express-view-cache
==================
[![NPM version](https://badge.fury.io/js/express-view-cache.svg)](http://badge.fury.io/js/express-view-cache)
[![Build Status](https://travis-ci.org/vodolaz095/express-view-cache.png)](https://travis-ci.org/vodolaz095/express-view-cache)
[![Dependency Status](https://gemnasium.com/vodolaz095/express-view-cache.svg)](https://gemnasium.com/vodolaz095/express-view-cache)

Unobtrusive solution to express 4.0.0 framework - cache response content in Redis database.

Why do we need this plugin and how does it work?
==================

Let's consider we have a NodeJS application with code like this:

    app.get('/getPopularPosts',function(req,res){
        req.model.posts.getPopular(function(err,posts){
            if(err) throw err;
            res.render('posts',{"posts":posts});
        });
    });

The method `getPopular` of `posts` requires a call to database and executed slowly. Also rendering the template of posts
requires some time. So, maybe we need to cache all this? Ideally, when visitor gets the page with url  `/getPopularPosts`
we have to give him info right from cache, without requests to database, parsing data received, rendering page and other things
we need to do to give him this page. The most expressJS way to do it is to make a separate middleware, that is ran before
router middleware, and returns page from cache (if it is present in cache) or pass data to other middlewares, but this caching
middleware adds a listener to response, which SAVES rendered response to cache. And for future use, the response is taken from CACHE!

It is turned that it works best with [node-redis](https://github.com/mranney/node_redis) with [redis](http://redis.io) >v2.6.16



Example
==================
There is a complete example of NodeJS + ExpressJS (4.x.x) application which responds with current time.

```javascript

    var express = require('express'),
      morgan = require('morgan'),
      errorHandler = require('errorhandler'),
      request = require('request'),
      http = require('http'),
      EVC = require('express-view-cache'),
      app = express(),
      evc = EVC('redis://redis:someLongAuthPassword@localhost:6379');

    app.set('port', process.env.PORT || 3000);
    app.use(morgan('dev'));
    app.use('/cacheFor5sec', evc.cachingMiddleware(5000));
    app.use('/cacheFor3sec', evc.cachingMiddleware(3000));
    app.get('*', function (request, response) {
      response.json({
        'Page Created At': new Date().toLocaleTimeString()
      });
    });
    app.use(errorHandler());

    http.createServer(app).listen(app.get('port'), function () {
      console.log("Express server listening on port " + app.get('port'));

      setInterval(function(){
        request('http://localhost:'+app.get('port')+'/', function(error, response, body){
          console.log('GET /',body);
        })
      }, 1000);

      setInterval(function(){
        request('http://localhost:'+app.get('port')+'/cacheFor3sec', function(error, response, body){
          console.log('GET /cacheFor3sec',body);
        })
      }, 1000);
    });

```


Options
==================

    var evc = EVC(options);
    app.use('/someURItoBeCachedForFiveSeconds', evc.cachingMiddleware(5000));

`Options` can be a redis connection string like `redis://usernameTotallyIgnored:someLongPassword@redis.example.org:6379`

also `options` can be an dictionary object with this fields:

* `host` - default is `localhost` - the hostname of redis server
* `port` - default is `6379` - the port where the redis server listens
* `pass` - [password for redis authorization](https://github.com/mranney/node_redis#clientauthpassword-callback), default is null
* `client` - ready to use [node-redis](https://github.com/mranney/node_redis) client - this option overrides all previous
* `appPort` - port of nodejs application to use, default is `process.env.PORT` or `3000`

Tests
==================

    $ npm test

License
====================
The MIT License (MIT)

Copyright (c) 2014 Shubham Singh shubham2102@gmail.com et al.

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


