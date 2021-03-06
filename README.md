# NAME

MojoX::Plugin::AnyCache - Cache plugin with blocking and non-blocking support

# SYNOPSIS

    $app->plugin('MojoX::Plugin::AnyCache' => {
      backend => 'MojoX::Plugin::AnyCache::Backend::Redis',
      server => '127.0.0.1:6379',
    });

    # For synchronous backends (blocking)
    $app->cache->set('key', 'value');
    my $value = $app->cache->get('key');

    # For asynchronous backends (non-blocking)
    $app->cache->set('key', 'value' => sub {
      # ...
    });
    $app->cache->get('key' => sub {
      my $value = shift;
      # ...
    });

# DESCRIPTION

MojoX::Plugin::AnyCache provides an interface to both blocking and non-blocking
caching backends, for example Redis or Memcached.

It also has a built-in replicator backend ([MojoX::Plugin::AnyCache::Backend::Replicator](https://metacpan.org/pod/MojoX::Plugin::AnyCache::Backend::Replicator))
which automatically replicates values across multiple backend cache nodes.

## SERIALISATION

The cache backend module supports an optional serialiser module.

    $app->plugin('MojoX::Plugin::AnyCache' => {
      backend => 'MojoX::Plugin::AnyCache::Backend::Redis',
      server => '127.0.0.1:6379',
      serialiser => 'MojoX::Plugin::AnyCache::Serialiser::MessagePack'
    });

#### SERIALISER WARNING

If you use a serialiser, `incr` or `decr` a value, then retrieve
the value using `get`, the value returned is deserialised.

With the FakeSerialiser used in tests, this means `1` is translated to an `A`.

This 'bug' can be avoided by reading the value from the cache backend
directly, bypassing the backend serialiser:

    $self->cache->set('foo', 1);
    $self->cache->backend->get('foo');

## TTL / EXPIRES

### Redis

Full TTL support is available with a Redis backend. Pass the TTL (in seconds)
to the `set` method.

    $cache->set("key", "value", 10);

    $cache->set("key", "value", 10, sub {
      # ...
    });

And to get the TTL (seconds remaining until expiry)

    my $ttl = $cache->ttl("key");

    $cache->ttl("key", sub {
      my ($ttl) = @_;
      # ...
    });

### Memcached

Full TTL set support is available with a Memcached backend. Pass the TTL (in seconds)
to the `set` method.

    $cache->set("key", "value", 10);

    $cache->set("key", "value", 10, sub {
      # ...
    });

Unlike a Redis backend, 'get' TTL mode in Memcached is emulated, and the time
remaining is calculated using timestamps, and stored in a separate prefixed key.

To enable this, set `get_ttl_support` on the backend:

    $cache->backend->get_ttl_support(1);

This must be done before setting a value. You can then get the TTL as normal:

    my $ttl = $cache->ttl("key");

    $cache->ttl("key", sub {
      my ($ttl) = @_;
      # ...
    });
