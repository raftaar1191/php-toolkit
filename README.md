## PHP Toolkit

**A collection of lightweight, dependency-free PHP libraries designed for WordPress and PHP projects.**

### Purpose

This toolkit provides standalone PHP components that work without requiring external PHP extensions (like libxml2, curl, or libzip). Each library is designed to be:
- ✅ **WordPress-first** but framework-agnostic
- ✅ **Dependency-free** – minimal external requirements
- ✅ **Reusable** across different environments (web, CLI, browser extensions)
- ✅ **Compatible** with PHP 7.2+ and major WordPress versions

### Available Components

| Component | Description | Key Benefit |
|-----------|-------------|-------------|
| **XMLProcessor** | Stream-parse XML files | No libxml2 required |
| **Git** | Pure PHP Git client and server | No system Git needed |
| **HttpClient** | Streaming, non-blocking HTTP client | No curl dependency |
| **Zip** | Stream-parse and stream-write ZIP files | No libzip dependency |
| **Data Liberation** | Streaming data importers for WordPress | Supports WXR, markdown, Git repos, URL rewriting |
| **ByteStream** | Composable byte streaming utilities | Readers, writers, and filters |
| **Markdown** | Convert between markdown and block markup | No dependencies |
| **Filesystem** | Unified API for multiple storage backends | Works with local files, Git, Google Drive, memory |
| **Blueprints** | WordPress site setup automation | Run Blueprints v1 and v2 |
| **HTML/BlockParser** | HTML and WordPress block parsing | Process HTML and blocks |
| **HttpServer** | HTTP server implementation | Build HTTP servers in PHP |

### Quick Start

#### Install via Composer

Install the entire toolkit:
```bash
composer require wp-php-toolkit/php-toolkit
```

Or install specific components you need:
```bash
composer require wp-php-toolkit/http-client
composer require wp-php-toolkit/data-liberation
composer require wp-php-toolkit/git
# ... and so on for other components
```

The individual components are distributed via Composer at [https://packagist.org/packages/wp-php-toolkit](https://packagist.org/packages/wp-php-toolkit).

#### Use the Blueprints CLI Tool

Download [blueprints.phar from the latest release](https://github.com/WordPress/php-toolkit/releases) and run:
```sh
php blueprints.phar
```

If you want to use Blueprints as a library, you absolutely can. It is designed to be reusable,
compatible with web and CLI environments on PHP 7.2+. There's not much technical documentation
at this point but you can refer to the [blueprints.php file](https://github.com/WordPress/php-toolkit/blob/219dc4e846af270a5009e523244d0ec23baaa32a/components/Blueprints/bin/blueprint.php#L226) to see
how the runner is implemented.

#### PHAR Distribution

For convenience, standalone tools from this repository (including the Blueprints runner) are shipped as phar files available in the [GitHub releases](https://github.com/WordPress/php-toolkit/releases).

---

This fork consolidates a few earlier projects and explorations into a single composer package.

### Who Should Use This?

- **WordPress Plugin Developers** – Build plugins without worrying about missing PHP extensions
- **PHP Developers** – Need lightweight, portable libraries that work across different hosting environments
- **WordPress Playground Users** – Run WordPress in browsers, CLIs, or desktop apps
- **CI/CD Pipeline Engineers** – Need reliable PHP tools without system dependencies

### Design goals

-   Build re-entrant data tools that can start, stop, resume, tolerate errors, accept alternative media files, posts etc. from the user.
-   WordPress-first – Everything is built in PHP using WordPress coding standards. The divergences are strategic and minimal, such as the use of namespaces.
-   Compatibility – Support for major WordPress versions, PHP version (7.2+), and Playground runtime (web, CLI, browser extension, desktop app, CI etc.).
-   Dependency-free – No PHP extensions are required and only minimal Composer dependencies are allowed when absolutely necessary.
-   Simple – The architectural role model is [WP_HTML_Processor](https://developer.wordpress.org/reference/classes/wp_html_processor/) – a **single class** that can parse nearly all HTML. There's no "Node", "Element", "Attribute" classes etc. Let's aim for the same here. Some OOP patterns are used when useful, but we're explicitly avoiding ideas like AbstractSingletonFactoryProxy.
-   Extensibility – Playground should be able to benefit from, say, WASM markdown parser even if core WordPress cannot.
-   Reusability – Each library should be framework-agnostic and usable outside of WordPress.

### Development

#### Testing

To run the PHPUnit test suite, run:

```sh
composer test
```

#### Linting

To run the PHP_CodeSniffer linting suite, run:

```sh
composer lint
```

To fix the linting errors, run:

```sh
composer lint-fix
```

#### Composer

The root composer.json file is an amalgamation of composer.base.json all
component composer.json files. To regenerate it, run:

```sh
bin/regenerate_composer.json.php
```

This will merge all the package-specific dependencies and the autoload rules into
the root composer.json file.

### Windows compatibility

Windows compatibility is achieved on a few different fronts:

#### Newlines

This repository comes with a `.gitattributes` file to ensure that the unit test
files and fixtures are normalized to `\n` on checkout. It's important, because
Windows uses `\r\n` for newlines in text files. Unix-based systems use `\n`.
Without the `.gitattributes`, git on Windows would replace all the `\n` with `\r\n` 
on checkout.

The strings produced by the library uses `\n` for newlines where it can make
that choice. For example, the `WXRWriter` class will separate XML tags with
`\n` newlines to make sure the generated XML is consistent across platforms.

#### Paths

The `Filesystem` components makes a point of using Unix-style forward slashes
as directory separators, even on Windows.

As a library consumer, ensure all the local paths you pass to the library are
using Unix-style forward slashes as directory separators. A simple str_replace
will do the trick:

```php
if (DIRECTORY_SEPARATOR === '\\') {
	$path = str_replace('\\', '/', $path);
}
```

The reason for using Unix-style forward slashes is care for data integrity.
Windows understands both forward slashes and backslashes, so the replacement
operation is safe there. On Unix, however, a backslash can be used as a part
of a filename so it cannot be safely translated.

Importantly, do not just run this str_replace() on every possible path.
`C:\my-dir\my-file.txt` is both, a valid Windows absolute path and a valid Unix
filename and a relative path. Furthermore, `Filesystem` supports more filesystems
than just local disk.

Anytime you're handling paths, consider:

-   Which filesystem is this path related to? Local? Remote? Git?
-   Which OS are you on? Windows? Unix?

If the answers are "local" and "Windows", you may need to apply the `str_replace()`
slash normalization. Otherwise, just keep the path as it is.

The takeaway from this section is: **paths are difficult**.

For a fun read on the topic, check out this article: [Windows File Paths](https://www.fileside.app/blog/2023-03-17_windows-file-paths/).
