# How to deploy your Bedrock site with Moss

WordPress developers have their own workflows, and they sometimes differ from the usual ones in other CMS or frameworks. However, [Bedrock](https://roots.io/bedrock/) allows you to manage WordPress and related plugins as any other PHP dependency via `composer`. It also creates a better project structure that puts WordPress in its own subdirectory. These nice features make Bedrock a great alternative to consider when developing WordPress sites, since you can follow a typical PHP development workflow: Update dependencies locally, commit your `composer.lock`, and deploy your application to your testing/staging environment (or to production, eventually).

If you also take advantage of **Moss**'s **zero-downtime deployments**, your WordPress development workflow becomes a breeze. In this article we'll see how to create and deploy your Bedrock site with Moss. If you don't have an account yet, sign up at our **[Free Plan](https://moss.sh/signup/)** which includes **unlimited sites and servers**.


## Create your Bedrock project

In order to benefit from zero-downtime deployments, you need your WordPress site to be in a **git repository**. If you have an existing Bedrock project under git source control, you can skip this section and jump onto the next one.

In this example we're going to create a new Bedrock project called "bedrock" by means of `composer`. Once created, let's init the local git repo, commit our changes, and push them to the corresponding *origin*. For this post I've created a public repo in GitHub called "bedrock", so my *origin* is `git@github.com:fjros/bedrock.git` (use yours instead).

```
composer create-project roots/bedrock
cd bedrock/
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:fjros/bedrock.git
git push -u origin master
```

We already have a minimal Bedrock application under version control. Now it's time to set up our web and database server before deploying it. You can check the whole process in the following video:

```embed video
```


## Create the site in Moss

I assume you have a Moss account and at least one connected server. If that's not the case, first you must either [connect an existing server](https://docs.moss.sh/en/articles/3272248-connect-an-existing-server) or [launch a server from Moss](https://docs.moss.sh/en/articles/3272004-launch-a-new-server-on-a-cloud-provider).

Once you have it, just click "New site" and fill out the form:

* Select the server.
* Choose WordPress as framework. Don't check the "Install WordPress" box.
* Select the PHP version you want.
* Create the server user that will run your site.
* Provide the domain name of your site. If you cannot change your DNS records at this moment, you can use the test domain Moss gives you.
* Give your site a name. This name helps you identify your site on that server, even if you change your domain name later on.
* Tell Moss if it must set up HTTPS. In such case, either provide your own certificate or ask Moss to automatically issue and renew free Let's Encrypt certificates. In case you're just testing out Moss, you can omit this step and set up the certificate later on - especially if you're using the [test domain](https://docs.moss.sh/en/articles/2333466-browse-your-website-using-your-server-s-ip-address) suggested by Moss.
* Create a new database connection. For this you must provide the database name, database username, and corresponding password. Moss will create the database, user, and grant the corresponding permissions.
* Check the "Install Postfix" box if you want WordPress to send emails from your server.
* Type `web` as the "Root dir".

Finally click "Create" and Moss will start a background **operation** to install the required software and perform the related configurations. Once the operation ends, your web and database server's been configured to host your Bedrock-powered WordPress site. But you haven't deployed it yet... and that's what you'll do in the next steps.


## Set your git repo

If you head to the "Deployments" tab you'll see the option to "add" a **git repo** to the site. In case you haven't linked your GitLab, GitHub, or Bitbucket account with Moss yet, this is a good time to do it (you can also choose the "Custom" option if you use a different git provider).

Once you've selected the provider, you must choose the user who owns your site's repo and select the latter from the list that will appear. Tip: If you have many repositories, the search filter will come handy to find the one you want quickly.

Finally, tell Moss the *branch* or *tag* it must deploy when you ask for it. Usually this can be `master`, but you may adopt the convention you like better (e.g. `deploy`, `trunk`, etc).

While Moss finishes the required configs, let's set up the deployment process.


## Set up your deployments

### Environment file

Your `.env` file doesn't belong to your git repo, since it contains environment-specific content (e.g. development, staging, or production configs) including passwords and secrets. Moss calls these kinds of files (or folders) **persistent** because they're shared across deployments. I.e. deploying your website won't override the `.env` file.

Hence, you must tell Moss that there's a persistent file called `.env` and you must provide it by any other means - since cloning your repo won't pull that file. The easier way is to edit the file locally and upload it to the proper path on your server using SCP or an [SFTP client](https://docs.moss.sh/en/collections/1899375-ssh-sftp).

Most of the information you need to fill out the `.env` file can be easily obtained from Moss, as shown in the video above. You can generate the remaining secrets from the [roots.io](https://roots.io/salts.html) site. Here you can see an example of what the file looks like:

```DB_NAME=bedrock
DB_USER=bedrock
DB_PASSWORD=mvQcodMzZ5mSlJ568a6IxoAlcGoTsEaf

# Optionally, you can use a data source name (DSN)
# When using a DSN, you can remove the DB_NAME, DB_USER, DB_PASSWORD, and DB_HOST variables
# DATABASE_URL=mysql://database_user:database_password@database_host:database_port/database_name

# Optional variables
# DB_HOST=localhost
# DB_PREFIX=wp_

WP_ENV=production
WP_HOME=https://bedrock.46.101.131.224.getmoss.site
WP_SITEURL=${WP_HOME}/wp

# Generate your keys here: https://roots.io/salts.html
AUTH_KEY='{oxl(siw&wNhj$_Pj|5B[/qyEIy6NuY=CG=OR#9L9;*#z`:cYV+_mj$]LNO(TbA:'
SECURE_AUTH_KEY='=vn`/|sSss+o;wtakM#*@R-tx[Hk}a`eN(QT0IymU2{zsdT9K+Ls@Kxn}+PJ[_WC'
LOGGED_IN_KEY='Q$HQ(A_S_QNI0*-dX-l%%HcJ}UB@vC{!A?]o>ASWenb%zvL(JWdTud$TT8O>Qh1d'
NONCE_KEY='):Wr*,(6dfpKP2vK@Ul=kaaLsADg}b/lh]k!RWqQ^*v1QI6G$,vTbY*wc,n}M?7q'
AUTH_SALT='`x6?OmXm9jXZhb$:X7Iq$f=W+mppU53CA>^?,;Lc.jDa<wo5oUbVH|sltt}-[`0Z'
SECURE_AUTH_SALT='-ywl;3&G0S=S=@qFIyTh/<HHp@H@N0VY`ATQKC?$vVKw^E$7+0n.|Fw{Y9.Ug+{?'
LOGGED_IN_SALT='B1ZoY!_`w!$9DL$<Tw*[8@lYSc&!hsYlwn4E*O-]2uDDU$*!}:&-fzIQL&XLgPOY'
NONCE_SALT='CvwJp&!bZ`H>zLwzAyGB_3|]vc*^W&sT@fzD_><l9/gMtjT+f%bd{|Vj_8URO?{]'
```

E.g. you may upload the file using SCP like this:

```
scp .env <site-user>@<server-ip>:~/sites/<site-name>/shared/
```

Once uploaded, let's tell Moss the command it must run to build your site during a deployment.

## Deployment scripts

Moss shows you the **Deployment Flow** it follows during a deployment. You can edit the **pre-activation** and **post-activation** scripts it runs. Since you're setting up a WordPres site using Bedrock, just edit your pre-activation script to run `composer`:

```
composer install
```

Add here any other command you may need to build your site, and let's deploy it at last!

## Deploy your WordPress site

### 1-Click

You just need to click "Deploy". When the corresponding operation is done, you can browse your site and you'll see the WordPress installator. You may also check the **Operation Log** to see the details of your deployment.


### Push-to-deploy

You may also deploy automatically by pushing your changes to the branch/tag you specified in Moss. In order to enable this feature, head to the Deployment Flow tab and turn on the **Pust-to-deploy** switch.



## Additional advantages of using Moss

From now on you can keep coding your application and deploy whenever you need automatically or with a single click. But using Moss hass additional advantages:

* We've already seen that all git-based deployments in Moss are zero-downtime. This also means that if a new deployment fails for any reason, your users won't be impacted because your previous release is being served at every moment.
* You'll receive a notification via email or Slack when your deployments succeed or fail.
* As you've seen in the accompanying video, Moss offers real-time visibility about the operations which are being run on your servers and sites. In addition you can check the log of each operation, what can help you troubleshoot an issue when it comes up.
* You may create the database on a remote server right from Moss, and it automatically opens the firewall so that your application servers may reach the database. Note: Remember to update the `DB_HOST` env var accordingly in this case.
* Moss performs lots of configurations aimed at improving the security of your servers.
* Moss only installs the software that each server actually needs.
* Moss supports teams in all plans (unlimited teammates).
* You may monitor a server and all the sites it hosts for US$5 per month (at most). In this way you can track the resource consumption of your servers, check that your sites are up, or whether they're reachable via HTTPS, among others. Oh, and you'll receive an alert if something goes wrong.
* [Help Center](https://docs.moss.sh/). Higher plans also include support via chat.
* And more! Moss has lots of features and we keep building more on a regular basis.


## Conclusion

In this article we've seen how you can follow a typical PHP development flow and easily deploy your WordPress sites without downtime using Bedrock and the Free Plan of Moss. You can use the latter for all your applications and servers, and upgrade to a paid plan only if you need additional integrations, deploy with higher frequency, or reach our support team.