# Support

If you need help or encounter any problems with Shortnr, shout out to [@felix_mc](http://twitter.com/#!/felix_mc) on Twitter.

# Installation

### 1. Get the code

	git clone git://github.com/felixmc/shortnr.git

### 2. Install Modules

	cd shortnr
	npm install

### 3. Configure Shortnr

Edit the `config.js` file and enter your database configuration as well as well as specify the length of the URLs the service will generate as this will affect the database setup.

Make sure that you already created the MySQL database you plan on using with Shortnr.

### 4. Generate Database Tables

The following will create the appropriate tables in your database based on your configuration. 

	npm db_config

If you change the "CODE_LENGTH" parameter of your configuration, you will need to run this again. Doing so might delete any data you had stored in your Shortnr database.

### 5. Running Shortnr

Finally, you can run Shortnr like any other node.js app:

	node app

If you would like to run Shortnr as a daemon on your server, take a look at [forever](https://github.com/nodejitsu/forever).

### 6. Using Nginx as reverse proxy (optional)

If you'd like to use Nginx as a reverse proxy for your Shortnr service, a sample configuration file is provided in extras/nginx.conf.

Simply replace `3000` and `example.com` with your own values and place the file inside your nginx conf directory.


# Configuration

The information below is supposed to help you when setting up your config.js file.

The file itself is thoroughly documented and contains most of the same info.

### Service

- The `database` property should be self-explanatory. You can use the `tables` property to customize the name of the tables in which different things will be stored.

- The `PORT` property determines what port the service will listen to.

- `BASE_URL` is the web address to the service and will be used when generating shortened URLs.

- `SHOW_STATIC_PAGE` determines whether a static website should be loaded at the root ("/") of the service. This allows you to setup a home page for the service.

- `STATIC_LOCATION` is the location the service will return to the client when accessing "/" if the `SHOW_STATIC_PAGE` property is set to `true`. Rather than redirecting the uesr, it will act as a proxy, loading the static location at the root ("/") of the service. This was designed to be used with an AWS S3-like static site.

- `ENABLE_STATS` determines whether clients can access service and URL statistics via "/stats/" and "/stats/:urlCode".

- `CONSOLE_LOG_LEVEL` configures what is outputted to the console. It uses the [Custom-Logger](https://github.com/felixmc/custom-logger) module internally.
	- 0 => All actions are logged.
	- 1 => Only Warnings and Errors are logged.
	- 2 => Only Errors are logged.
	- 3 => Nothing is logged.

### URL and URL Code

The URL code is the unique identifier that is associated with a URL in the database. It is the part that shows up after the domain in a shortened URL. For example, in the shortened URL "http://fmc.io/abcde", "abcde" is the URL code.

- `CODE_LENGTH` specifies how many character the URL code should be. This property should be filled out before running `npm db_setup` as it will affect the length of certain table fields.

- `CODE_CHARACTERS` specifies what characters Shortnr should use when generating a new URL code. This probably won't be edited too much.

- `CODE_PATTERN` specifies how URL codes are identified. It is *very* important that the Regex pattern assigned to this property reflex the values in your `CODE_LENGTH` and `CODE_CHARACTERS` properties. Currently there is no way of generating this automatically. There's a plan to fix this in the future.

- `URL_PATTERN` specifies a Regex pattern for validating URLs before being accepted by the service. The default Regex is pretty robust, however you can edit this property if you'd like to specify your service to accept only certain types of URLs, such as images maybe, or to restrict the domain of the allowed URLs.

### Restrictions

For the purpose of this service, a client is someone using the API. Clients are distinguished by their IP address. This might change in future versions.

- `ALLOW_SHORT_URLS` determines whether the service will allow already short URLs to be shortened by the service. Setting this value to `false` might be useful in preventing spam by double-shortening the URL in order to conceal its final destination.

- `limits` determines how many times a client can query to shorten a URL within a specif timeframe (minute, hour, day). This is useful for preventing spam or DDOS attacks against your Shortnr service. Setting any of the `limits` properties to `0` will disable that limit.

- `STRICT_LIMITS` if set to true, the service will count both successful and unsuccessful attempts to shorten a URL against the client's limits. Unsuccessful attempts include attempts to insert already short URLs, invalid URLs, and requests with an unproperly formatted body. Any request that does not return a 200 or 201 status code is considered unsucessful.

- `WHITELIST` specifies a file path containing a list of IP addresses separated on different lines. If enabled, the whitelist will determine which clients are allowed to connect. Leave blank or set to undefined to disable feature.

- `BLACKLIST` specifies a file path containing a list of IP addresses separated on different lines. If enabled, the whitelist will determine which clients are allowed to connect. Leave blank or set to undefined to disable feature.

- `WHITELIST_LAST` determines whether the whitelist or blacklist takes predence when they are both enabled.
	- When set to `true`, the whitelist "wins" if the client is on both lists (aka the client goes through even though they are also blacklisted).
		- In this scenario, the client also goes through if they are NOT on the blacklist, regardless of if they are on the whitelist or not.
		- This is because the blacklist is checked first, and the whitelist is only checked if the client is blacklisted.
		- This is similar to Apache's "Order Deny, Allow" option.
	- When set to true false, the blacklist "wins" if the client is on both list.
		- This means that the whitelist is checked first, and ONLY clients on the whitelist will proceed.
		- Then, the client is checked against the blacklist and is allowed to proceed ONLY IF they are NOT on the blacklist.
		- This means that only clients who are on the whitelist, but NOT on the blacklist will proceed. 
		- This is similar to Apache's "Order Allow, Deny" option.
	- This option is ignored if either the whitelist or the blacklist is undefined.
	- For the actual filtering logic, see app.js.

- `LIST_SCOPE` determines the scope of the whitelist/blacklist functionality (aka what the whitelist/blacklist will protect).
	- 0 => does not affect anything, listing functionality is turned off.
	- 1 => affects only POST requests to /api
	- 2 => affects both POST and GET requests to /api as well as requests to /stats/
	- 3 => affects all requests to the service.


# License

Shortnr uses the MIT License. Take a look at LICENSE for more info.


# Todos

For a list of things that still need to be done, take a look at TODO.md.