DocPad websites can be deployed anywhere. Here are a few of the most common deployments.


## To a Node.js Hosting Provider



### Inside your website's directory

1. Add the following to your website's `package.json` file. Add all the dependencies you are using and make sure their versions are correct - as well as ensure all commas are correctly placed.

	``` javascript
	"engines" : {
		"node": "0.12",
		"npm": "2"
	},
	"dependencies": {
		"docpad": "6",
		"docpad-plugin-blah": "2"
	},
	"main": "node_modules/.bin/docpad-server",
	"scripts": {
		"start": "node_modules/.bin/docpad-server",
		"test": "node_modules/.bin/docpad generate --debug --silent",
		"info": "node_modules/.bin/docpad info --silent"
	}
	```

1. For [Travis CI](http://travis-ci.org) support, add a `.travis.yml` file that contains:

	``` yaml
	# March 17, 2015
	# https://docpad.org/docs/deploy
	language: node_js
	script: "npm test"
	node_js:
	  - "0.12"
	cache:
	  directories:
	    - node_modules
	```


### For deployment to [Heroku](http://www.heroku.com)

1. Create a `Procfile` file inside your project that contains:

	```
	web: node_modules/.bin/docpad-server
	```

1. Set your heroku instance to run in production mode

	```
	heroku config:add NODE_ENV=production
	```

1. [Follow the rest of the Heroku guide here](http://devcenter.heroku.com/articles/node-js)



### For deployment to [AppFog](https://www.appfog.com)

1. Create a `app.js` file inside your project that contains:

	``` javascript
	module.exports = require(__dirname+'/node_modules/docpad/out/bin/docpad-server');
	```

1. [Follow the rest of the AppFog guide here](https://docs.appfog.com/getting-started)



### For deployment to [Windows Azure](http://azure.microsoft.com/en-us/services/app-service/web/)

1. Create a deployment script that triggers the static content generation. To create the script run the following command using the [Windows Azure Cross-Platform Command-Line Interface](http://azure.microsoft.com/en-us/documentation/articles/xplat-cli/):

	```
	azure site deploymentscript --basic -t bash
	```

1. Modify the `deploy.sh` file by chaning the `# Deployment` section to the following lines. You can see a complete example of the deploy.sh file [here](https://gist.github.com/ntotten/4715760#file-deploy-sh).

	``` bash
	echo Handling deployment.

	# 1. Install npm packages
	if [ -e "$DEPLOYMENT_SOURCE/package.json" ]; then
	  cd "$DEPLOYMENT_SOURCE"
	  npm install --production --silent
	  exitWithMessageOnError "npm failed"
	  cd - > /dev/null
	fi

	# 2. Build DocPad Site
	echo Building the DocPad site
	cd "$DEPLOYMENT_SOURCE"
	./node_modules/.bin/docpad generate
	exitWithMessageOnError "DocPad generation failed"

	# 3. KuduSync
	echo Kudu Sync from "$DEPLOYMENT_SOURCE/out" to "$DEPLOYMENT_TARGET"
	$KUDU_SYNC_COMMAND -q -f "$DEPLOYMENT_SOURCE/out" -t "$DEPLOYMENT_TARGET" -n "$NEXT_MANIFEST_PATH" -p "$PREVIOUS_MANIFEST_PATH" -i ".git;.deployment;deploy.sh" 2> /dev/null
	exitWithMessageOnError "Kudu Sync failed"
	```

1. Last, create a `web.config` file in the `static` directory of your site with the URL rewrite rules shown below. These rules remove the HTML extensions from your URLs. You can see the main portions of this `web.config` file below. You can download the complete file [here](https://gist.github.com/ntotten/4715760#file-web-config).

	``` xml
	<rule name="RemoveHTMLExtensions" stopProcessing="true">
		<match url="^(.*)\.html$" />
		<action type="Redirect" url="{R:1}" appendQueryString="true" />
	</rule>
	<rule name="RewriteHTMLExtensions" stopProcessing="true">
		<match url="(.*)" />
		<conditions>
			<add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true"/>
			<add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true"/>
		</conditions>
		<action type="Rewrite" url="{R:1}.html" />
	</rule>
	```
	
1. [Follow the rest of the Azure guide here](http://blog.ntotten.com/2013/01/11/static-site-generation-with-docpad-on-windows-azure-web-sites/)



### For deployment to [Modulus](http://modulus.io)

1. [Follow getting started guide](http://help.modulus.io/customer/portal/articles/1640060-getting-started-guide)

### For deployment to [Docker](https://www.docker.io/)

1. [There is a docker file that should help with deployments.](https://github.com/docpad/dockerfile)



### Optional: Custom domains

If you're also wanting to use custom domains for your website, [follow the Heroku Guide here](https://devcenter.heroku.com/articles/custom-domains), or alternatively here is a generic guide:

1. Ping your server (e.g., `ping balupton.herokuapp.com`)

1. Grab the IP address from the output

1. Login to your domain's DNS manager

1. Create an A Record for your domain pointing to that IP address



## To Static Servers (Apache, Nginx, etc.)

1. Perform a generation for a static production environment using `docpad generate --env static`

2. Upload the generated directory to your server's `public_html` or `htdocs` directory

	1. If you use rsync, [checkout our DocPad rsync deploy script](https://gist.github.com/Hypercubed/5804999)


### For deployment to [GitHub Pages](http://pages.github.com)

1. Install the [GitHub Pages Plugin](/plugin/ghpages)

	```
	docpad install ghpages
	```

2. Deploy to GitHub Pages using the plugin

	```
	docpad deploy-ghpages --env static
	```


### For deployment to a Cloud Data Storage Provider (AWS S3, Google Storage, etc.)

1. [Checkout the DocPad Sunny Plugin](https://github.com/bobobo1618/docpad-plugin-sunny)

