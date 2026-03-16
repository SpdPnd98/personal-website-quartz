---
tags:
  - Quartz
  - "#Blog"
---

I used to think whether it is possible to host my own obsidian vault online. In just a few years, that went from "probably possible?" to "most probably possible!" .

---
## Step 1: Editor setup
Install [Obsidian](https://obsidian.md/) in your computer. Obsidian should act as your editor and viewer, and it should just be that. For most people, this is probably sufficient, and you wouldn't need to continue further than this.

## Step 2: Repo setup
Follow this tutorial from [Quartz docs](https://quartz.jzhao.xyz/). You should have a `v4` branch by default. As Quartz uses node, its best to install some node management tools. I usually use [`nvm`](https://www.freecodecamp.org/news/node-version-manager-nvm-install-guide/) to manage my node environments. 

Install `nvm` and do `nvm use v22.17.0`. 

Then run `npm i`. 

Then `npx quartz create`. Go through the setup process.

## Step 3: Preview
You can (and should!) run `npx quartz build --serve` to view how the page would look like in browser.

If you need to change the default serving port, go to `quartz/cli/arg.js` and modify the port

![[Pasted image 20260317025045.png]]
I needed to do this because I have VSCode Server running on port `8080`.

For me, on a bad day I would definitely forget to set my `nvm` environment. If you're like me, you can write a `preview.sh` with the following contents:

```preview.sh
#!/bin/bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"

nvm use v22.17.0
npx quartz build --serve
```

After that `sudo chmod +x preview.sh`. From here, you can just do `./preview.sh` and you can see the preview locally!

## Step 4: Setting up Github Pages
Follow [Quartz Github Guide](https://quartz.jzhao.xyz/setting-up-your-GitHub-repository) to setup properly. The TLDR is 

1. Clone repo.
2. Change the `origin` to your repo.
3. Done.

Additionally, I followed the [Quartz Hosting Guide](https://quartz.jzhao.xyz/hosting#github-pages), modified the `on push branch` to a `deploy` branch (you can keep as `v4`, functionally no difference). 

You should do a one-time setup to do an initial push to your repo:
````
npx quartz sync --no-pull
````

Supposedly, you should be able to see your page under `<username>.github.io/<repo-name>`.

You can link it to a custom domain as well. Luckily, I found a comprehensive site [here](https://gist.github.com/plembo/84f80c920bb5ac6f19e53fe6f8db1ff7) that has exactly what you need to do. Alternatively, you can continue to read in the [next section](https://quartz.jzhao.xyz/hosting#custom-domain) on how to do this. As I am using Namecheap, here is how it looks like on my configuration page:

![[Pasted image 20260317030446.png]]

So yea, that's about it. Every time you run `npx quartz sync`, it should trigger a github workflow that compiles your Quartz site and deploy it.

Again, I might not remember this exact command, so I made a deploy script `sync.sh:
```sync.sh
#!/bin/bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"

nvm use v22.17.0
npx quartz sync
```

After this, every time i run `./sync.sh`, it should upload all the contents to github, then run the workflow and deploy the quartz site.

## Step 5: Edit your files with Obsidian
In Obsidian, open the folder `content` as a vault, then start editing (Which brings us to where we are now)! Remember to run `./sync.sh` every time you make a save!

---
# Conclusion
I'm quite amazed and how much easier deploying Obsidian vaults online are now! There are more stuff I'd like to do like `frontmatter`, `styles`, and `rss feeds`, which will be explored in due time.