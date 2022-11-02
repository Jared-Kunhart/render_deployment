# Render.com Deployment for App Academy Grads

<sub>Skip this step if you already have a Render.com account connected to your GitHub account.</sub>

Navigate to the [Render homepage](https://render.com/) and click on "Get Started". On the Sign-Up page, click on the GitHub button. This will allow you to sign in to Render through your GitHub account, and easily connect your repositories to Render for deployment. Follow the instructions to complete your registration and verify your account information.

## Limitations:
- The main limitation of the free Render Postgres database instance is that it will be deleted after 90 days. In order to keep your application up and running, you MUST create a new database instance before the 90 day period ends. Check out [Removing and Renewing PSQL Database](#removing-and-renewing-psql-database) for more information.
- You may only have 'one active free tier database'.
- Render.com will automatically create a **PAID** postgresql database when you deploy a web service and don't assign it the free database_url, make sure to delete it if you don't want to be charged.

## App Academy Projects

### - First App Academy Project (PUG)

**-Using Heroku Migration starts you off with a 7$ psql starter plan.-**

This link shows you how to create a web service and migrate your data from Heroku database to Renders database. Use this option if you have a ton of data added to your heroku that isn't normally seeded. It costs 7$ monthly but you can cancel  [https://render.com/docs/migrate-from-heroku](https://render.com/docs/migrate-from-heroku)

**Everything else below was not found in the link above or wasn't clear in the steps.**
- **Make sure to delete the paid 'starter' database and go with a free plan database.** You only need one psql database, link all your projects to the one database, use the transfer for that new database. They won't charge you unless you select a paid plan upon creating a new service.
- Alternatively, you can
- In your GitHub project repo main folder, create a file named Procfile. Put this text inside it. ***web: npm start***
- In your render dashboard ⇾ click your named project web service ⇾ click environment tab, you’ll need to add the DATABASE_URL and SESSION_SECRET the CLI Plugin generated.
- When copying data from PostgreSQL, you need to set up the External Database URL for the last step. Go to your dashboard, go into the PostgreSQL database, scroll down to access control and add 0.0.0.0/0
- That adds an External Database URL above, copy and paste for Step 4 so that your command should look similar to -

```
pg_restore --verbose  --no-acl --no-owner -d pg_restore --verbose  --no-acl --no-owner -d postgres://yourpostgres_user:BtzXaS5@dpg5l8g-your-postgres.render.com/postgresql_name_250 latest.dump
```

- **Make sure to delete the 0.0.0.0/0 connection once you are done.**


### - **Second App Academy Project (React/Express)**
Similar to the first project, just adding more keys/secrets this time, including AWS keys if they were used.


### - **Third App Academy Project (React/Flask) -**
Similar to Capstone


### - **Capstone App Academy Project (React/Flask or React/Express)**

Thank [David Rogers](https://github.com/9ziggy9) for most, if not all, of this. This is just edited for students that have already graduated.

#### Part A: Adjust configuration for building static files for React

In the **app/__init.py__** file, you will configure where React will build the application, and the URL for the static resources for server-side rendering.

First, adjust the app variable to include additional arguments:
```py
# app/__init.py__ file
# ... other imports
app = Flask(__name__, static_folder='../react-app/build', static_url_path='/')
# ...
```

#### Part B (Recommended): Adjust build script for React-App

Navigate to the __react-app/package.json__ file, and add `CI=false` to the build script.

```js
// react-app/package-json file
// ...
    "build": "CI=false && react-scripts build",
// ...
```

This will allow your build to continue even if there are deprecation warnings on your dependencies. If this were a commercial application in production, you would probably want to keep the Continuous Integration (CI) enabled and address all the warnings to keep your application up-to-date. But on a personal project, those warnings are fine, and it is best to make sure they are not an obstacle to deployment.

If you made any changes in this phase, commit and then continue.

#### Part C: Create a Postgres Database Instance

Sign in to Render using your GitHub credentials, and navigate to your Dashboard.

Click on the "New +" button in the navigation bar, and click on "PostgreSQL" to create your Postgres database instance.

In the name field, give your database a descriptive name related to your project. For the region field, choose the location nearest to you. The rest of the fields in this section can be left blank.

Click the "Create Database" button to create the new Postgres database instance. Within a few minutes, your new database will be ready to use. Scroll down on the page to see all the information about your database, including the hostname, username and password, and URLs to connect to the database.

You can access this information anytime by going back to your Dashboard, and clicking on the database instance.

#### Part D: Create a New Web Service

From the Dashboard, click on the "New +" button in the navigation bar, and click on "Web Service" to create the application that will be deployed.

> _Note: If you set up your Render.com account using your GitHub credentials, you should see a list of applications to choose from. If you do not, click on "Configure Account" for GitHub in the right sidebar to make the connection between your Render and GitHub accounts, then continue._

Look for the name of the application you want to deploy, and click the "Connect" button to the right of the name. Now, fill out the form to configure the build and start commands, as well as add the environment variables to properly deploy the application.

#### - Configure the Start and Build Commands

Start by giving your application a name. This is the name that will be included the URL of the deployed site, so make sure it is clear and simple. The name should be entered in sword-case.

<sub>*Note: If the name for your application already exists, it seems Render adds a hash afterward, i.e. I named my app 'test-deploy', Render did not provide any feedback that this was already used, and just named my app 'test-deploy-ajhv' so originally my REACT_APP_BASE_URL was not correct</sub>

Leave the root directory field blank. By default, Render will run commands from the root directory.

Make sure the Environment field is set to "Python 3", the Region is set to the location closest to you, and the Branch, is set to "main". Next, add your Build script. (called "build commands" on render) This is a script that should include everything that needs to happen _before_ starting the server.

For your Flask project, enter the following script into the Build field, all in one line: (this field was prepopulated with 'pip install -r requirements.txt' for me)

```shell
# build script - enter all in one line
npm install --prefix react-app &&
npm run build --prefix react-app &&
pip install -r requirements.txt &&
pip install psycopg2 &&
flask db upgrade &&
flask seed all
```

This script will install dependencies for the frontend, and run the build command in the __package.json__ file for the frontend, which builds the React application. Then, it will install the dependencies needed for the Python backend, and run the migration and seed files. Now, add your start script in the Start field: (this field was prepopulated correctly for me)

```shell
# start script
gunicorn app:app
```

#### - Add the Environment Variables

Click on the "Advanced" button at the bottom of the form to configure the environment variables your application needs to access to run properly. In the development environment, you have been securing these variables in the __.env__ file, which has been removed from source control. In this step, you will need to input the keys and values for the environment variables you need for production into the Render GUI. Click on "Add Environment Variable" to start adding all the variables you need for the production environment.

Add the following keys and values in the Render GUI form:
- SECRET_KEY
- FLASK_ENV **production**
- FLASK_APP app
- REACT_APP_BASE_URL https://this-application-name.onrender.com

In a **new tab**, navigate to your dashboard and click on your Postgres database instance.

Add the following keys and values:
- DATABASE_URL (copy value from Internal Database URL field)
_Note: Add any other keys and values that may be present in your local __.env__ file. As you work to further develop your project, you may need to add more environment variables to your local __.env__ file. Make sure you add these environment variables to the Render GUI as well for the next deployment._

Next, choose "Yes" for the Auto-Deploy field. This will re-deploy your application every time you push to main.

Now, you are finally ready to deploy! Click "Create Web Service" to deploy your project. The deployment process will likely take about 10-15 minutes if everything works as expected. You can monitor the logs to see your build and start commands being executed, and see any errors in the build process. Some common errors include not having requirements.txt updated properly or not setting the `FLASK_ENV` variable set to "production" and the `DATABASE_URL` key set to the "Internal Database URL" value from your Render Postgres database instance.

When deployment is complete, open your deployed site and check to see if you successfully deployed your Flask application to Render! You can find the URL for your site just below the name of the Web Service at the top of the page.

(https://this-application-name.onrender.com).


## Removing and Renewing PSQL Database

__Set up calendar reminders for yourself to reset your Render Postgres database
instance every 85 days so your application(s) will not experience any
downtime.__

Each time you get your calendar reminder, follow the steps below.

1. Navigate to your Render Dashboard, click on your database instance, and
   click on either the "Delete Database" or "Suspend Database" button.

2. Next, follow the instructions in Phase #3 above to create a new database
   instance.

3. Finally, you will need to update the environment variables for EVERY
   application that was connected to the original database with the new database
   information. For each application:

  - Click on the application name from your Dashboard
  - Click on "Environment" in the left sidebar
  - Replace the value for `DATABASE_URL` with the new value from your new database instance, and then click "Save Changes"
  - At the top of the page, click "Manual Deploy", and choose "Clear build cache & deploy".

4. After each application is updated with the new database instance and
   re-deployed, manually test each application to make sure everything still
   works and is appropriately seeded.
