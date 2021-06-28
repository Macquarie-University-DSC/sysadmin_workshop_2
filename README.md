# The Enterprise System Administration Workshop

In this workshop we plan to 

1. Deploy a static website using devops principles
2. Deploy a dynamic website with an api with devops principles 

Notice that while the website, and the api are very simple, something that would
be trivial to deploy. These methods are not for these websites, these are mean't
to be applied to enterprise level apps on a large or small scale. So apps that
while they don't have to be massive apps, they are generally mean't to be apps
that teams of people would work on.

## Prerequisites

In terms of knowledge it would be helpful but not necessarily required to knowledge

- Basic unix/linux file structures and directory structures (not essential)
- Basic unix/linux commands (just look them up if you struggle)
- Vim or Vi text editing (can also use the nano text editor, but vscode is not 
  gonna help)
- The Rocky Linux Image (Included)

In terms of what software would be required to follow along in case you want to
follow along which is not required, can just listen.

1. A github account connected to your university or instituition (see 2)
2. Would need the student developer pack from github, should take about
   three seconds and a university email to apply for one.
3. Namecheap domain (free with student developer pack and a .me extension)
4. Some vps (digital ocean droplet recommended) just the account needed setting
   them up will be covered, also doing these things are provider agnostic, i'll 
   probably use digital ocean, but i'll try provide instructions for all 
   platforms

## Introduction: What is devops and continuous delivery

From what I have read on the internet, most sys admins describe devops as a way
of life.

Now devops is seperate from the agile methodology but they often are used 
together and I find that they are easier to explain together.

Before the way software used to be developed is you would have a phase of 
development, then you would extensively test the app, and then you would release
(deploy) the app. These would happen in big increments of months or years and
follow a strict plan or schedule, then after this process you would discard and
move onto the second version, often a complete rewrite. They would often have a 
support period where they have hotfixes and patches but rarely big rewrites or 
new features often resulting in overall outdated and less quality in software.

The agile methodology takes the same simple idea of having a 
development -> test -> deploy cycle but instead these happen in smaller 
increments, and often have not quite extensive testing, then they wait for new 
issues to arise whether it be from the development point of view, or the users
point of view, and then repeat the cycle. It's agile because it is a cycle of 
feedback and incremental versions.

While in theory agile works, in practice, having to redeploy the app every 
increment or version or redo testing is a tedious and not worth while job. It 
makes the agile methodology fall short. Devops and continuous delivery is a 
process used to automate the process of deployment and testing. These convert
the testing and the release phases of agile into what we would call a devops 
pipeline.

A devops pipeline looks like.

Push from source control -> Staging -> Deploy

Where the first step is the development process.

Staging is where you build the app, extensively test it, then you create 
what we call artifacts such as the actual release binaries of the app.

Deployment is the task of configuring the server for production, and making it
accessible on the internet, taking the artifacts from the build process.

You can easily apply devops to nearly all your projects, being on a university 
level or on a professional level but depending on the amount of enterpriseness 
you might change your setup for example you could use github actions for open 
source or university development as your dev ops pipeline, or you might use 
ansible and jenkins like we will for something more enterprizy.

# Steps to setup postgres

1. Install postgres
2. switch users with `sudo -iu postgres`
3. Create a new user `exampleuser123` with `createuser --interactive` and set
   all permissions to none.
4. Create a new database with `createdatabase` called `todo_db`
5. Go into psql with `psql` and type `ALTER ROLE exampleuser123 WITH PASSWORD 'password123';`
6. Now run `GRANT ALL PERMISSIONS ON DATABASE todo_db TO exampleuser123;` and quit with ctrl+D


