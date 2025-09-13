# 11ty Website with TinaCMS

This guide shows you how to run your own 11ty (Static Site Generator) website with TinaCMS, giving non-technical users an easy way to edit content in the browser. This repository is a fork of [Bojidar Marinov’s excellent example](https://github.com/bojidar-bg/tina-true-selfhosted-example). Bojidar dis the hard work of making TinaCMS run with 11ty without relying on cloud hosting. This version: 
  
  - Provides a slightly more fleshed-out 11ty starter template.
  - A more concise README that walks through the key setup steps.
  * Expanded instructions for setting up 11ty+TinaCMS on a public-facing server.

Follow the instructions below to create a local (on your machine) and production (live public website). Scroll to the bottom for a more detailed description of all the core technologies used. 

# Setup a local site
```bash
# INSTALL CORRECT VERSION OF NODE
nvm install node 24         # Install node version 24 (version tina+11ty were built with)
nvm use node 24             # Use node vetrsion 24

# DOWNLOAD THE EXAMPLE GIT REPOSITORY
git clone https://github.com/bojidar-bg/tina-true-selfhosted-example.git
cd tina-true-selfhosted-example/
cp .env.example .env
echo 'AUTH_SECRET="add a random string here"' > .env

# BUILD AND RUN
npm install
npm run dev
# npm audit fix --force may need to run this if there is an issue

# View the local 11ty website
http://localhost:8080

# View admin pages
http://localhost:8080/admin/

# Key folders 
#  /11ty+tinaCMS-project/
#    └── cFYz8uJE_CGFNpi2c57N7Uontent             # Where all 11ty content and templates are stored
#    │      └── _includes    # 11ty templates
#    │      └── posts        # Collection of text files for 'posts' section of website. 
#    └── public
    │      └── admin        # 11ty templates
    │      └── _uploads     # Where media is stored for upload
    └── .env                # File to store variables for server setup
    └── tina
    │      └── config.ts    # Define collections for section of webiste
    │      └── database.ts  # Define how version control is managed through Git

```

# Setup a production (live/public) server 
In a production deployment, you should setup **three Git repositories** on your server:
1. **Bare repository** – the “central” repository that you push to. It contains no files apart from Git configuration.
2. **Build repository** – this is where the static website is built.
3. **Edit repository** – this is the working repository TinaCMS modifies. The `GIT_REPO` environment variable should point here.

```bash
  # ON THE SERVER
  # Make sure software is available (installing on an arch linux machine)
  pacman -S git               # Install git 
  pacman -S nvm               # Install node version manager
  nvm install node 24         # Install node version 24.x.x (used with this setup)
  nvm use node 24             # Use node version 24.x.x
  pacman -S base-devel        # Install arch linux 'make' tools
  pacman -S nginx-mainline    # nginx is a web server 
  pacman -S certbot           # Setup free SSL certificates with nginx 
  pacman -S certbot-nginx     # Configer for nginx 
  pacman -S logrotate         # Log rotate to help manage server space
  
  # Create three Git repositories
  mkdir -p ~/site/bare
  cd ~/site/bare
  git init --bare
```

```bash
  # RUN 11ty+TinaCMS on a production server
  npm run clean-build    # Rebuild the site
  npm run serve          # Run 11ty+TinaCMS without watching for changes
```

## Settings
- **tina/config.ts** Configures the ‘collections’ in your 11ty site.
- **eleventy.config.mjs** 11ty specific configuration.
- **tina/server.ts** If you are behind a proxy like \`nginx\`, you can probably remove the \`express.static()\` line. Also,
- **tina/database.ts** Every edit in Tina becomes a Git commit, so you have a full history of changes, authors, and revertability. This allows you to collaborate using pull requests, and means edits are reversable.
    - **Local-only:** Set `pushRepo: false, pullRepo: false` to save edits in a local repository and then push changes to GitHub/GitLab etc. manually.
    - **Remote setup:** Set `pushRepo: true, pullRepo: true` to make TinaCMS try to `git pull` before edits and `git push` after commits. That way your live content keeps in sync with a remote repository. 


# Core Technologies

**11ty** A simple static site generator. It takes templates and content (Markdown, HTML, etc.) and builds a fast, no-database website. In this project, 11ty is what actually creates the site visitors will see.

**TinaCMS** A content management system that runs alongside the 11ty site, providing an in-browser editor. It means users can log in and update content and upload images without touching code.

**Node.js** The JavaScript engine that runs site generation (build) tools and servers for both 11ty and TinaCMS. Without it, nothing runs.

**pnpm**  A modern and fast package (software) manager that downloads and manages the libraries 11ty and TinaCMS need - older package managers such as npm or yarn may be familiar to you. Used to install software dependencies and run the required servers. 

**Git** Version control software. You’ll use it first to clone this repository, and later it can track changes or sync your site with others.

**systemd** A built-in service manager on most Linux servers. Used to keep TinaCMS running in the background so your site stays online, even after reboots or crashes.

**Arch Linux** Operating System used on the live server, instructions for the production server will differ slightly for other OS’s. 