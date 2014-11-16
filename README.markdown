## Don't forget to commit source

    git add .
    git commit -m 'your message'
    git push origin source

## How to post

    rake new_post["Zombie Ninjas Attack: A survivor's retrospective"]
    rake generate   # Generates posts and pages into the public directory
    rake preview    # Watches, and mounts a webserver at http://localhost:4000
    rake deploy     # push _deploy to the master branch

