## &#10162; Introduction:


### &#9780; Overview:
1. [Pre-requirements](#-pre-requirements)
2. [Installation](#-installation)
3. [Directory Structure](#-directory-structure)
4. [Post Setup](#-post-setup)
5. [Deployment](#-deployment)

### &#10022; Pre-Requirements:

*Summary:*

1. PHP (with required extensions)
2. Composer
3. Web Server
4. Database
5. Node.js/npm (for asset compilation)
6. Terminal/Command Line
7. Git (for version control)

*Requirements:*

1. PHP:

	- Laravel requires PHP. Laravel 8 requires PHP 7.3 or higher. PHP 8 is also supported.
	- Following PHP extensions should be enabled:
	  - BCMath PHP Extension
	  - Ctype PHP Extension
	  - Fileinfo PHP Extension
	  - JSON PHP Extension
	  - Mbstring PHP Extension
	  - OpenSSL PHP Extension
	  - PDO PHP Extension
	  - Tokenizer PHP Extension
	  - XML PHP Extension

2. Composer:

	- Composer is a dependency manager for PHP. Laravel uses Composer to manage its dependencies. Before install Laravel, need Composer installed in the system.

3. Web Server:

	- Require a web server to serve Laravel application. Commonly used development server is Apache (with mod\_rewrite enabled)

4. Database:
	- Most of the Laravel applications need a database to store and retrieve data. Ensure the database server installed and configured as per the requirement. 
	- Most common and supported databases are:
		- MySQL/MariaDB
		- PostgreSQL
		- SQLite
		- SQL Server

5. Node.js and npm (Optional but often needed):
	- If you need to use Laravel Mix for asset compilation (CSS, JavaScript), then it is require Node.js and npm (Node Package Manager).

6. Terminal/Command Line:
	- Familiarity with the command line is essential for running Composer commands, Artisan commands, and other development tasks.

7. Git (Optional but recommended):
	- Git is a version control system that is highly recommended for managing code versions.

### &#10022; Installation:

1. Via Composer Create-Project (Recommended):
	- This is the most common and recommended method.
	- Open terminal and navigate to the directory where it to be created Laravel project.
	- Run the following command:

		```bash
		composer create-project --prefer-dist laravel/laravel <project-name>
		```

	- Replace `<project-name>` with the desired name for your project.
	- Composer will download and install Laravel and its dependencies.

2. Install Specific Version Via Composer Create-Project:
	
	- It is possible to specify the desired Laravel version by adding a version constraint after `laravel/laravel`.

 	- For example, to install Laravel 8.83.0, you would run:

	  ```bash
	  composer create-project --prefer-dist laravel/laravel:v8.83.0 your-project-name
	  ```

	- For example, to install the latest Laravel 8 version, you can use:

	  ```bash
	  composer create-project --prefer-dist laravel/laravel:"8.*" your-project-name
	  ```

	- If you need a specific version range, then use standard Composer version constraints. For example, to install any version within the 8.x range:

	  ```bash
	  composer create-project --prefer-dist laravel/laravel:"^8.0" your-project-name
	  ```
  	- Where, the '^' symbol means, install the latest minor or patch release of version 8.

3. Via Laravel Installer (Less common for specific version):

	- First, it is required to install the Laravel installer globally via Composer:

		 ```bash
		 composer global require laravel/installer
		 ```

	- Then, decide and navigate to the directory where you want to create Laravel project, and run below command:

		 ```bash
		 laravel new <project-name>
		 ```

	- This will create a new Laravel project in the specified directory.

4. Checking the Installed Version:

	- After the installation is complete, To check the installed Laravel version by navigating to installed project directory and running below artisan command:

	  ```bash
	  php artisan --version
	  ```

**Note:**

- To install via **docker container**, check official documentation page.
- Ensure that the Laravel version must compatible with PHP version. Refer to the official Laravel documentation for version compatibility information.
- Always recommended to refer official Laravel documentation for the most up-to-date installation instructions and version compatibility information.
- The laravel installer global command always installs the latest version. To use composer create-project is the best method to use specific versions.

### &#10022; Directory Structure:

After installation, that contains files and folders as below. *It is not complete but an overview.*

```
<project-name>/
├── app/
│   ├── Console/
│   ├── Exceptions/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   ├── Models/
│   ├── Providers/
├── bootstrap/
│   ├── cache/
├── config/
├── database/
│   ├── factories/
│   ├── migrations/
│   ├── seeders/
├── public/
├── resources/
│   ├── css/
│   ├── js/
│   ├── lang/
│   ├── views/
├── routes/
│   ├── api.php
│   ├── channels.php
│   ├── console.php
│   ├── web.php
├── storage/
│   ├── app/
│   ├── framework/
│   ├── logs/
├── tests/
│   ├── Feature/
│   ├── Unit/
├── vendor/
├── .env
├── artisan
├── composer.json
├── composer.lock
├── package.json
├── package-lock.json
├── phpunit.xml
├── server.php
```

**Explanation:**

- `app`: it is core the application code, including models, controllers, middleware, and providers.
- `bootstrap`: it contains bootstrapping files, including the cache directory.
- `config`: it contains all configuration files for the project application.
- `database`: it contains database migration, factory and seeder files.
- `public`: it contains the public-facing files of the application such as `index.php`, `.htaccess` and assets.
- `resources`: it contains application assets such as views, CSS, JavaScript, and language files.
- `routes`: it contains all of the application route definitions.
- `storage`: it contains files generated and uploaded in the application such as logs, cached files, and user-uploaded files.
- `tests`: it contains application automated tests.
- `vendor`: it contains the composer dependencies of the application.
- `.env`: it contains environment variables for the application.
- `artisan`: It is an Artisan command-line tool.
- `composer.json` & `composer.lock`: those are composer dependency management files.
- `package.json` & `package-lock.json`: those are Node.js dependency management files.
- `phpunit.xml`: it is a PHPUnit configuration file.
- `server.php`: it is ab uilt in php server file.

### &#10022; Post Setup:

1. Navigate to the Project Directory:

	 ```bash
	 cd <project-name>
	 ```

2. Check installed version:

	- After the installation is complete, check Laravel version for ensure, it is installed properly:

	  ```bash
	  php artisan --version
	  ```

3. Configure Environment Variables:

	- Copy the `.env.example` file to `.env`:

		 ```bash
		 cp .env.example .env
		 ```

	- Open the `.env` file and configure required database settings,generate application key, and set other environment variables.

4. Generate the application key:

	 ```bash
	 php artisan key:generate
	 ```

5. Install Node Dependencies (If using Laravel Mix):

	 ```bash
	 npm install
	 ```

6. Run the Development Server (Optional):

	 ```bash
	 php artisan serve
	 ```

	- This will start Laravel's built-in development server.

	**Note:**
	
	- When serve with **docker container**, refer to the official Laravel documentation page.

7. Database Migrations (Optional but usually needed):

	 ```bash
	 php artisan migrate
	 ```

	- This will run your database migrations.

	**Note:**
	
	- This will migrate some tables into the database which is configured before in `.env` file.

### &#10022; Deployment:

---
[&#8682; To Top](#-introduction)

[&#10094; Previous Topic](../README.md) &emsp; [Next Topic &#10095;](./artisan-commands.md)

[&#8962; Goto Home Page](../README.md)