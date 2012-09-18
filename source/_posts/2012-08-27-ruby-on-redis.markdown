---
layout: post
title: "Ruby on Redis"
date: 2012-08-27 10:00
comments: true
sharing: true
categories: [ruby, redis]
author: Sebastian Borrazas
description: Easy introduction to Redis gem on a Ruby project
---

Ever since I found out about the no-sql databases and since they became very popular lately, I wanted to write a blog post about my favourite one: [Redis](http://redis.io/).

What I love so much about it is its simple API. In my honest opinion, one of the very best API's that I've found so far.

Also, since it's a lightweight and fast key-value store, it's specially useful for speeding up slow processes on your system. If things go slow, you should always consider using Redis, if it applies, it will speed things up.

<!-- more -->

The Ruby gem redis is as easy to use as redis API.
Here's a quick look of it's usage on Rails. Where there's an AR Ranking model, which stores in a sorted set a list of albums to easily get the top [n] albums.

``` ruby On config/initializers/redis.rb
# Setup the connection to the redis server
$redis = Redis.new(host: 'localhost', port: 6379)
```

``` ruby On app/models/album_ranking.rb
class AlbumRanking < ActiveRecord::Base

  def top_albums(n)
    $redis.zrevrange(albums_key, 0, n - 1)
  end

  def albums=(albums)
    $redis.multi do
      remove_all_albums
      albums.each {|a| add_album(a) }
    end
  end

  def add_album(album)
    $redis.zadd(albums_key, album.downloads_number, album.id)
  end

  def remove_album(album)
    $redis.zrem(albums_key, album.id)
  end

  def remove_all_albums
    $redis.del(albums_key)
  end

  def albums_key
    "album_ranking:#{id}:albums"
  end
end
```

Besides keeping track of sorted elements in a fast way, there are also many other useful ways in which redis could be used.
Here's a list of them with a few interesting related resources:

- Caching resources (in XML or JSON), [LRU caching](http://antirez.com/post/redis-as-LRU-cache.html).
- Having data which automatically expires. Another excellent feature needed for LRU caching.
- Queues. Highly used by the [resque gem](https://github.com/defunkt/resque/blob/master/lib/resque/queue.rb#L29)
- Keeping highly variable counters (page views, thumbs up, etc). Here's a [Flask snippet](http://flask.pocoo.org/snippets/71/).
- Working with large sets of data (where you don't want repeated elements).

Enjoy!

