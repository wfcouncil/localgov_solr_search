# Contents

 * Introduction
 * Requirements
 * Recommended modules
 * Installation
 * Configuration

## Introduction

The localgov_solr_search module configures a default site-search that works out
of the box with with a correctly configured Lando setup.

 * Creates a Search API server and index with minimal fulltext search fields.
 * Creates a search page view with an search bar block.
 * Creates view modes for search results and search result child pages.
 * Comes with Autosuggestions, Spellcheck suggestions and 'Did you mean?' functionality.

## Requirements

This module requires the following modules:

 * [Views](https://www.drupal.org/project/views)
 * [Search API](https://www.drupal.org/project/search_api)
 * [Search API Solr](https://www.drupal.org/project/search_api_solr)
 * [Search API Solr Autocomplete](https://www.drupal.org/project/search_api_solr_autocomplete)
 * [Search API Spellcheck](https://www.drupal.org/project/search_api_spellcheck)
 * LocalGov Core

## Recommended Modules

 * [Search Overrides](https://www.drupal.org/project/search_overrides):
  Allows individual nodes to be excluded and promoted for certain search queries
  and provides a drag and drop interface for doing so.

 * [Search API Exclude Entity](https://www.drupal.org/project/search_api_exclude_entity):
  Allows individual nodes to be excluded completely from the search index.

## Installation

### Local Lando setup

  - Add a solr service to your Landofile

  ```
  proxy:
    solr:
      - solr.lndo.site:8983

  ...

  services
    solr:
      type: solr:8
      portforward: true
      core: lando
      config:
        dir: solr-config
  ```
  - Create a solr-config directory at the project root

  - Ensure Search API Solr is installed

  - Copy all files from ```jump-start/solr{your-version-number}/config-set``` in
  the Search API Solr module to the newly created ```solr-config``` directory.

  - ```lando start or lando rebuild```

  - Install localgov_solr_search via Drush or the Drupal admin UI

### Setting up Solr (single core) - From the search_api_solr README

In order for this module to work, you need to set up a Solr server.
For this, you can either purchase a server from a web Solr hosts or set up your
own Solr server on your web server (if you have the necessary rights to do so).
If you want to use a hosted solution, a number of companies are listed on the
module's [project page](https://drupal.org/project/search_api_solr). Otherwise,
please follow the instructions in this section.

Note: A more detailed set of instructions is available at:
* https://lucene.apache.org/solr/guide/8_4/installing-solr.html
* https://lucene.apache.org/solr/guide/8_4/taking-solr-to-production.html
* https://lucene.apache.org/solr/guide/ - list of other version specific guides

As a pre-requisite for running your own Solr server, you'll need a Java JRE.

Download the latest version of Solr 8.x from
https://lucene.apache.org/solr/downloads.html and unpack the archive
somewhere outside of your web server's document tree. The unpacked Solr
directory is named `$SOLR` in these instructions.

Note: Solr 6.x is still supported by search_api_solr but strongly discouraged.
That version has been declared end-of-life by the Apache Solr project and is
thus no longer supported by them.

**_Before_** creating the Solr core (`$CORE`) you will have to make sure it uses
the proper configuration files. They aren't always static but vary on your
Drupal setup.

But the Search API Solr Search module will create the correct configs for you!

1. Make sure you have Apache Solr started and accessible (i.e. via port 8983).
   You can start it without having a core configured at this stage.
2. Visit Drupal configuration (/admin/config/search/search-api) and create a
   new Search API Server according to the search_api documentation using
   "Solr" as Backend and the connector that matches your setup.
   Input the correct core name (which you will create at step 4, below).
3. Download the config.zip from the server's details page or by using
   `drush solr-gsc` with proper options, for example for a server named
   "my_solr_server": `drush solr-gsc my_solr_server config.zip 8.4`.
4. Copy the config.zip to the Solr server and extract. The unpacked
   configuration directory is named `$CONF` in these instructions.

**_Now_** you can create a Solr core using this config-set on a running Solr
server. There're different ways to do so. For most Linux distributions you can
run
```
sudo -u solr $SOLR/bin/solr create_core -c $CORE -d $CONF -n $CORE
```

You will see something like
```
$ sudo -u solr /opt/solr/bin/solr create_core -c test-core -d /tmp/solr-conf -n test-core

Copying configuration to new core instance directory:
/var/solr/data/test-core
```

If you're forced to create the core before you can run Drupal to generate the
config-set you could also use the appropriate jump-start config-set you'll
find in the `jump-start` directory of this module.

**You must not create a core without a proper drupal config-set!**
If you do so - even by accident - you won't recognize it immediately. But you'll
run into trouble like this soon:
[SolrException: Can not use FieldCache on multivalued field: boost_document](https://www.drupal.org/project/search_api_solr/issues/3056971)

Note: Every time you add a new language to your Drupal instance or add a custom
Solr Field Type you have to update your core configuration files. Using the
example above they will be located in /var/solr/data/test-core/conf. The Drupal
admin UI should inform you about the requirement to update the  configuration.
Reload the core after updating the config using
`curl -k http://localhost:8983/solr/admin/cores?action=RELOAD&core=$CORE` on
the command line or enable the search_api_admin sub-module to do it from the
Drupal admin UI.

Note: There's file called `solrcore.properties` within the set of generated
config files. If you need to fine tune some setting you should do it within this
file if possible instead of modifying `solrconf.xml`.

Afterwards, go to `http://localhost:8983/solr/#/$CORE` in your web browser to
ensure Solr is running correctly.

CAUTION! For production sites, it is vital that you somehow prevent outside
access to the Solr server. Otherwise, attackers could read, corrupt or delete
all your indexed data. Using the server as described below WON'T prevent this by
default! If it is available, the probably easiest way of preventing this is to
disable outside access to the ports used by Solr through your server's network
configuration or through the use of a firewall.
Other options include adding basic HTTP authentication or renaming the solr/
directory to a random string of characters and using that as the path.

For configuring indexes and searches you have to follow the documentation of
search_api.

## Configuration

  - Go to /admin/config/search/search-api/server/localgov_solr/edit to
  change your Solr host and/or core name to match your Solr installation
  details. If using Lando, make sure to update your Landofile and rebuild.

  - Customise the index's fields, processors and autocomplete as desired.

  - Index your content at /admin/config/search/search-api/index/site_search or
  using ```drush sapi-i site-search```.

  - Customise the search view at /admin/structure/views/view/site_search.




