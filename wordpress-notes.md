# [Theme Handbook](https://developer.wordpress.org/themes/)

Themes take the content and data stored by WordPress and display it in the browser.

A block theme is composed entirely of blocks. [Block markup](https://developer.wordpress.org/themes/block-themes/templates-and-template-parts/#block-markup) is the HTML code that WordPress uses to display the block. 

A block is added to a template with an HTML comment that includes a prefix, the block name, and optional attributes and supports.

Block attributes are stored inside a JSON object inside the block comment. Settings for the [attributes](https://fullsiteediting.com/lessons/block-grammar-basics/#h-attributes) are created in the block’s editor.js file using basic components like the toggle control.

[Block supports](https://fullsiteediting.com/lessons/block-grammar-basics/#h-supports) are settings built to be reused by many blocks. The blocks can then register if they want full or partial support and if they want the control to be visible by default in the editors.

[The easiest way to get](https://fullsiteediting.com/lessons/block-grammar-basics/#how-to-find-the-block-markup) the HTML code for a block is to add the block in the editor and copy the code from the code editor mode (⇧⌥⌘M).

## [Local environment setup](https://developer.wordpress.org/themes/getting-started/setting-up-a-development-environment/#your-wordpress-local-development-environment)

For developing WordPress themes, you need to set up a development environment suited to WordPress.

You will need:
- a local server stack: Docker and, 
- a text editor: VSCode

### [@wordpress/env](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/)

`wp-env` lets you easily set up a local WordPress environment for building and testing plugins and themes. It’s simple to install and requires no configuration.

Ensure that Docker is running, then:

```shell
$ cd /path/to/a/wordpress/theme
$ npm -g i @wordpress/env
$ wp-env start
```

The local environment will be available at `http://localhost:8888` (Username: **admin**, Password: **password**).

### [WP_DEBUG PHP](https://developer.wordpress.org/themes/getting-started/setting-up-a-development-environment/#wp-debug)

When developing your theme.json file, you can disable cache to see your changes applied faster.

To enable this functionality set WP_DEBUG to true within your *wp-config.php* file.


```php
define( 'WP_DISABLE_FATAL_ERROR_HANDLER', true ); // 5.2 and later
define( 'WP_DEBUG', true );
```

### [Theme Test Data](https://codex.wordpress.org/Theme_Unit_Test)

Theme testing correlates to a WordPress export (WXR) file that you can import into a WordPress installation to test your Theme.

### [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/)
These guidelines are the “gold standard” for quality theme development.


## [File structure](https://developer.wordpress.org/themes/block-themes/block-theme-setup/#file-structure)

Block themes require a **style.css** file and a **templates/index.html** file. 


```
assets (dir)
      - css (dir)
      - images (dir)
      - js (dir)
patterns (dir)
      - example.php
parts (dir)
      - header.html
      - footer.html
      - sidebar.html
templates(dir)
      - index.html (required)
      - front-page.html 
      - singular.html
      - archive.html
      - 404.html
functions.php
index.php
README.txt
screenshot.png
style.css (required)
theme.json
```

## [Main stylesheet](https://developer.wordpress.org/themes/basics/main-stylesheet-style-css/)

In order for WordPress to recognize the set of theme template files as a valid theme, the **style.css** file needs to be located in the root directory of your theme.

WordPress uses the header comment section of a style.css to display information about the theme in the Appearance ->  Themes dashboard panel.

```css
/*
Theme Name: Twenty Twenty
Theme URI: https://wordpress.org/themes/twentytwenty/
Author: the WordPress team
Author URI: https://wordpress.org/
Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor. Organizations and businesses have the ability to create dynamic landing pages with endless layouts using the group and column blocks. The centered content column and fine-tuned typography also makes it perfect for traditional blogs. Complete editor styles give you a good idea of what your content will look like, even before you publish. You can give your site a personal touch by changing the background colors and the accent color in the Customizer. The colors of all elements on your site are automatically calculated based on the colors you pick, ensuring a high, accessible color contrast for your visitors.
Tags: blog, one-column, custom-background, custom-colors, custom-logo, custom-menu, editor-style, featured-images, footer-widgets, full-width-template, rtl-language-support, sticky-post, theme-options, threaded-comments, translation-ready, block-styles, wide-blocks, accessibility-ready
Version: 1.3
Requires at least: 5.0
Tested up to: 5.4
Requires PHP: 7.0
License: GNU General Public License v2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html
Text Domain: twentytwenty
This theme, like WordPress, is licensed under the GPL.
Use it to make something cool, have fun, and share what you've learned with others.
*/
```

## [Theme Functions](https://developer.wordpress.org/themes/basics/theme-functions/)

The **functions.php** file is where you add unique features to your WordPress theme. It can be used to hook into the core functions of WordPress to make your theme more modular, extensible, and functional.

It’s important to namespace your functions with your theme name.

### [Block Theme setup](https://developer.wordpress.org/themes/block-themes/block-theme-setup/#theme-setup-function)

The block theme setup function is hooked into the after_setup_theme hook, which runs before the init hook.

```php
if ( ! function_exists( 'myfirsttheme_setup' ) ) :

function myfirsttheme_setup() {
	// Add support for block styles.
	add_theme_support( 'wp-block-styles' );

	// Enqueue editor styles.
	add_editor_style( 'editor-style.css' );
}
endif;
add_action( 'after_setup_theme', 'myfirsttheme_setup' );
```

#### [Include CSS for block styles](https://developer.wordpress.org/themes/block-themes/block-theme-setup/#including-css-for-block-styles)

```php
function myfirsttheme_setup() {
	/*
	 * Load additional block styles.
	 */
	$styled_blocks = ['latest-comments'];
	foreach ( $styled_blocks as $block_name ) {
		$args = array(
			'handle' => "myfirsttheme-$block_name",
			'src'    => get_theme_file_uri( "assets/css/blocks/$block_name.css" ),
			$args['path'] = get_theme_file_path( "assets/css/blocks/$block_name.css" ),
		);
		wp_enqueue_block_style( "core/$block_name", $args );
	}
}
add_action( 'after_setup_theme', 'myfirsttheme_setup' );
```

#### [Enqueuing Scripts and Styles](https://developer.wordpress.org/themes/basics/including-css-javascript/#enqueuing-scripts-and-styles)

The proper way to add scripts and styles to your theme is to enqueue them in the **functions.php** files.

It is best to combine all enqueued scripts and styles into a single function, and then call them using the wp_enqueue_scripts action. This function and action should be located somewhere below the initial setup.

```php
function add_theme_scripts() {
	wp_enqueue_style( 'style', get_stylesheet_uri() );

	wp_enqueue_style( 'slider', get_template_directory_uri() . '/css/slider.css', array(), '1.1', 'all' );

	wp_enqueue_script( 'script', get_template_directory_uri() . '/js/script.js', array( 'jquery' ), 1.1, true );
}
add_action( 'wp_enqueue_scripts', 'add_theme_scripts' );
```



## [Template Files](https://developer.wordpress.org/themes/block-themes/templates-and-template-parts/)

In block themes, templates are HTML files that contain HTML markup representing blocks.

The [Template Hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/#the-template-file-hierarchy) is the logic WordPress uses to decide which theme template file to use, depending on the URL being requested. 

WordPress matches every query string to a query type to decide which page is being requested.

[Common template files](https://developer.wordpress.org/themes/basics/template-files/#common-wordpress-template-files)

## [Templates are loaded in the body element](https://fullsiteediting.com/lessons/creating-block-based-themes/#h-templates-are-loaded-in-the-body-element)

```html
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
	<meta charset="<?php bloginfo( 'charset' ); ?>" />
	<?php wp_head(); ?>
</head>

<body <?php body_class(); ?>>
<?php wp_body_open(); ?>

(template)

<?php wp_footer(); ?>
</body>
</html>
```

On the front, your templates and all your blocks are loaded in the `<body>` element, inside a `<div>` with the class wp-site-blocks.

```html
<body>
<div class="wp-site-blocks">
...
</div>
</body>
```

### [Template Parts](https://developer.wordpress.org/themes/basics/template-files/#template-partials) 
Template parts are used to organize a theme in smaller reusable structural parts. A template part can be embedded in multiple templates

### [Post Types](https://developer.wordpress.org/themes/basics/post-types)
All of the Post Types are stored in the wp_posts database table — but are differentiated by a database column called post_type.

As the whole purpose of a Template file is to display content a certain way, the Post Types purpose is to categorize what type of content you are dealing with.

[Default post types](https://developer.wordpress.org/themes/basics/post-types/#default-post-types)

[Custom Post Types](https://developer.wordpress.org/themes/basics/post-types/#custom-post-types) should be created in a plugin. This ensures the portability of your user’s content, and that if the theme is changed the content stored in the Custom Post Types won’t disappear.

## [Taxonomies](https://developer.wordpress.org/themes/basics/categories-tags-custom-taxonomies/)

Taxonomies are the method of classifying content and data in WordPress. 

The default taxonomies are categories, tags and post formats.

[Terms](https://developer.wordpress.org/themes/basics/categories-tags-custom-taxonomies/#terms) are items within your taxonomy. Terms are stored in the wp_terms database table.

[Custom Taxonomies](https://developer.wordpress.org/themes/basics/categories-tags-custom-taxonomies/#custom-taxonomies) should be created in a plugin.

The wp_term_relationships database table relates the term taxonomy to an object ID.

## [Theme .json](https://developer.wordpress.org/themes/advanced-topics/theme-json/)

**theme.json** is a configuration file for theme styles and block settings.

A [JSON scheme](https://developer.wordpress.org/block-editor/how-to-guides/themes/theme-json/#developing-with-theme-json) is useful to remember the theme.json settings and properties while you develop.

To use the schema in Visual Studio Code, add `"$schema": https://schemas.wp.org/trunk/theme.json"` to the beginning of your theme.json file.

There are two main sections:
- **Settings**, where you define your block controls and color palettes, font sizes, and more. Default settings apply to all blocks, but you can override them per block.
- **Styles**, where you apply these colors and font sizes to the website and blocks.

You can now add default styles for all blocks, including third-party blocks from plugins. 

A block must register support for a feature for the control to be visible in the editors. Exising control ca be disable in *theme.json*.

### [Preset Values](https://developer.wordpress.org/themes/advanced-topics/theme-json/#preset-values)

WordPress uses data from *theme.json* to control the editor’s block settings and create CSS custom properties.

WordPress parses the JSON and creates a CSS custom property for each preset value, and adds them to the `<body>` element.

```css
--wp--preset--{preset-category}--{preset-slug}
```

You can define presets for both the website and per block type.

### [Custom values](https://developer.wordpress.org/themes/advanced-topics/theme-json/#custom-values)

Values stored in the *settings.custom* area generates CSS properties that you can use elsewhere in the theme.json file or the theme’s CSS.



## [Tools](https://developer.wordpress.org/themes/basics/tools-resources/#tools)

### [Theme Tutorials](https://developer.wordpress.org/themes/basics/tools-resources/#training-material)

### [Developer Resources](https://developer.wordpress.org/)
- [Core Blocks Reference](https://developer.wordpress.org/block-editor/reference-guides/core-blocks/)
- [Carolina Nymark Blocks Reference](https://fullsiteediting.com/block-reference/)
- [Theme.json Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/)
- [Block Patterns Directory](https://wordpress.org/patterns/)
- [Block Theme Generator](https://fullsiteediting.com/block-theme-generator/)
- [Theme Experiments](https://github.com/WordPress/theme-experiments#generating-your-own-starter-theme)

#### Woocommerce Resources
- [WooCommerce Blocks Handbook](https://github.com/woocommerce/woocommerce-blocks/tree/trunk/docs#woocommerce-blocks-handbook)
- [WooCommerce Action and Filter Hook Reference](https://woocommerce.github.io/code-reference/hooks/hooks.html)
