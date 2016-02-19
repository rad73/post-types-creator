# Post types creator
[![Build Status](https://travis-ci.org/travis-ci/travis-web.svg?branch=master)](https://travis-ci.org/travis-ci/travis-web)

Helper plugin that provides an easy interface for creating fully translated custom post types and taxonomies according to best practice with only a few lines of code for WordPress.

<!-- Generated by doctoc (https://github.com/thlorenz/doctoc) -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Contents**

- [Features](#features)
- [Planned features](#planned-features)
- [Common problems & troubleshooting](#common-problems-&-troubleshooting)
- [Installation](#installation)
  - [Composer](#composer)
  - [Normal manual install](#normal-manual-install)
  - [Use the example plugin as a boilderplate for your custom post type plugin](#use-the-example-plugin-as-a-boilderplate-for-your-custom-post-type-plugin)
- [Usage](#usage)
  - [Adding custom post types](#adding-custom-post-types)
    - [Minimal:](#minimal)
    - [Example / typical:](#example--typical)
  - [Adding taxonomies:](#adding-taxonomies)
  - [More examples / Example plugin](#more-examples--example-plugin)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

##Features
- Easy interface, just a few lines of code requiered.
- Flexible. Does not restrict anything, you can pass any arguments to ` register_post_type() ` and ` register_taxonomy() ` by simply specifying them in the normal way and they will override the defaults. 
- Translation ready (All the labels in WP Admin will be translated and you only need to specify singular and plural form of the post type name). Translation currently supports the following languages:
  - English
  - Swedish
  - Norwegian
  - Plase help me to add more! 
- Translated permalinks/slugs for post types and taxonomies are generated automatically, but it is possible to override this deafault. 
- Custom columns in admin with only a few lines of code
- easily add custom post statuses
- Drag & drop sortable (drag and drop in the normal list view in WP Admin for both posts and taxonomies/terms)
- Integration with Advanced Custom Fields(optional)

##Planned features
- Unit tests
- Use new merm meta functionality for sorting for WordPress 4.4+

##Common problems & troubleshooting

The plugin is very simple to use if you know basic PHP. Most of the problems you are likely to run into is related to WordPress built in permalink cache or messed up or non-existing meta data for sorting(the posts will not show up in the post linst in wp-admin). To fix both those problems. Just add `&ptc-reinit=1` to the URL/querystring anywhere in wp-admin to run a force reinitialization that will fix the problems, for example: `http://example.com/wp-admin/edit.php?post_type=page&ptc-reinit=1`

##Installation

###Composer
Add the repository and add ` pelmered/post-types-creator ` to the require section in your composer.json. Example of typical full composer.json file:

```
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/pelmered/post-types-creator"
    }
  ],
  "require": {
    "pelmered/post-types-creator": "dev-master"
  },
  "extra": {
    "wordpress-install-dir": "public/wp",
    "installer-paths": {
        "public/wp-content/plugins/{$name}/": ["type:wordpress-plugin"]
    }
  }
}
```
Note: Edit the installer paths to reflect your installation

###Normal manual install
First, install the plugin as usnual by uploading the plugin to you plugins folder, typically ` wp-content/plugins/  `.

###Use the example plugin as a boilderplate for your custom post type plugin
Secondly, copy the example plugin from ` example-plugin/my-custom-post-types/ ` in this plugin to your plugins folder, typically ` wp-content/plugins/ ` or install the example plugin from the zip file in  ` example-plugin/my-custom-post-types.zip `. Change the name, description etc of the plugin and edit the data acording to your needs.

##Usage

###Adding custom post types

####Minimal:
Registers a post type with the slug ` stores ` and the labels tranlatable based on ` Post type plural ` (plural) and ` Post type singular ` (singular).

```php
$options = array(
  // Use ACF for storing meta data
  'use_acf' => true
);

$ptc = new Post_Type_Creator();
$text_domain = 'text-domain';
        
$ptc->set_post_types(array(
    'stores' => array(
        'singular_label' => _x('store', 'Post type singular', $text_domain),
        'plural_label'  => _x('stores', 'Post type plural', $text_domain)
    )
));

add_action( 'init', array($ptc, 'init'), 0 );
```

#####Example / typical:
Same as minimal, but allso adds a description, makes it drag and drop sortable in the admin list, adds a custom admin column and overides some ` register_post_type() ` defaults, for example connecting the taxonomy ` area `(see example below).

```php
$ptc = new Post_Type_Creator();
$text_domain = 'text-domain';
        
$ptc->set_post_types(array(
    'stores' => array(
        'singular_label' => _x('store', 'Post type singular', $text_domain),
        'plural_label'  => _x('stores', 'Post type plural', $text_domain),
        'description'   => _x('All company stores', 'Post type description', $text_domain),
        
        // Make post type drag and drop sortable in admin list view (default: false)
        'sortable'      => true,
        'taxonomy_filters' => true,
        'admin_columns' => array(
            'image' => array(
                //Column header/label
                'label' => 'Image',
                //In what position should the column be (optional)
                'location'  => 2,
                //callback for column content. Arguments: $post_id
                'cb'    => 'example_get_featured_image_column' 
            )
        )
        
        // Override any defaults from register_post_type()
        // http://codex.wordpress.org/Function_Reference/register_post_type
        'supports'            => array( 'title', 'editor', 'thumbnail',),
        'taxonomies'          => array( 'area' ),
    )
));

add_action( 'init', array($ptc, 'init'), 0 );

function example_get_featured_image_column( $post_id )
{
    echo get_the_post_thumbnail( $post_id, 'thumbnail' );
}
```

####Adding taxonomies:
Typical taxonomy that is drag and drop sortable in the normal admin list view and connected to the ` stores ` post type in the example above.

```php
$ptc = new Post_Type_Creator();
$text_domain = 'text-domain';

$ptc->set_taxonomies(array(
    'area' => array(
        'singular_label' => _x('area', 'Post type singular', $text_domain),
        'plural_label'  => _x('areas', 'Post type plural', $text_domain),
        'description'   => _x('Areas for grouping stores', 'Post type description', $text_domain),
        'post_type'    => 'stores',
        
        // Make post type drag and drop sortable in admin list view (default: false). Affects all get_terms()-queries
        'sortable'      => true,
        
        // Override any defaults from register_taxonomy()
        // http://codex.wordpress.org/Function_Reference/register_taxonomy
        
    )
));

add_action( 'init', array($ptc, 'init'), 0 );
```

##### For more examples, or help to get started see the example plugin in ` example/example-plugin.php `. Copy the example plugin to your plugins directory for the fastest way to get started.


##Documentation

### Adding features to post types from core or other plugins

Example for adding sortable to WooCommerce product categories

```php
$ptc->set_taxonomies(array(
    'product_cat' => array(
        'register'      => false,
        'post_type'     => array( 'product' ),
        'sortable'      => true,
    ),
));
```


### Labels & description

You only need to send in a singular and a plural version of the post type label, all other labels are automatically generated.

If you want to override something, you can hook into the filter `pe_ptc_post_type_labels` like this:

```php
add_filter( 'pe_ptc_post_type_labels', 
	function( $generated_args, $post_type_slug, $post_type_args ) {
		
		// do what you want with $generated_args
		//And then return it back to the plugin
		return $generated_args;
	}, 
10, 3)
```


```php
$post_type['singular_label_ucf'] = ucfirst($post_type['singular_label']);
$post_type['plural_label_ucf'] = ucfirst($post_type['plural_label']);

$generated_args = array(
    'label'               => __( $post_slug, $this->text_domain ),
    'description'         => __( $post_type['plural_label_ucf'], $this->text_domain ),
    'labels'              => array(
        'name'                  => _x( $post_type['plural_label_ucf'], 'Post Type General Name', $this->text_domain ),
        'singular_name'         => _x( $post_type['singular_label_ucf'], 'Post Type Singular Name', $this->text_domain ),
        'menu_name'             => __( $post_type['plural_label_ucf'], $this->text_domain ),
        'parent'                => sprintf(__( 'Parent %s', $this->text_domain ), $post_type['singular_label']),
        //'parent_item_colon'     => sprintf(__( 'Parent %s:', $this->text_domain ), $post_type['singular_label']),
        'all_items'             => sprintf(__( 'All %s', $this->text_domain ), $post_type['plural_label']),
        'view'                  => sprintf(__( 'View %s', $this->text_domain ), $post_type['singular_label']),
        'view_item'             => sprintf(__( 'View %s', $this->text_domain ), $post_type['singular_label']),
        'add_new'               => sprintf(__( 'Add %s', $this->text_domain ), $post_type['singular_label']),
        'add_new_item'          => sprintf(__( 'Add new %s', $this->text_domain ), $post_type['singular_label']),
        'edit'                  => __( 'Edit', $this->text_domain ),
        'edit_item'             => sprintf(__( 'Edit %s', $this->text_domain ), $post_type['singular_label']),
        'update_item'           => sprintf(__( 'Update %s', $this->text_domain ), $post_type['singular_label']),
        'search_items'          => sprintf( __('Search %s', $this->text_domain), $post_type['plural_label']),
        'not_found'             => sprintf(__( 'No %s found', $this->text_domain ), $post_type['plural_label']),
        'not_found_in_trash'    => sprintf(__( 'No %s found in trash', $this->text_domain ), $post_type['plural_label']),
    ),
);
```

### Extra arguments

#### admin_columns

#### sortable

#### Taxonomy filters

Pass `true` for making filters of all taxonomies for the post type

            'taxonomy_filters'               => false,

### Overriding `register_post_status()` defaults

All the defaults and arguments that you can pass in to [`register_post_status()`](https://codex.wordpress.org/Function_Reference/register_post_status)

