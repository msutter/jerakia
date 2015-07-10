### Datasources ###

## File ##

# Description #

The file data source performs hierarchical searches across YAML or JSON formatted files and searches for the lookup key

# Options #

* `:format`  The file format, one of :yaml and :json
* `:docroot` The root of the documents
* `:searchpath` An array specifying the order of hierarchical lookups

# Example #

    lookup :foo do
      datasource :file, {
        :format => :yaml,
        :docroot => "/etc/jacaranda/data",
        :searchpath => [
          scope[:certname],
          scope[:environment],
          'global'
        ]
      }
    end

# File system layout #

Jacaranda requests consist of a lookup key and an array representing the namespace.  By default the file data source will look for the lookup key in a filename based upon the `docroot`, `searchpath` and `namespace`.  For example if we use the above lookup directive with the following request:

    {
      :key => "port",
      :namespace => [ "apache" ],
    }

Assuming the scope has `certname => foo.acme.com` and `environment => development` then Jacaranda will search for the key `port` in the followint order

* `/etc/jacaranda/data/foo.acme.com/apache.yml`
* `/etc/jacaranda/data/development/apache.yml`
* `/etc/jacaranda/data/global/apache.yml`

More than one element used in the namespace will just expand the filesystem hierarchy, for example `:namespace => [ "profile", "web" ]` will result in something like `/etc/jacaranda/data/global/profile/web.yml`

# hiera_compat #

Hiera has a slightly different way of working, as the file name is made out of the scope and hierarchy, with namespacing implied in the look up key.  In Hiera for example we would store the key `apache::port` in a file called `global.yml`.  This default behaviour can lead to very large YAML files in larger environments, hence the decision to change this convention in Jacaranda.  However, for users wishing to maintain their Hiera filesystem layout there is a lookup plugin called _hiera_compat_ which will perform rewrites on the request object on the fly to provide compatibility to Hiera.

So, if we use this lookup block

    lookup :foo do
      datasource :file, {
        :format => :yaml,
        :docroot => "/etc/jacaranda/data",
        :searchpath => [
          scope[:certname],
          scope[:environment],
          'global'
        ]
       hiera_compat
      }
    end

When the following request is recieved

    {
      :key => "port",
      :namespace => [ "apache" ],
    }

The lookup will get automatically rewritten to be

    {
      :key => "apache::port",
      :namespace => '',
    }

This will change the behaviour of Jacaranda, which will now look for the key `apache::port` in the following files 

* `/etc/jacaranda/data/foo.acme.com.yml`
* `/etc/jacaranda/data/development.yml`
* `/etc/jacaranda/data/global.yml`

Unless you are migrating or sharing data with Hiera, it's recommended to use the default filesystem layout.
