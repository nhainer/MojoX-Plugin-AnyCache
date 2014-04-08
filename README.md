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